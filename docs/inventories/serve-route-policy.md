# Serve fixed-route and policy ledger

- **Kind:** source inventory; not an implementation promise
- **Companion:** [RFC-0003: maw Serve/God Gateway](../RFC-0003-maw-serve-gateway.md)
- **Observed baseline:** `maw-rs alpha@775c709b67add2a69c7c2dd3ecee18bbb5fcdb6e`
- **Source tree:** `2e66f58be215d5151e8326a43a9b034b96f6be11`

This ledger freezes what the baseline registers. **Observed** means the source does it today.
**Proposed** means RFC-0003 requires a decision or correction; it is not a claim about alpha.

## Counting rule

The baseline has **65 fixed method/path combinations**: 59 HTTP API/module combinations,
three core WebSocket upgrades, and three fixed HTML views. Axum's implicit `HEAD` handling for
`get(...)` is not counted separately. CORS `OPTIONS`, the static fallback, and the zero-to-four
runtime patterns per discovered plugin are policies rather than fixed registrations and are
covered in the [WS/plugin appendix](serve-ws-plugin-routes.md). The fixed wildcard
`POST /api/plugins/*plugin_path` counts once.

## Observed policy stack

The process defaults to `0.0.0.0:3456`. Middleware first rejects a supplied, disallowed Origin;
missing Origin is accepted for non-browser clients. Allowed browser origins are the exact God
origin, exact loopback origins, or an exact configured origin. The configured list is limited to
16 entries/4 KiB; duplicate Origin headers, wildcard/null/malformed origins, userinfo, and suffix
lookalikes fail closed.

CORS preflight accepts only `GET` or `POST`, only `Authorization`, `Content-Type`, and
`X-Maw-Token` (512-byte request-header-name ceiling), and an exact optional private-network flag.
Origin is a routing constraint, not authentication.

| Class | Observed authority |
|---|---|
| O | Origin policy only; no API credential gate. |
| A | O + `/api/*` token gate. `/api/health` is exempt. Non-WS requests fall open when no token is configured or forced-open is selected; actual-loopback exemption defaults on. |
| P | A + central `maw_auth::is_protected` default-deny. Accepts an Origin-token operator marker, actual loopback, or a verified HMAC/from-sign/Ed25519 identity. Auth buffers at most 64 KiB. |
| F | A + handler-local federation verification. Accepts operator context or actual loopback; otherwise requires a known/pinned sender and verified signed body/path. No route-local body ceiling. |
| L | A + handler-local actual connection-IP loopback check; forwarded-address headers do not help. |
| W | A + workspace HMAC over `METHOD:path:timestamp` in a 300-second window. Query and body are not signed. |
| S | O + strict core-WS gate: actual loopback or a configured token header. No-token and forced-open modes do **not** open these sockets. |
| T | WS-ticket mint only: supplied allowed Origin + configured token; no loopback exemption or forced-open path. |

Effects below are `R` read, `M` mutation/side effect, `Stub` success-shaped or deliberately
nonfunctional, `WS` long-lived stream, and `View` file/inline content. `--` in the bounds column
means no route-local ceiling was found; it does not assert that framework or OS limits do not
exist. `U` names an explicit inventory debt (unbounded cardinality/output/retention or no native
limit). All rows also inherit the Origin policy and, where applicable, the class policy above.

## Fixed registrations 1-26: native gateway handlers

