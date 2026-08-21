# tmux Route and Ownership Ledger

- **Status:** Proposed inventory and cutover decisions
- **Parent:** [RFC-0002: Full maw tmux Substrate](../RFC-0002-maw-tmux-substrate.md)
- **Source baseline:** `maw-rs alpha@775c709b67add2a69c7c2dd3ecee18bbb5fcdb6e`
- **Source tree:** `2e66f58be215d5151e8326a43a9b034b96f6be11`

This ledger separates the privileged `maw tmux` administration namespace from RFC-0001's five
product-kernel commands. It records current source truth and a proposed owner; it does not claim
that similarly named handlers are aliases.

Source abbreviations below are relative to the baseline tree:

- **C/** = `crates/maw-cli/src/core_impl/`
- **T/** = `crates/maw-tmux/src/core_impl/`
- **H/** = `crates/maw-plugin-manifest/src/core_impl/wasm_host/`

## Disposition vocabulary

| Term | Meaning |
|---|---|
| `KEEP-NATIVE` | Keep a native operator/admin facade over typed `maw-tmux`; this does not add a product-kernel feature. |
| `PLUGIN-CUTOVER` | Move the public workflow facade to an accepted external plugin; only its finite typed host action stays native. |
| `COMPAT/RETIRE` | Freeze the current spelling while callers/fixtures migrate, then remove it in a separately approved change. |
| `UNRESOLVED-GATE` | Do not change ownership until the named security or compatibility decision is closed. |
| `BLOCK-LEGACY-ABI` | Never expose this authority through a new guest's legacy raw tmux imports. |

“Audit: none” means no route-local durable audit is emitted by the indexed handler. It does not
mean tmux or the surrounding shell produces no logs.

## Indexed `maw tmux` namespace

The registry has **11 semantic routes and 14 spellings**. `tmux_usage()` advertises only
`ls|peek|split|attach|break`; the six omitted semantic routes are frozen help debt, not hidden or
dead code.

| Current canonical surface | Post-cutover owner / disposition | Compatibility | Current authority | Current failure semantics | Audit semantics | Proof |
|---|---|---|---|---|---|---|
| `maw tmux ls` (`list`) | Native read-only admin, `KEEP-NATIVE` | Keep `list` alias while fixtures require it | Full local session/window inventory; `--json` changes rendering | Connect/list error is exit 1 “tmux unreachable”; JSON serialization fallback is `[]` | Current none; target logs only policy-sensitive remote use | `C/tmux_dispatch.rs::{TMUX_BUILTIN_SUBS,run_tmux_ls}` |
| `maw tmux peek` | `UNRESOLVED-GATE`: either bounded native admin capture or retire behind top-level `peek` | Freeze raw target, permissive flag handling, bytes, stdout, exits first | Raw pane/window target and captured pane text; `--lines` is any parsed `u32`, malformed/missing becomes 30 | Missing target exits 2; runner failure is target error; extra/malformed options may be ignored/defaulted | Current none; target must audit sensitive capture metadata, never content | `C/tmux_dispatch.rs::run_tmux_peek` |
| `maw tmux split` | Native admin action, `KEEP-NATIVE`; top-level `maw split` is separately `PLUGIN-CUTOVER` | Freeze nested grammar independently | Mutates target via `split_pane_action`; optional caller command; dry-run only prints a plan | Missing target exits 2; malformed pct silently becomes 50; action failure is explicit | Current none; target mutation audit includes target, frozen command digest and outcome | `C/tmux_dispatch.rs::run_tmux_split` |
| `maw tmux attach` | Top-level kernel `maw attach/a`; nested planner is `COMPAT/RETIRE` | Do not call it an alias: nested route renders plans and never attaches | Reads live sessions unless caller supplies `--alive`; renders local/remote attach or recover plan | Parse errors exit 2; tmux unreachable exit 1; recover plan exit 1; remote plan returns 0 | Current none; retirement mapping recorded | `C/tmux_attach.rs::run_attach_plan` |
| `maw tmux break` | Native pane lifecycle admin, `KEEP-NATIVE` + `BLOCK-LEGACY-ABI` | Top-level `break` becomes compatibility only | Validates target; refuses current pane unless `--force`; runs detached `break-pane` | Parse/help preserve codes; validation/refusal/runner failure are explicit | Current none; target attempt/outcome audit | `C/tmux_break.rs::{TMUX_SUB_280,tmux_break_with_runner}` |
| `maw tmux close` (`unsplit`) | Native pane lifecycle admin, `KEEP-NATIVE` + `BLOCK-LEGACY-ABI` | Keep `unsplit` alias until fixture-led retirement | Explicit target is detached; no target sequentially detaches every valid non-current pane | Explicit failure is loud; bulk mode skips malformed panes and failed detach calls, then reports only success count | Current none; target audit must preserve per-pane partials | `C/tmux_close.rs::{TMUX_SUB_281,tmux_close_with_runner}` |
| `maw tmux kill` | Native destructive admin, `KEEP-NATIVE` + `BLOCK-LEGACY-ABI` | Top-level `kill` is richer and not an alias | Re-lists, resolves exact pane/session, protects fleet-like sessions unless `--force`, then kills | Usage exits 2; ambiguity, protection, list and mutation failures are loud | Current none; target records preflight generation, force and confirmed/ambiguous outcome | `C/tmux_kill.rs::{TMUX_SUB_282,tmux_kill_session,tmux_kill_pane}` |
| `maw tmux layout` | Native window admin, `KEEP-NATIVE` + `BLOCK-LEGACY-ABI` | No top-level native spelling at baseline | Validates target; strips pane suffix; permits five fixed layout presets | Parse/validation/runner errors are loud; one `select-layout` effect | Current none; target mutation audit | `C/tmux_layout.rs::{TMUX_SUB_283,tmux_layout_with_runner}` |
| `maw tmux open` | Native pane/window admin, `KEEP-NATIVE` + `BLOCK-LEGACY-ABI` | No top-level native spelling at baseline | Target mode splits 50%; no-target mode sequentially joins single-pane hidden windows | Requires tmux; target failure loud; bulk mode skips malformed/failed joins and reports success count | Current none; target audit preserves per-window partials | `C/tmux_open.rs::{TMUX_SUB_284,tmux_open_with_runner}` |
| `maw tmux pipe` (`pipe-pane`) | `UNRESOLVED-GATE` + `BLOCK-LEGACY-ABI`: retain only if operator process authority and audit are approved | No top-level native spelling; never generalize into raw command | Opens/closes pane input/output pipe; accepts an explicit process command as joined positional text | Parse/target/command/runner errors loud; absent command closes the pipe | Current none; target requires command digest, direction, target and outcome before retention | `C/tmux_pipe.rs::{TMUX_SUB_285,tmux_pipe_with_runner}` |
| `maw tmux sync` | Native window admin, `KEEP-NATIVE` + `BLOCK-LEGACY-ABI` | No top-level native spelling at baseline | Sets `synchronize-panes` to a closed on/off value on resolved window | Parse/validation/runner errors are loud; one option mutation | Current none; target mutation audit | `C/tmux_sync.rs::{TMUX_SUB_286,tmux_sync_with_runner}` |

## Adjacent top-level tmux-facing routes

These are native dispatcher entries at the baseline, but only `peek` and `attach/a` belong to the
five-command product kernel. A target `maw tmux ...` spelling in this table is a proposed admin
facade, not proof that such a nested spelling exists today.

| Current canonical surface | Post-cutover owner / disposition | Compatibility | Current authority | Current failure semantics | Audit semantics | Proof |
|---|---|---|---|---|---|---|
| `maw ls` | Native fleet/read observer, `KEEP-NATIVE` outside the five-command product kernel | Keep Team annotation behind a generic reader; it is not a `maw tmux ls` alias | Composed fleet/session observation | Async planner errors/rendering are route-specific | Current route-local audit none; target read-policy metadata only | `C/fleet_list.rs::DISPATCH_151` |
| `maw capture` | `maw tmux capture` native admin, current top level `COMPAT/RETIRE` | Preserve `--full`, tail/window/pane resolution first | Pane text capture with caller-selected target/line range | Numeric/target/resolve/runner errors are explicit | Current none; target sensitive-read audit | `C/capture.rs::DISPATCH_77` |
| `maw alive` | `maw tmux alive` read admin, top level `COMPAT/RETIRE` | Preserve text/JSON dead-state contract | Probes session/pane liveness | Intentionally renders dead states for some probe failures | Current none; target policy metadata only | `C/tmux_alive_inspect.rs::DISPATCH_268` |
| `maw inspect` | `maw tmux inspect` read admin, top level `COMPAT/RETIRE` | Preserve text/JSON fields | Reads pane command, cwd and identity fields | Inspection/parse behavior must be frozen by fixtures | Current none; target sensitive-field audit | `C/tmux_alive_inspect.rs::DISPATCH_268` |
| `maw panes` | `maw tmux panes` read admin, top level `COMPAT/RETIRE` | Preserve target resolution, table and `--pid`/`--all` | Lists pane identity/size/title/command and optional PID fields | Missing/ambiguous/unreachable remain distinct where currently surfaced | Current none; target sensitive-field audit | `C/tmux_panes.rs::DISPATCH_76` |
| `maw peek` | RFC-0001 kernel, `KEEP-NATIVE` | This is canonical; nested raw peek may not substitute | Local/federated resolution, overview or pane capture under its own contract | See `docs/contracts/peek.md`; current routing and bounds debt stay explicit | Kernel observation audit policy | `C/tmux_peek.rs::DISPATCH_134` |
| `maw attach`, `maw a` | RFC-0001 kernel, `KEEP-NATIVE` | Both spellings remain public aliases | Resolve, plan, switch/attach or remote bridge under attach contract | See `docs/contracts/attach-a.md` | Kernel attach audit policy | `C/attach.rs::DISPATCH_111` |
| `maw split` | Verified external split plugin, `PLUGIN-CUTOVER`; typed split action stays native | Retire native facade only after artifact and fixtures pass | Target/orientation/percent/cwd/optional command mutation | Current parser/action/output remain frozen until atomic owner switch | Current none; typed host mutation audit required | `C/split.rs::DISPATCH_113` |
| `maw break` | `maw tmux break`, top level `COMPAT/RETIRE` | Current and nested handlers differ; migrate by fixture | Detached pane lifecycle mutation | Current top-level errors differ from nested route | Current none; target mutation audit | `C/tmux_break.rs::DISPATCH_280` |
| `maw kill` | Native lifecycle policy action; top-level owner `UNRESOLVED-GATE`, then admin or plugin facade | Rich local/peer/window/pane grammar is not nested kill | Remote forward plus guarded session/window/pane destruction | Exact/ambiguous/last-window/remote/partial contracts are route-specific | Current none; target signed destructive audit | `C/kill.rs::DISPATCH_78` |
| `maw join` | `maw tmux join`, top level `COMPAT/RETIRE` + `BLOCK-LEGACY-ABI` | Add typed admin facade before retiring | Resolves source then `join-pane` to destination | Parse/resolve/runner failure explicit | Current none; target mutation audit | `C/tmux_join.rs::DISPATCH_264` |
| `maw swap` | `maw tmux swap`, top level `COMPAT/RETIRE` + `BLOCK-LEGACY-ABI` | Preserve resolver and orientation | Swaps panes | Parse/resolve/runner failure explicit | Current none; target mutation audit | `C/tmux_swap.rs::DISPATCH_266` |
| `maw pane swap` | Consolidate into typed `maw tmux swap`; `COMPAT/RETIRE` | Distinct title/index/edge resolver must migrate, not vanish | Inventories and swaps two panes | Ambiguity and malformed inventory behavior fixture-gated | Current none; target mutation audit | `C/pane_swap.rs::DISPATCH_73` |
| `maw resize` | `maw tmux resize`, top level `COMPAT/RETIRE` | Preserve direction/amount grammar | Resizes selected pane | Current wrapper maps runner result to CLI result | Current none; target mutation audit | `C/tmux_resize_focus.rs::DISPATCH_328` |
| `maw focus` | `maw tmux focus`, top level `COMPAT/RETIRE` | Preserve target resolver | Selects pane | Resolve/runner failure explicit | Current none; target mutation audit | `C/tmux_resize_focus.rs::DISPATCH_328` |
| `maw rename-pane` | `maw tmux rename-pane`, top level `COMPAT/RETIRE` | Preserve title grammar | Selects pane and changes title | Validation/runner failure explicit | Current none; target mutation audit | `C/tmux_resize_focus.rs::DISPATCH_328` |
| `maw tab` | `maw tmux tab`, top level `COMPAT/RETIRE` | List/new/peek/send subflows migrate independently | Lists/creates windows, captures, or submits text | Multi-step creation/submission can be partial; fixture before move | Current none; target per-action audit | `C/tmux_tab.rs::DISPATCH_43` |
| `maw rename` | `maw tmux rename`, top level `COMPAT/RETIRE` | Preserve number/name resolver | Renames a window | Missing/usage/runner errors are route-specific | Current none; target mutation audit | `C/rename.rs::DISPATCH_38` |
| `maw zoom` | `maw tmux zoom`, top level `COMPAT/RETIRE` | Preserve fuzzy target and default-window behavior | Toggles window zoom | Missing/ambiguous/runner errors explicit | Current none; target mutation audit | `C/tmux_zoom.rs::DISPATCH_80` |
| `maw tag` | Typed tag admin action; top level `COMPAT/RETIRE` + `BLOCK-LEGACY-ABI` | Preserve read/write modes and metadata namespace | Reads/writes title and pane metadata | Title failure loud; option-read failure currently tolerated | Current none; target write audit | `C/tmux_tag.rs::DISPATCH_82` |
| `maw take`, `maw handover` | `maw tmux take`; aliases `COMPAT/RETIRE` + `BLOCK-LEGACY-ABI` | Preserve both aliases and split/session behavior until migrated | Creates session, moves window, may kill default window | Sequential mutation has truthful partial/failure debt | Current none; target per-effect audit | `C/tmux_handover.rs::DISPATCH_81` |
| `maw mv` | `maw tmux mv`, top level `COMPAT/RETIRE` + `BLOCK-LEGACY-ABI` | Do not conflate with `take` aliasing | Moves/renumbers window in session | Validation/runner failures explicit | Current none; target mutation audit | `C/tmux_handover.rs::DISPATCH_81` |
| `maw view` | Native view/lifecycle admin, owner `UNRESOLVED-GATE` | Grouped-session, split, attach, clean and kill modes need separate rows before move | Reads and creates grouped views; optional guarded pane/session cleanup | Plan-vs-mutate, sequential partials and destructive confirmation are route-specific | Current none; target signed per-effect audit | `C/tmux_readonly_view.rs::DISPATCH_260` |
| `maw send-text` | `maw tmux send-text` admin plus internal typed submit action; top level `COMPAT/RETIRE` | Preserve literal/buffer/pending-input confirmation contract | Sends frozen text and Enter to resolved pane | Current implementation detects conflicting/persistent pending input and ambiguous confirmation | Current none; target payload-digest/outcome audit | `C/send_text.rs::DISPATCH_84` |
| `maw send-enter` | `maw tmux send-enter`, top level `COMPAT/RETIRE` + `BLOCK-LEGACY-ABI` | Preserve current positive-`usize` count grammar as uncapped debt; a finite cap is a reviewed correction | Sends Enter one or more times | Stops on first failure and reports prior accepted count; acceptance is not delivery | Current none; target per-attempt audit | `C/tmux_attach.rs::DISPATCH_305` |
| `maw send-key`, `maw send-escape` | `maw tmux send-key`; aliases `COMPAT/RETIRE` + `BLOCK-LEGACY-ABI` | Preserve closed key allowlist and Escape alias | Sends one allowed tmux key to resolved pane | Dry-run supported; failure is explicitly unconfirmed | Current none; target per-attempt audit | `C/tmux_attach.rs::DISPATCH_305` |

## Non-alias and absence rules

1. Nested `maw tmux peek` is raw capture; top-level `maw peek` has resolver, overview and federation
   behavior. They are not aliases.
2. Nested and top-level `split`, nested planner and top-level `attach/a`, nested and top-level
   `break`, and the two `kill` routes have distinct parsers or policies. Shared tmux primitives do
   not make their observable contracts interchangeable.
3. There is no native top-level `layout`, `open`, `close`, `unsplit`, `pipe`, `pipe-pane`, `sync`,
   or `tile` dispatcher at the baseline.
4. `new`, `promote`, `done`, `finish`, `run`, `hey`, `send`, `wake`, and `work` are higher-level
   workflows that consume tmux; they are not aliases for raw/admin routes. RFC-0001 controls the
   native status of `wake`, `work`, and `hey`.

## Proof required before any cutover

- Generate the registry rows from `DISPATCH_*`, `TMUX_BUILTIN_SUBS` and `TMUX_SUB_*`; fail on an
  unclassified spelling or duplicate canonical owner.
- Freeze grammar, stdout, stderr, exit, argv, ordering, and partial effects for every row before
  moving it. Never delete a fixture to make a proposed owner fit.
- Prove the accepted plugin artifact by direct invocation before any `PLUGIN-CUTOVER`; then switch
  exactly one owner atomically.
- For every mutation, test invalid/ambiguous/stale targets, preflight generation, runner failure,
  possible-effect timeout, no automatic retry, and audit coverage.
- Keep raw runner/socket/argv access unreachable from guests; the companion authority ledger defines
  the only acceptable typed boundary.
