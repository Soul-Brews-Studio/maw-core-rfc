# tmux Compiled Authority and Guest-Boundary Ledger

- **Status:** Proposed current-truth appendix and target boundary
- **Parent:** [RFC-0002: Full maw tmux Substrate](../RFC-0002-maw-tmux-substrate.md)
- **Source baseline:** `maw-rs alpha@775c709b67add2a69c7c2dd3ecee18bbb5fcdb6e`
- **Source tree:** `2e66f58be215d5151e8326a43a9b034b96f6be11`

This ledger distinguishes compiled `maw-tmux` code from stale lookalike files, records existing
fail-soft semantics without endorsing them, and closes the target guest boundary. Source prefixes:
**T/** = `crates/maw-tmux/src/core_impl/`; **H/** =
`crates/maw-plugin-manifest/src/core_impl/wasm_host/`.

## Compiled include truth

Rust `include!` edges, not filename resemblance, determine the implementation compiled at the
baseline.

| Root module | Direct compiled includes | Compiled descendant methods | Similar files that are **not** included |
|---|---|---|---|
| `T/client_pane_send.rs` | `client_pane_send_parts/pane_send_methods.rs`, `pane_title_writer.rs` | `pane_capture_resize_methods.rs`, `pane_layout_tag_methods.rs`, `pane_text_send_methods.rs` | `pane_capture_resize_method_defs.rs`, `pane_layout_tag_method_defs.rs`, `pane_text_send_method_defs.rs` |
| `T/client_session_window.rs` | `client_session_window_parts/client_constructors.rs`, `client_session_window_methods.rs` | `locked_split_methods.rs`, `session_listing_methods.rs`, `window_pane_methods.rs` | `locked_split_method_defs.rs`, `session_listing_method_defs.rs`, `window_pane_method_defs.rs` |

The `*_method_defs.rs` files are tracked but unreachable from these include chains. They MUST NOT be
cited as current behavior, used to “correct” a compiled observation, or copied into a new boundary.
Any cleanup is a later mechanical PR after caller and fixture proof.

## Current compiled fail-soft and partial semantics

“Current” freezes what callers can observe. A target correction needs a RED compatibility/security
fixture and approval; prose MUST NOT silently describe a proposed fail-closed behavior as present.

| Compiled symbol | Current observable behavior | Risk / target decision |
|---|---|---|
| `T/client_mode_env.rs::exit_mode_if_needed` | A failed `pane_in_mode` probe returns `Ok(false)`; cancellation errors containing “not in a mode” also become `Ok(false)` | Cannot prove “already normal” versus unreachable; typed caller chooses an explicit frozen compatibility row or fail-closed outcome |
| `T/client_mode_env.rs::try_run` | Any runner error becomes an empty string | Never use its result as authority or proof of absence |
| `set_option`, `set`; pane/window resize | Void/best-effort methods call `try_run`, so rejected mutations are not reported (`set_environment` is separate and returns `Result`) | New typed mutations return confirmed/failed-before-effect/ambiguous-after-possible-effect |
| `select_window`, `switch_client`, `kill_window`, `kill_pane` | Void methods discard runner failure through `try_run` | Existing callers require a ledger; destructive typed callers cannot use these void methods |
| `has_session` | Any `has-session` runner error becomes `false` | “Absent” is not proven when tmux is unreachable |
| `first_pane_id` | Runner error, empty output, or no nonblank line becomes `None` | Keep missing/unreachable/malformed distinct in typed resolution |
| `read_pane_tags` | Title probe failure is returned; metadata `show-options` failure becomes empty metadata | Display metadata cannot authorize a pane; expose explicit partial-read status where needed |
| pending-input capture heuristics | Unrecognized/empty prompt capture becomes `Cleared`/no pending; the compiled `capture_pending_input` runner error itself propagates | Heuristic “cleared” is not proof of agent consumption; preserve ambiguous confirmation and refuse unsafe retry |
| `new_session` | `new-session` failure is returned; later `renumber-windows` failure is discarded | Success may be partial: session exists but option is unset |
| `new_grouped_session` | Creation failure is returned; optional `window-size` and `select-window` failures are discarded | Success may be partial after creation |
| `get_pane_command` | Successful empty output returns `Ok("")` | Empty is malformed/unknown, not a safe command classification |
| `get_pane_info` | Successful empty output or a line without tab returns empty command and/or cwd | Cwd/command are display evidence only; malformed rows need an explicit typed error |

Route-level loops also have frozen partials: nested `close` and no-target `open` skip malformed
members and failed per-pane effects while reporting a success count. The route ledger owns those
facade contracts.

## `split_window_locked` is not a lock

The compiled `T/client_session_window_parts/locked_split_methods.rs::split_window_locked` builds one
`split-window` argv vector and calls the runner. It does **not** acquire a mutex, serialize other
callers, sleep, re-inventory, request `-P`, return the created pane identity, or verify a
postcondition. Its own documentation says callers own scheduling/locking.

Therefore:

1. the name is compatibility history, not a concurrency guarantee;
2. a typed split coordinator MUST acquire the scope lock before its final inventory and hold it
   through effect and result classification;
3. the action MUST request/parse an exact created-pane identity, re-inventory it, and return a typed
   confirmed/failed/ambiguous outcome; and
4. no guest may call `split_window_locked` or supply its optional shell command directly.

Required race tests run two same-window splits behind a barrier, prove unique issued pane refs, prove
no cross-target result, and make timeout-after-possible-effect non-retriable.

## Broad legacy plugin ABI: current evidence

The eight imports below are registered in `HOST_FN_NAMES` and dispatched by
`H/host_dispatch_audit.rs`. Their implementation is `H/host_tmux.rs`. “Success audit” means early
parse/capability/runner failures return before that method's audit call.

| Legacy import | Current capability check | Guest-controlled authority / data | Current audit | Target disposition |
|---|---|---|---|---|
| `maw.tmux.list_sessions` | `tmux:read` | Receives full session/window inventory | None in method | `BLOCK-LEGACY-ABI`; replace with scoped, field-minimal inventory |
| `maw.tmux.capture` | `tmux:capture` or `tmux:read` | Raw target, optional line count, and pane text; optional ANSI stripping | None in method | `BLOCK-LEGACY-ABI`; new guests get bounded declared marker matches only |
| `maw.tmux.send_keys` | `tmux:send`; force variants for destructive/forced paths | Raw target, joined keys/text, literal mode, optional Enter and AI-pane override flags | Success only, after effect; early denials/errors absent | `BLOCK-LEGACY-ABI`; replace with host-frozen payload ref and typed submit |
| `maw.tmux.run` | Delegates to `send_keys` with literal text plus Enter | Raw target and arbitrary submitted text | Logged as successful `maw.tmux.send_keys`; no distinct run audit | `BLOCK-LEGACY-ABI`; replace with a declared mission-pointer submit intent |
| `maw.tmux.command` | `tmux:read` for four read command names, otherwise exact `tmux:raw:<command>`; argv allowlist also applies | Raw command variant and argv/targets; allowlist includes mutations and limited shell-bearing split shapes | Success only; early denials/validation/runner errors absent | `BLOCK-LEGACY-ABI`; no raw-command equivalent in the new host |
| `maw.tmux.send_enter` | `tmux:send` | Raw target and count capped at five | None in method | `BLOCK-LEGACY-ABI`; typed submit owns Enter and confirmation |
| `maw.tmux.tags_read` | `tmux:read` | Raw path used as pane target; returns title and metadata | None in method | `BLOCK-LEGACY-ABI`; scoped pane ref plus field-minimal read |
| `maw.tmux.tags_write` | `tmux:write-tags` | Raw target/title and caller values under `@maw-*`/`maw-*` keys | None in method | `BLOCK-LEGACY-ABI`; closed tag keys and host-frozen values only |

The adjacent legacy SSH tmux capture/send imports carry remote process/session authority and receive
the same block rule; they require their own federation migration ledger. “Managed argv allowlist”
does not make `maw.tmux.command` a safe general guest boundary: the guest still selects raw targets,
operation shapes, and values, and the host constructs a native `CommandTmuxRunner` inside the call.

## Compatibility quarantine

Legacy imports may remain only for a recorded existing artifact whose repository path, source tree,
artifact SHA-256, manifest schema, and required import set all match an allowlist entry. The label
MUST include an owner, reason, migration issue, last permitted host generation, and expiry/review
date.

- A rebuilt, relocated, rehashed, widened, or newly versioned guest is not the same exception.
- A legacy guest gains no typed capability merely because it has a quarantine entry.
- New workflow artifacts MUST fail link/validation if they import `maw.tmux.*`, SSH tmux, a raw
  command host call, or any runner/socket escape.
- Removing a legacy exception follows accepted typed support, direct artifact invocation, an atomic
  owner switch, and a later mechanical cleanup; it is never inferred from low observed traffic.

## Target typed guest boundary

| Typed intent | Guest supplies | Host binds and proves | Limits and outcome |
|---|---|---|---|
| `tmux.inventory.v1` | Declared team/member scope and requested minimal fields | Same-invocation scope ref and source generation; display rows are non-authoritative | Row/byte/time cap; found/missing/ambiguous/unreachable are distinct |
| `tmux.observe_markers.v1` | Manifest-declared marker IDs | Pane scope ref and host-owned marker bytes | Boolean matches only; no pane text or repeated substring oracle; call/byte/time cap |
| `tmux.split.v1` | Closed orientation/size intent and an issued plan ref | Exact target, cwd and optional operator-approved command frozen before issuance | One-shot mutation budget; typed created-pane ref; confirmed/failed/ambiguous outcome |
| `tmux.submit.v1` | Issued payload ref and declared submit intent | Exact target and payload bytes/digest; pending-input policy and source generation | Shared cooldown/quota; no retry after possible effect; acceptance does not claim consumption |
| `tmux.layout.v1` | Issued pane-set ref and allowlisted preset | Exact member set, scope and pre-effect inventory | One-shot; truthful sequential partial or confirmed outcome as declared |
| `tmux.lifecycle.v1` | Closed action and issued owned-target ref | Membership, last-window/session protection, force role and fresh inventory | Destructive audit; confirmed/failed/partial/ambiguous; no automatic retry |
| `tmux.tags.v1` | Closed tag operation and issued pane ref | Allowed key set and host-frozen authoritative values | Field/byte/call cap; explicit partial read/write outcome |

All mutation refs are opaque, route/intent/plugin/team/member/repository/pane/generation/invocation
bound, atomically consumed before dispatch, and unreplayable after every terminal outcome. Budgets and
consumption state are shared across cloned host-function handles. A raw pane ID, title, cwd, command,
socket, environment value, capture text, or argv vector is never accepted back as authority.

Every call, including parse failure, denial and runner error, records plugin/artifact identity,
intent/version, scope and ref IDs, target/payload digests (not secret text), preflight generation,
budget state, effect phase, outcome class, duration and correlation ID. Audit failure for a mutation
fails closed before effect; audit failure after possible effect produces an ambiguous outcome.

## Mandatory proof and tests

1. **Include truth:** a test or generated manifest walks the actual `include!` graph and fails if a
   compiled methods file or lookalike `*_method_defs.rs` changes classification unnoticed.
2. **Pure types:** table tests reject empty, padded, leading-dash, control, malformed, ambiguous and
   cross-scope targets; opaque refs reject forged, stale, replayed and cross-route substitutions.
3. **Budgets:** cloned host handles share row/byte/time/call/mutation/cooldown limits; opening another
   guest instance cannot reset an invocation budget.
4. **Substitution:** target, pane set, cwd, command and payload changes after plan issuance are denied
   before a runner call; multi-target refs bind the exact allowed set.
5. **Effects:** fake-runner tests assert exact argv, selected socket, preflight ordering, postcondition,
   partial and ambiguous mapping, and no retry after timeout or possible mutation.
6. **Guest denial:** new-host link/dispatch tests prove all eight legacy imports, SSH tmux, raw command,
   `TmuxRunner`, `CommandTmuxRunner`, socket and environment access are unreachable.
7. **Observation:** marker APIs never return capture bytes and enforce finite caps; malformed or
   unreachable reads never become empty/absent success unless an explicit compatibility row says so.
8. **Audit:** success, parse failure, capability denial, stale ref, quota denial, pre-effect failure,
   partial and ambiguous outcomes each produce one correlated record without leaking payload text.
9. **Integration:** hermetic real-tmux tests cover cold start, failed existing server, stale/non-socket
   socket, target races, locked split concurrency, pending input, destructive lifecycle and teardown.
10. **Release:** invoke the accepted external artifact directly, run `scripts/gate.sh full` unpiped on
    the exact maw-rs candidate, and record candidate/tag tree equality plus artifact hashes.