| # | Method and path | Auth | Effect | Observed bound / debt |
|---:|---|:---:|:---:|---|
| 1 | `POST /api/auth/ws-ticket` | T | M | 128 B, exact JSON/path; 30 s, 256-entry mint store; no WS consumer. |
| 2 | `POST /api/send` | F | M | -- body/fields; delivery logs truncate text to 2,000 chars. |
| 3 | `GET /api/feed` | A | R | Process feed retains at most 200 events. |
| 4 | `POST /api/feed` | F | M | -- body; marks agent status and acknowledges, does not persist the body. |
| 5 | `GET /api/sessions` | P | R | 64 KiB auth read; U tmux pane/session output; tmux failure is 503. |
| 6 | `GET /api/capture` | P | R | 64 KiB auth read; U full pane capture; tmux failure is 400. |
| 7 | `POST /api/probe` | F | Stub | -- body; fixed `sessions: []` success response. |
| 8 | `POST /api/wake` | F | M | -- body/target/task; invokes the native wake executor. |
| 9 | `POST /api/pane-keys` | F | Stub | -- body; returns `ok` without a pane-key action. |
| 10 | `GET /api/transport/status` | A | Stub | Fixed connected transport row; no live transport probe. |
| 11 | `POST /api/transport/send` | F | Stub | -- body; always returns retryable 502 after authentication. |
| 12 | `GET /api/health` | O | R | Fixed process/bound-port projection; deliberately token-exempt. |
| 13 | `GET /api/message-ledger` | A | R | Scans the at-most-200 feed; query text has no local length ceiling. |
| 14 | `GET /api/requests` | A | R | U in-memory request store and result count. |
| 15 | `GET /api/trust` | P | R | 64 KiB auth read; U trust-file entries/output. |
| 16 | `POST /api/trust` | P | M | 64 KiB request/auth ceiling; persistent trust write, key redacted in response. |
| 17 | `POST /api/trust/revoke` | P | M | 64 KiB request/auth ceiling; requires JSON `yes: true`. |
| 18 | `POST /api/request` | A | M | U JSON/RAM store; labels the entry `delivered` without delivery evidence. |
| 19 | `POST /api/reply/:correlation_id` | A | M | U JSON/RAM store; one reply per existing entry. |
| 20 | `POST /api/workspace/create` | A | M | U JSON/RAM store; returns workspace token and join code. |
| 21 | `POST /api/workspace/join` | A | M | U JSON/RAM store; advertised join-code expiry is not enforced. |
| 22 | `GET /api/workspace/:id/agents` | W | R | U agents/output; only method/path/time are signed. |
| 23 | `POST /api/workspace/:id/agents` | W | M | -- body; U agents/nodes; body is unsigned. |
| 24 | `GET /api/workspace/:id/status` | W | R | U nodes, agents, and output cardinality. |
| 25 | `GET /api/workspace/:id/feed` | W | R | Caller `limit` has no maximum; U feed retention/output. |
| 26 | `POST /api/workspace/:id/message` | W | M | -- unsigned body; U feed retention. |

## Fixed registrations 27-59: serve-core modules

| # | Method and path | Auth | Effect | Observed bound / debt |
|---:|---|:---:|:---:|---|
| 27 | `GET /api/serve-core/pipeline` | A | R | Fixed middleware-order projection. |
| 28 | `POST /api/orchestration/workon` | P | M | 64 KiB; validates a workon plan then spawns through native orchestrator. |
| 29 | `POST /api/triggers/fire` | P | M | 64 KiB; module middleware records a trigger-bus event. |
| 30 | `POST /api/plugins/*plugin_path` | P | Stub | 64 KiB auth ceiling; fixed protected-stub response. |
| 31 | `GET /api/agents` | A | R | U tmux pane output; strict query grammar; tmux failure is 503. |
| 32 | `GET /api/agent` | A | R | Alias of row 31 with the same behavior. |
| 33 | `GET /api/plugins` | A | Stub | Default module state reports an empty plugin list. |
| 34 | `GET /plugins` | O | Stub | HTML rendering of the same default-empty plugin state. |
| 35 | `POST /api/plugins/reload` | P | Stub | 64 KiB auth ceiling; returns `reload-pending-auth`, performs no reload. |
| 36 | `GET /api/federation/status` | A | R | U configured-peer fan-out/output; performs live peer/session reads. |
| 37 | `GET /api/peers/discoveries` | A | R | Result `limit` defaults/caps at 50. Production discovery state is empty. |
| 38 | `GET /api/peers/discovered` | A | R | Alias of row 37. |
| 39 | `GET /fed.json` | O | R | U live peer fan-out; off-loopback redacts URL detail, IPs, agents, errors. |
| 40 | `GET /api/config` | A | R | Query strings rejected; typed, secret-free daemon-start snapshot. |
| 41 | `GET /api/costs` | A | Stub | Fixed zero/empty cost payload. |
| 42 | `GET /api/teams` | A | R | U ambient `~/.claude/{teams,tasks}` scan/output plus tmux liveness. |
| 43 | `GET /api/ui-state` | A | R | U JSON file read from serve current directory. |
| 44 | `POST /api/ui-state` | A | M | 64 KiB valid JSON; direct non-atomic file replacement. |
| 45 | `GET /api/asks` | A | R | U JSON file read from serve current directory. |
| 46 | `POST /api/asks` | A | M | 64 KiB valid JSON; direct non-atomic file replacement. |
| 47 | `GET /api/pin-info` | A | R | Fixed `{length,enabled}` projection; pin value is not returned. |
| 48 | `GET /api/identity` | A | R | Reads identity/peer-key; returns 409 when unpaired and exposes public key. |
| 49 | `GET /info` | O | R | Fixed public node/oracle/version/endpoints identity projection. |
| 50 | `POST /api/pair/generate` | A | M | -- JSON; U RAM code-store cardinality; no federation token in response. |
| 51 | `GET /api/pair/status/:code` | A | R | RAM-only pairing state; no persistence across restart. |
| 52 | `POST /api/pair/:code` | A | M | -- JSON; consumes RAM state and may return the federation token. |
| 53 | `POST /api/people/analyze` | L | M | 16 KiB; tmux target validation, process dedupe, then local delivery. |
| 54 | `GET /api/threads` | L | R | List limit defaults/caps at 50. |
| 55 | `POST /api/thread` | L | M | 64 KiB; store caps threads at 10,000 and text at 64 KiB. |
| 56 | `GET /api/thread/:id` | L | R | Numeric ID; U messages/output within one thread. |
| 57 | `GET /api/triggers` | A | R | U trigger-bus snapshot/output. |
| 58 | `GET /api/worktrees` | A | R | U `git worktree` subprocess output/list cardinality. |
| 59 | `POST /api/worktrees/cleanup` | P | M | 64 KiB auth ceiling; contained path then `git worktree remove`; U log size. |

