# Serve WebSocket, fallback, and runtime-plugin route ledger

- **Kind:** source inventory and proposed hardening appendix
- **Companion:** [RFC-0003: maw Serve/God Gateway](../RFC-0003-maw-serve-gateway.md)
- **Fixed-route ledger:** [65 fixed method/path combinations](serve-route-policy.md)
- **Observed baseline:** `maw-rs alpha@775c709b67add2a69c7c2dd3ecee18bbb5fcdb6e`
- **Source tree:** `2e66f58be215d5151e8326a43a9b034b96f6be11`

**Observed** sections describe the frozen source. **Proposed** sections are acceptance targets,
not claims about alpha. In particular, current production code **does not consume WS tickets**.

## Observed core WebSocket registry

All three paths are fixed `GET` upgrades. A supplied Origin must pass the global exact-origin
policy. Their strict S gate then requires either the actual connection IP to be loopback (when the
loopback exemption is enabled) or an exact configured token in `Authorization`/`X-Maw-Token`.
Unlike ordinary `/api/*`, no-token and forced-open modes do not fall open for `/ws` or `/ws/*`.

| Path | Owner/kind | Target and operation | Observed limits and debt |
|---|---|---|---|
| `/ws` | God UI / Engine | Optional target; initial feed/session data, status and pane previews, inbound engine frames. | Target <=128 safe characters. Core connection/frame timers apply, but generated snapshots/captures have no independent output-byte ceiling. |
| `/ws/pty` | WS module / Pty | Optional validated tmux/PTY target; native process-engine bridge. | Same core limits; bridge availability is reported as engine/PTY failure. |
| `/ws/tmux` | WS module / Tmux | Optional validated tmux target; native tmux stream. | Same core limits; target failure does not broaden to another pane. |

`target` or `session` is taken from the raw query by simple splitting, then validated: nonempty,
at most 128 bytes, trimmed, not `--`/flag-shaped/control-bearing, and limited to the native safe
character set. There is no guest-selected raw command.

### Core stream limits

| Control | Observed default |
|---|---:|
| Stable negotiated subprotocol | `maw.ws.v1` only |
| Inbound text/binary frame | 64 KiB |
| Global active connections | 128 |
| Idle timeout | 30 s (environment accepts 1..3600 s) |
| Heartbeat | 10 s |
| Send timeout | 2 s |
| Capture/previews interval | 2 s (environment accepts 100..30000 ms) |

Oversized inbound text/binary frames trigger close. Ping receives pong, close is echoed, and send
operations are timed. The active counter is held by an RAII guard for the stream.

### Observed ordering defect

Origin/auth middleware and target validation run before the handler, but both `/ws` implementations
call `engine.servecore_ws_open(...)` **before** checking the connection ceiling. A non-atomic
pre-check is followed only after HTTP upgrade by the atomic connection guard. A race can therefore
open engine authority and/or return `101`, then close with code 1013. RFC-0003 proposes:

```text
Origin -> credential/ticket decision -> target -> reserve quota -> engine open -> 101 -> stream
```

No engine/process/tmux action may happen before the quota reservation, and every rejected step must
release any authority already reserved.

## Observed WS-ticket reality

`POST /api/auth/ws-ticket` accepts only an allowed supplied Origin plus an exact configured token.
It rejects query strings, requires JSON content type and an exact `{"path": ...}` object, caps the
body at 128 bytes, and permits only `/ws`, `/ws/pty`, or `/ws/tmux`. The store issues a random,
Origin/path-bound entry with 30-second TTL, one-use consume semantics, and a 256-entry ceiling.

However, those consume semantics are unused in the gateway:

1. the production router injects the store only into the mint handler;
2. neither the API token gate nor any of the three upgrade handlers receives the store;
3. no production `maw-cli` call invokes `WsTicketStore::consume`;
4. an allowed Origin plus old-style token header upgrades without a ticket; and
5. the server negotiates only `maw.ws.v1`; it never echoes a ticket carrier.

Thus minting is live but authentication by ticket is not. A standard remote browser cannot set the
required custom token header, so the current mint endpoint does not make remote browser WS usable.
A status-line-only `101` probe is insufficient: the browser also verifies subprotocol negotiation
and `Sec-WebSocket-Accept`.

### Proposed ticket decision

Acceptance must choose exactly one client-first contract:

- restore Origin/path-bound, one-use consumption before target/quota/engine work, with replay,
  wrong-Origin, wrong-path, expiry, credential-not-echoed, and browser-equivalent tests; **or**
- declare remote standard-browser WS unsupported, remove/deprecate the misleading mint path, and
  document token-capable non-browser plus loopback clients.

Until that decision lands, specifications must say “tickets are minted but not consumed.”

## Observed fixed views and fallback

| Match | Auth | Observed behavior |
|---|:---:|---|
| `GET /topology` | O | Reads configured topology HTML; missing file is 404. No output-size ceiling. |
| `GET /fed` | O | Returns fixed inline federation HTML. |
| `GET /` | O | Reads configured door HTML, falling back to the inline federation page. |
| unmatched `/api` or `/api/*` | O/A | JSON 404; never falls through to an SPA page. `/api/*` still traverses the A gate first. |
| unmatched non-API `GET`/`HEAD` | O | Serves `ui_dist_dir` through `ServeDir` after traversal/containment checks. |
| unmatched non-API other method | O | 405. |
| any `OPTIONS` | O | Global CORS middleware answers valid preflight; this is not a registered handler. |

The static guard rejects literal/encoded parent traversal and backslashes and canonical-checks
existing files. Static file bytes and directory cardinality have no route-local ceiling.

