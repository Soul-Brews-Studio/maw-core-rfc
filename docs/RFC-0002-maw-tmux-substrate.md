# RFC-0002: Full maw tmux Substrate

- **Status:** Proposed
- **Parent:** [RFC discussion #1](https://github.com/Soul-Brews-Studio/maw-core-rfc/issues/1)
- **Related:** RFC-0001 (minimal kernel), RFC-0003 (serve/God gateway)
- **Implementation dependency:** [maw-rs#963](https://github.com/Soul-Brews-Studio/maw-rs/issues/963)
- **Source baseline:** `maw-rs alpha@775c709b67add2a69c7c2dd3ecee18bbb5fcdb6e`
- **Source tree:** `2e66f58be215d5151e8326a43a9b034b96f6be11`

## Summary

`maw-tmux` is the privileged native session/window/pane substrate beneath RFC-0001's `wake`,
`attach/a`, `work`, `hey`, and `peek`, beneath selected administration commands, and beneath narrow
plugin host operations. It is not a sixth product-kernel feature and is never a raw plugin API.

This RFC specifies the complete architectural boundary: public facades, pure types/parsers,
resolution, native process/socket adapter, high-level actions, safety/failure semantics, typed guest
mediation, and conformance. Current-state statements are source-derived; proposed changes are
normative targets and require behavior fixtures before implementation.

## Architecture

```text
operator commands / verified plugin intents / serve bridge
                |
        typed semantic plan + opaque refs
                |
        maw-cli native policy adapter
                |
 pure maw-tmux types -> resolver -> TmuxClient actions
                |
       native CommandTmuxRunner only
                |
          tmux argv + selected socket
```

The layers MUST remain separable: pure types/parsers; live-state resolution; semantic plans and
outcomes; injected `TmuxRunner` tests; native-only `CommandTmuxRunner`; and maw-cli policy,
rendering, auth, intent, and plugin adapters.

## Current source map

| Layer | Source |
|---|---|
| domain types/constants | `crates/maw-tmux/src/core_impl/types_runner_parts/tmux_domain_types.rs` |
| outcomes/runner trait/errors | `types_runner_parts/action_outcomes_runner_traits.rs` |
| native runner/socket proof | `types_runner_parts/command_runner_adapter.rs` |
| parsers/target resolution | `parsers_resolution.rs` and `parsers_resolution_parts/` |
| live state | `live_state.rs` and `live_state_parts/` |
| safety/action resolution | `action_resolution.rs` and `action_resolution_parts/` |
| sessions/windows/locked split | `client_session_window.rs` and parts |
| pane capture/layout/tags/send | `client_pane_send.rs` and parts |
| nested CLI namespace | `crates/maw-cli/src/core_impl/tmux_dispatch.rs`, `tmux_*.rs` |
| plugin host today | `crates/maw-plugin-manifest/src/core_impl/wasm_host/host_tmux.rs` |

Important source types include `TmuxSession`, `TmuxWindow`, `TmuxPane`, `TmuxClient`, `TmuxRunner`,
`CommandTmuxRunner`, `PaneTargetResolution`, `TmuxAttachAction`, `TmuxSendCommandOutcome`,
`TmuxSplitActionOptions`, `TmuxKillOutcome`, and `TmuxError`.

## Current public inventory

The indexed `maw tmux` namespace currently registers:

```text
ls | list        peek          split          attach
break            close | unsplit              kill
layout           open          pipe | pipe-pane               sync
```

Adjacent top-level tmux-facing routes include at least:

```text
alive inspect capture view panes split break join swap pane resize focus
rename-pane send-text send-enter send-key send-escape tab tag zoom take
handover mv kill new done attach a peek wake work hey
```

This is an inventory seed, not permission to omit anything. Before acceptance, a generated route
ledger MUST enumerate every dispatcher entry and alias with its current handler, read/mutation class,
callers, fixtures, and exactly one disposition:

- RFC-0001 kernel, native tmux administration, external plugin, compatibility alias, or approved
  removal.

Shared files do not decide ownership. Nested `maw tmux peek` and top-level `maw peek`, or nested and
top-level split, are distinct/drifting implementations today and are not assumed semantic aliases.
The current `maw tmux` usage text advertises only five of the eleven semantic groups above; that is
frozen compatibility debt until a reviewed help correction lands.

## Public versus privileged boundary

- Top-level `attach/a` and `peek` remain RFC-0001 public features.
- `wake`, `work`, and `hey` consume native tmux operations internally.
- Top-level `split` moves to a verified external plugin backed by one typed action.
- `maw tmux ...` and every other adjacent verb need an explicit admin/plugin/compatibility decision;
  their existence does not expand the five-feature product kernel.
- `pipe|pipe-pane` is process authority and receives a dedicated decision; it cannot be generalized
  into “run any tmux command.”
- Serve's `/ws/pty` and `/ws/tmux` remain RFC-0003 native bridges over this substrate, not bypasses.

## Identity and target model

1. A session, window, and pane have distinct typed identities. Display names/indices/paths are not
   reusable authority.
2. Accepted target grammar is closed and rejects empty, padded, leading-dash, control-character,
   malformed, ambiguous, and cross-scope values before any runner call.
3. Resolution returns exact found/missing/ambiguous/unreachable states; it never silently chooses the
   first fuzzy match.
4. A mutation re-inventories and verifies the same session/window/pane immediately before effect.
5. Plugin and remote callers use same-invocation opaque refs bound to route, intent, scope, source
   generation, and target. Raw pane IDs alone do not prove team/member ownership.
6. Multi-target intents bind the exact allowed set (`--only`, `--keep`, current pane, charter/wave
   state, or reviewed operator roles); a valid ref for another member remains unauthorized.

## Socket, runner, and platform contract

`CommandTmuxRunner` remains native-only. It builds argv vectors, validates program/option values,
selects the reviewed socket, captures bounded output, and maps process errors into `TmuxError`; no
shell interpolation is permitted.

The error taxonomy distinguishes proven cold start/no server; failed existing server; stale,
absent, non-socket, or regular-file socket; invalid output; rejected action; and confirmed failure,
partial result, or ambiguity after possible mutation.

On Linux, cold-start proof may use socket metadata and `/proc/net/unix`; non-Linux implementations
MUST fail closed when absence cannot be proven. `TmuxRunner::is_initial_cold_start` defaults false.
A socket choice, environment value, or display path supplied by an untrusted guest is never accepted.

## Read and observation contract

Inventory, inspect, list, capture, panes, and peek operations declare scope, row/byte/line limits,
timeout, and data classification. Read error is not empty state. Native public `peek` may render
bounded pane text under RFC-0001; plugin observation receives only manifest-declared marker IDs and
boolean matches, never captured text or a repeated substring oracle.

Pane inventory exposes only fields needed by its declared intent. Any cwd or pane value later used
for Git/tmux authority is represented by an opaque issued ref; display fields are non-authoritative
and no host operation accepts them back.

Current top-level `peek` accepts any positive `u32` line count and `--history` requests full history.
A finite cap is a proposed hardening correction, not frozen current behavior. Likewise one attach
listing path currently maps tmux failure to empty while other paths preserve unreachable; the route
matrix must freeze and explicitly resolve that conflict rather than claiming it is already fail-closed.

## Send and submission contract

The source substrate implements literal send, paste-buffer paths, Enter, pending-input detection,
composer checks, settle/grace windows, confirmation, cooldown, quota, and retry tracking.

Normative rules:

1. Operator text, Enter-only, and host-derived mission-pointer payloads are distinct bound variants.
2. Exact payload bytes and target are frozen before effect; guest substitution is refused.
3. Every effect has a host-enforced call budget shared across cloned host handles.
4. Pending/destructive input checks occur before submission when the route requires them.
5. Outcomes are `confirmed`, `failed-before-effect`, or `ambiguous-after-possible-effect` with bounded
   evidence. Agent consumption is not implied by tmux acceptance.
6. A non-idempotent ambiguous submission is never retried automatically.
7. Quota/cooldown state cannot be reset by opening another guest instance.

## Split, layout, and lifecycle contract

Split binds target, orientation, percentage/size, cwd, and optional operator command into one opaque
one-shot plan; the guest cannot add or alter a command. Locked split prevents concurrent allocation
from racing the selected target.

Layout/join/break/close/swap/resize/focus/rename/zoom/tab/open/sync/kill operations use closed action
enums and issued pane sets. Allowed layouts come from the native allowlist. Every route freezes its
existing preflight-versus-sequential mutation ordering and truthful partials; changing a currently
partial sequence into all-before-first-effect atomicity requires a RED fixture and explicit approval.

Kill/done/lifecycle actions prove owned membership, re-inventory immediately before effect, protect
last-window/session rules, and distinguish graceful finish from forced termination. A mere inventoried
pane ID never authorizes destructive lifecycle work.

## Plugin host contract

Current alpha exposes broad legacy imports such as list/capture/send/run/command/tags and SSH tmux.
That is current evidence, not the target boundary. New or changed workflow artifacts MUST NOT receive
raw tmux, raw command, wildcard send, arbitrary SSH tmux, or `CommandTmuxRunner` access.

The target versioned host surfaces are finite semantic operations: scoped inventory, bounded marker
observation, split, pane submission, layout, lifecycle/batch launch, tags, and separately approved
administration actions.

Each surface has a manifest-declared intent, opaque scope/target/payload refs, finite read budget,
one-shot mutation budget, schema/version negotiation, and explicit outcome. Legacy exceptions are
path/source/hash-bound, gain no new typed capabilities, and have a recorded expiry/disposition.

## Safety invariants

- No raw tmux command is constructed from guest strings.
- No capture output, title, cwd display, or current command becomes authority.
- No valid ref crosses route, subcommand, team, member, pane, repository, or invocation scope.
- Action refs are consumed atomically before dispatch and cannot be replayed after confirmed, failed,
  partial, or ambiguous completion.
- Validation and budgets are shared per invocation across every registered host-function clone.
- Timeouts never convert possible mutation into safe retry.
- Platform/probe/parser failures remain explicit errors unless a frozen compatibility row says
  otherwise; any fail-closed correction needs separate approval.

## Compatibility and migration

1. Freeze grammar, aliases, stdout/stderr, JSON, exits, target resolution, timing-visible ordering,
   state effects, and errors for every facade.
2. Add pure/typed clients and fake-runner tests before changing the native runner or host policy.
3. Add accepting ABI/SDK support before any guest requires it.
4. Migrate one route/action at a time from broad legacy access to typed intent-scoped operations.
5. Build and directly invoke an accepted external artifact before downstream native ownership change.
6. Atomically switch one owner; delete inert code only in later bounded mechanical PRs.
7. Keep RFC-0001 regressions and RFC-0003 bridge tests green throughout.

## Conformance evidence

Required suites cover pure parser/resolver tables, fake runner argv/output/error mapping, hermetic live
tmux behavior, cold start, no server, stale/non-socket socket, empty server, malformed output, target
ambiguity/injection, locked split races, pending send, quotas, partial/ambiguous mutation, and no
retry. Plugin-host denial tests cover forged/stale/cross-scope refs, target/payload substitution,
replay, cloned-handle budgets, raw-import refusal, and runner inaccessibility.

A generated source/caller/fixture/ownership ledger and immutable logs/hashes accompany independent
review. `scripts/gate.sh full` runs unpiped on the exact maw-rs candidate; release evidence records
candidate and tagged tree equality rather than pretending a promotion merge has the same commit ID.

## Non-goals

- Counting the tmux adapter or admin namespace as a sixth public product feature.
- Exposing `TmuxRunner`, `CommandTmuxRunner`, raw capture, raw argv, sockets, or tmux environment to
  plugins.
- Rewriting operator UX while moving ownership.
- Assuming a route is safe to delete because its name is absent from this prose.
- Specifying Serve HTTP/WS policy, which belongs to RFC-0003.

## Acceptance

This RFC is ready for issue generation only after the exhaustive route ledger is source-generated,
every public facade and internal caller has one disposition, every action has a target/payload/scope,
call-budget and outcome contract, current-vs-proposed behavior is labeled, legacy raw access has an
explicit migration, the platform/error matrix is reviewed, and conformance can prove the privileged
runner remains unreachable from every workflow guest.