## Fixed registrations 60-65: streams and views

| # | Method and path | Auth | Effect | Observed bound / debt |
|---:|---|:---:|:---:|---|
| 60 | `GET /ws` | S | WS | Target <=128 chars; inbound frame 64 KiB, 128 connections, 30 s idle; engine opens before quota. |
| 61 | `GET /ws/pty` | S | WS | Same core limits; native PTY bridge; engine opens before quota. |
| 62 | `GET /ws/tmux` | S | WS | Same core limits; native tmux bridge; engine opens before quota. |
| 63 | `GET /topology` | O | View | U configured HTML file size. |
| 64 | `GET /fed` | O | View | Fixed inline HTML. |
| 65 | `GET /` | O | View | U configured door HTML; falls back to inline federation HTML. |

## Observed cross-route debts

1. The A gate's no-token/forced-open behavior can expose A-only mutations on the default broad
   bind; a same-host reverse proxy also appears loopback when loopback exemption is enabled.
2. A-only mutation examples include request/reply, workspace create/join, UI/asks writes, and
   pairing. Pair acceptance can return a reusable federation token.
3. Several success-shaped stubs report availability or acceptance without performing the named
   operation. RAM stores and file/list/capture outputs commonly lack retention or byte ceilings.
4. W authentication does not bind query or body, so an authenticated envelope does not prove the
   workspace payload. Workspace join advertises but does not enforce expiration.
5. This table records registration, not safety: a `200` cannot prove delivery, persistence,
   transport health, or an unchanged target.

## Proposed decisions required by RFC-0003

1. Generate this ledger from the router and fail startup/tests on a duplicate, missing class, or
   undocumented route. Freeze schemas, status codes, effects, idempotency, and limits separately.
2. Explicitly preserve or correct `0.0.0.0` and A's fall-open modes. Non-loopback exposure must not
   become trusted merely because a token is absent, forced-open is configured, or a proxy is local.
3. Give every mutation a bounded native authority and confirmed/failed/ambiguous outcome model;
   eliminate or explicitly retain each stub under a compatibility decision.
4. Bound request fields, response bytes, store cardinality/retention, subprocess output/time, and
   peer fan-out. Sign the complete semantic workspace request if W remains.
5. Resolve ownership of ambient Team files, workon, pairing/trust, and plugin routes before
   extraction. No observed debt is silently normalized without a frozen RED and approval.

## Source anchors

- `crates/maw-cli/src/core_impl/serve.rs`, `serve_request_auth.rs`, `serve_workspace.rs`
- `crates/maw-cli/src/serve_core/mod.rs`, `process_engine.rs`, and `serve_core/modules/*.rs`
- `crates/maw-auth/src/core_impl/request_verify.rs`, `ws_ticket.rs`
- Runtime plugin and fallback details: [serve WS/plugin appendix](serve-ws-plugin-routes.md)