## Observed runtime plugin patterns

At startup, every discovered manifest with `engine.serve` may add zero to four patterns. All valid
paths begin `/api/`, so they receive the ordinary A gate, **not** P or the strict core-WS S gate.

| Runtime pattern per plugin | When mounted | Method | Observed operation |
|---|---|---|---|
| `<prefix>` | `command` exists | ANY | HTTP reverse proxy or WebSocket upgrade; lazily starts plugin process. |
| `<prefix>/*path` | `command` exists | ANY | Same proxy for descendants; GET 404 may retry `<prefix>/index.html`. |
| `<health_path>` | always | GET | Proxies only when command process is already running; otherwise synthetic `ok`. |
| `<event_path>` | `eventPath` declared | GET | Returns manifest event names; not a live event stream. |

The launcher allocates a loopback port, splits the manifest command on whitespace, starts it in the
plugin directory with three serve environment variables, nulls stdio, waits up to roughly 500 ms,
and retains the child for kill-on-drop. HTTP forwards method, path/query, body, and Content-Type;
it does not forward arbitrary client headers. WebSocket bridging forwards frames bidirectionally
and passes the offered subprotocol header upstream.

### Observed plugin gaps

1. Because plugin WS lives under `/api/<prefix>`, it is treated as ordinary A traffic: no-token or
   forced-open mode can fall open, and it has no core WS connection guard, frame limit, idle timer,
   send timeout, stable-protocol selection, target validation, or ticket logic.
2. HTTP request bodies (`Bytes`), upstream response bodies (`response.bytes()`), execution time,
   concurrent requests, process count, and WS frames/connections are not route-locally bounded.
3. Health can report synthetic success before a command has started; event routes return metadata,
   not streaming delivery. Proxy/WS failures may close or return 502 without a durable outcome.
4. Discovery does not establish reviewed artifact identity here. A manifest supplies executable
   program/arguments and working directory; the gateway owns broad process authority.
5. `CORE_PREFIXES` is a hand-maintained, one-direction collision list. It omits fixed families such
   as `/api/auth`, `/api/config`, `/api/costs`, `/api/teams`, `/api/ui-state`, `/api/asks`,
   `/api/pin-info`, `/api/identity`, `/api/pair`, `/api/people`, and thread routes; it also checks
   only each plugin prefix, not a generated set of every fixed and peer-plugin pattern.
6. A colliding plugin is skipped with stderr rather than making the reviewed registry fail closed.

## Proposed plugin and fallback hardening

1. Build one typed registry from fixed and plugin routes. Compare method plus full pattern in both
   directions; reject duplicate/prefix/health/event/static conflicts before binding. Generate the
   public ledger and tests from that same registry.
2. Require verified, hash-pinned artifacts and typed argv. Keep process, socket, filesystem, tmux,
   environment, and secrets authority native and narrowly allowlisted.
3. Declare auth per plugin method/pattern. Default command, mutation, and WS surfaces to protected
   fail-closed authority; an A-only read requires explicit review. Never infer WS safety from an
   `/api/` prefix.
4. Add request/response/frame byte caps, deadlines, concurrency/process/connection quotas, idle and
   send timeouts, startup readiness failure, bounded logs, and deterministic shutdown.
5. Negotiate only reviewed subprotocols and do not forward or echo credential carriers. Validate
   any target before process/upstream work. Preserve JSON 404 for unknown API paths.
6. Report confirmed, failed, or ambiguous proxy outcomes. Do not turn “plugin unavailable” into a
   global serve failure or disable health/recovery routes.

## Stale-document ledger

- `docs/reference/serve-daemon-surface.md` says Origin-present WS must offer and consume a ticket.
  That is stale against this baseline; its consumption and “burn on failure” text is not current.
- The same historical reference describes WS as having no auth at its layer and lists an older
  protected-route set. Current strict S and expanded P policy come from the compiled Rust source.
- `docs/guides/plugin-http-and-streaming.md` correctly describes the proxy mechanism and one known
  collision, but “refuses a prefix that collides with a core route” is broader than the stale
  `CORE_PREFIXES` constant can prove. Treat it as guidance, not exhaustive registry evidence.

## Acceptance evidence

1. Assert the exact fixed and dynamic route/auth registry, duplicate negatives, and fallback/API
   separation against the unchanged candidate tree.
2. Run real-wire loopback, token-capable non-browser, and remote standard-browser rows. Record
   Origin, requested protocol, random WebSocket key, `101`, exact `maw.ws.v1` echo, matching
   `Sec-WebSocket-Accept`, an application frame, and clean close.
3. Prove wrong Origin/auth, same-host proxy policy, quota-before-engine-open, frame/body/time limits,
   tmux/engine/plugin unavailable, and credential-not-echoed negatives. Test replay only if ticket
   consumption is restored.
4. Test plugin hash/argv/route collisions, non-loopback forced-open behavior, upstream timeout,
   oversized HTTP/WS traffic, process cleanup, and bounded failure isolation.
5. Update or explicitly supersede the stale documents above. A bare health response, TCP connect,
   `101`, or test-only ticket-store consume is not acceptance evidence.

## Source anchors

- `crates/maw-cli/src/core_impl/serve.rs`, `serve_plugin_proxy.rs`, `serve_tests.rs`
- `crates/maw-cli/src/serve_core/mod.rs`, `process_engine.rs`
- `crates/maw-cli/src/serve_core/modules/god_mode_ui.rs`, `websocket_routes.rs`, `static_views.rs`
- `crates/maw-auth/src/core_impl/ws_ticket.rs`, `request_verify.rs`
