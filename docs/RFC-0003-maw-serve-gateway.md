# RFC-0003: maw Serve/God Gateway

- **Status:** Proposed
- **Parent:** [RFC discussion #1](https://github.com/Soul-Brews-Studio/maw-core-rfc/issues/1)
- **Related:** RFC-0001 (minimal kernel), RFC-0002 (`maw-tmux` substrate)
- **Extraction regression dependency:** [maw-rs#963](https://github.com/Soul-Brews-Studio/maw-rs/issues/963)
- **Implementation tracking:** dedicated serve issues to be generated after this RFC is accepted
- **Source baseline:** `maw-rs alpha@775c709b67add2a69c7c2dd3ecee18bbb5fcdb6e`
- **Source tree:** `2e66f58be215d5151e8326a43a9b034b96f6be11`

## Summary

`maw serve` is a native authenticated HTTP/WebSocket control plane for Oracle state, federation,
worktree orchestration, bounded tmux access, plugins, and the God browser UI. It remains native, but
it is not one of RFC-0001's five minimal public-kernel commands. Its remote attack surface,
browser protocol, daemon lifecycle, and release evidence require an independent specification.

Paragraphs marked **Observed** describe the frozen source. **Proposed** requirements are targets,
not claims about alpha. A security improvement that changes observable behavior requires a RED
fixture and an explicitly approved correction.

## Why a separate boundary

The five-command kernel must remain locally recoverable when optional systems fail. Serve instead
opens a network listener, authenticates peers and browsers, bridges WebSockets to engines/tmux,
projects persistent state, and can initiate mutations. A kernel cutover must not silently change
that network contract, and a serve failure must not redefine the local kernel.

## Current source ownership

| Concern | Native source |
|---|---|
| CLI server, routes, auth gate | `crates/maw-cli/src/core_impl/serve.rs` |
| `status`/`stop`, PID and port evidence | `core_impl/serve_daemon.rs` |
| API/WS pipeline and registries | `crates/maw-cli/src/serve_core/mod.rs` |
| God UI routes | `serve_core/modules/god_mode_ui.rs` |
| WS `/ws/pty` and `/ws/tmux` | `serve_core/modules/websocket_routes.rs` |
| identity/federation/pairing | `identity_routes.rs`, `federation_routes.rs`, `pairing.rs` |
| agents/worktrees/threads/triggers | corresponding files in `serve_core/modules/` |
| black-box and wire behavior | `core_impl/serve_tests.rs`, `serve_core` module tests |

`serve_daemon.rs` also co-dispatches `messages`; shared-file ownership does not automatically make
`messages` part of this RFC. It requires a separate disposition before either source is moved.

## CLI and lifecycle contract

The frozen command grammar is:

```text
maw serve [--host|--bind <IP>] [--port <0..65535>] [--cached-pubkey <key>]
maw serve status|--status|stop
```

1. **Observed:** unknown flags, unexpected positionals, missing values, invalid IPs, and invalid
   ports return the established usage/error exit behavior before binding or mutation.
2. **Observed:** start owns one PID record and listener; status combines PID, health, and listener
   evidence rather than trusting only a stale file; stop is bounded and idempotent.
3. **Observed defect:** pidfile PIDs receive maw identity validation, but a responding configured
   listener PID can be selected for SIGTERM without that validation. A non-maw `/api/health`
   responder may therefore be killed.
4. **Proposed hardening:** identity-validate every selected PID immediately before signaling and
   refuse unrelated/changed listeners. This requires a frozen RED and human approval.
5. Stale PID cleanup, port-without-PID reporting, and stopped/live rendering remain fixture-frozen.
6. Logs, PID files, and state paths remain native and are never guest-selected.

## Bind and exposure contract

**Observed:** current alpha defaults to `0.0.0.0:3456`; `--host`/`--bind` accepts an IP address and
`--port` overrides the port. Current parsing does not use `MAW_HOST` for this default.

That broad bind is observable compatibility, not a recommended deployment default. Changing it to
loopback is a separate human-approved behavior correction. It conflicts with the restrictive-network
default in maw-rs's constitution, so acceptance MUST record an explicit preserve-or-correct decision.
Until then:

- startup MUST print the effective bind and authentication mode;
- non-loopback exposure MUST never weaken authentication because the source network seems private;
- firewall/NAT state is outside maw's proof and MUST NOT be inferred from a successful bind;
- operators need explicit documentation for loopback, LAN/VPN, and authenticated reverse-proxy use;
- a status response proves listener state, not end-to-end Internet reachability or confidentiality.
- a same-host reverse proxy appears as a loopback TCP peer; with `loopbackExempt=true`, proxy traffic
  can bypass the serve token unless the proxy authenticates it or the exemption is disabled.

## HTTP route families

The native router owns the route families below. This section is an architectural umbrella, not the
blocking full registry appendix; acceptance requires a generated method/path/auth/limit/effect table.

- health/info/configuration and static God views;
- sessions, bounded capture, agents, and pane-key delivery;
- wake/workon orchestration and worktree inventory/cleanup;
- send/feed/message ledger/request/reply and threads;
- identity, peer federation, discovery, probe, trust, and pairing;
- plugin manifests, proxy routes, health, and reload;
- triggers, people analysis, costs, teams, UI state, asks, and pin information.

Every route MUST declare method, path, authentication class, body/query limits, response schema,
side effects, idempotency, and failure codes. Duplicate registrations fail startup. Unknown API
paths remain API errors and MUST NOT fall through to an SPA page.

`/api/teams` currently reads `~/.claude/teams`. Before Team extraction, its owner must be explicitly
classified as a bounded native projection or replaced by a typed external-state projection; it may
not keep ambient Team filesystem policy accidentally. `/api/orchestration/workon` likewise remains
native only through RFC-0001's trusted work/workon boundary.

## Authentication and Origin policy

1. Origin is a browser routing constraint, not authentication.
2. The exact allowed God origin is `https://god.buildwithoracle.com`; validated configured origins
   and exact loopback origins may also be allowed.
3. Suffix lookalikes, trailing-slash variants, wildcard/null origins, malformed ports, duplicated
   Origin headers, and untrusted browser origins fail closed.
4. The Origin check precedes any loopback exemption and every mutation.
5. Missing Origin may support non-browser clients only when the route's normal token/signature rule
   succeeds; forwarded-address headers cannot manufacture loopback trust.
6. **Observed:** normal browser API requests use token auth; peer routes use native signed identity
   and pinned/trust-store rules. Credentials and secrets never appear in logs or response bodies.
7. Trust and pairing mutations remain native, auditable, atomic, scoped, and consent-controlled.

## WebSocket contract

- The stable application subprotocol is `maw.ws.v1`.
- **Observed:** the ticket-mint endpoint remains, but current alpha does not parse or consume a WS
  ticket. WS auth accepts a token header or loopback exemption and echoes only `maw.ws.v1`.
- **Observed gap:** a standard remote browser cannot set the required token header, so merely offering
  `[maw.ws.v1, ticket]` does not authenticate it. Current tests prove loopback negotiation, not
  cross-host browser authentication.
- **Proposed decision:** either restore path/origin-bound one-use ticket consumption client-first, or
  declare remote standard-browser WS unsupported. Credential carriers MUST never be echoed.
- `/ws` serves the God event/control stream; `/ws/pty` and `/ws/tmux` bridge only validated targets
  through RFC-0002's native adapters.
- **Observed ordering debt:** engine-open currently precedes the connection-limit check. Proposed
  behavior validates target, auth, Origin, any restored ticket, and quota before engine/bridge open.
- Binary/text frame handling, engine hooks, close behavior, idle timeout, and connection limits are
  bounded and fixture-frozen.
- Tmux-unreachable is a typed failure. The gateway never guesses another pane or broadens target
  scope after an error.

An HTTP `101` status is not sufficient proof: a browser rejects an upgrade that fails subprotocol
negotiation even when a status-line probe reports success.

## Orchestration and plugin boundary

Server-side wake/workon requests are parsed into bounded native plans. Repository containment,
engine/command tokens, prompt limits, authentication, and target existence are validated before
execution. Raw shell, Git, filesystem, tmux, environment, or secrets are not delegated to plugins.

Plugin-provided HTTP/WS surfaces require verified artifacts, collision-free route registration,
explicit auth classification, body/time/connection limits, and native proxy/process isolation. A
bad or unavailable plugin route fails locally; it does not disable the core health/recovery routes.

## State, concurrency, and truthful outcomes

- Thread, feed, request/reply, delivery, UI, and trigger state have explicit size/count/retention
  bounds and atomic or append-only storage rules.
- Duplicate delivery keys are idempotent within the defined window.
- Inbox write, tmux delivery, wake, worktree cleanup, plugin reload, and trust changes report
  confirmed, failed, or ambiguous outcomes. Non-idempotent ambiguous work is never retried blindly.
- Locks and compare/revalidate steps cover read-to-mutate transitions; a disappeared or substituted
  target fails closed before the next effect.
- Public response models expose display facts, never reusable filesystem/process authority.

## Compatibility and versioning

Freeze CLI bytes, route/method schemas, auth classes, CORS headers, WS protocol/frame behavior,
mutation ordering, defaults, and exit/error rows before change. API evolution is additive and
client-first: accepting clients land before a server requires a new field or protocol. Removed or
renamed routes need an explicit deprecation decision; extraction work alone is not authorization.

## Verification and browser-equivalent canary

Required automated evidence includes parser/lifecycle fixtures, route-registry collision tests,
auth and Origin matrices, signed-peer tests, CORS preflight/actual-response tests, body/connection
limits, orchestration injection guards, plugin proxy failure, tmux-unreachable, idle close, and
real-wire HTTP/WS tests. Ticket replay is required only if ticket consumption is restored.

Before promotion, run against the exact installed candidate and record:

1. candidate maw commit/tree and `maw --version` output;
2. effective bind/port and non-secret auth mode;
3. exact `Origin: https://god.buildwithoracle.com`;
4. the random `Sec-WebSocket-Key` sent by the client;
5. HTTP `101`, echoed `Sec-WebSocket-Protocol: maw.ws.v1`, and actual
   `Sec-WebSocket-Accept` matching that key;
6. an authenticated application frame and clean close; and
7. immutable logs/hashes tied to the reviewed candidate.

Run separate rows for a loopback browser, a non-browser client that can set token headers, and a
remote standard browser. Include wrong Origin, missing/wrong auth, non-loopback peer address,
credential-not-echoed, and—if restored—ticket replay negatives. A tokio client with custom headers
is not browser-equivalent authentication evidence.

A bare `PASS`, health response, TCP connection, or status-only handshake does not satisfy this gate.
Promotion records the candidate-to-tag SHA mapping, identical tree hashes, and installed tagged
version; a merge promotion cannot pretend both commits have the same identity.

## Failure and rollback

Retain the prior release artifact/config long enough to roll back. Rollback restores the previous
serve candidate; it does not reactivate shadow product routes or weaken auth. If state migration is
partial or authority is ambiguous, stop and report recovery instructions rather than guessing.
RFC-0001's five local commands remain usable while serve is stopped or rolled back.

## Non-goals

- Making serve a sixth minimal-kernel command.
- Moving auth, signing, trust, raw sockets, tmux, process, or filesystem authority into WASM.
- Treating broad bind as proof a firewall should be removed.
- Replacing the full route inventory with this summary; implementation must generate and test the
  exact registry from source.
- Normalizing observable behavior without a frozen RED and explicit approval.

## Acceptance

This RFC is ready for issue decomposition when the exact route/auth inventory is generated from the
frozen source, every remote mutation has a bounded authority and outcome model, Team/workon/plugin
ownership ambiguities are resolved, browser and non-browser matrices are independently reviewed,
and the browser-equivalent canary is reproducible against an unchanged candidate.
