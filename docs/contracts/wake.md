# `maw wake` native command contract

- **Status:** Proposed compatibility contract
- **Parent:** [RFC-0001](../RFC-0001-minimal-maw-kernel.md)
- **Source baseline:** `maw-rs@775c709b67add2a69c7c2dd3ecee18bbb5fcdb6e`
- **Source tree:** `2e66f58be215d5151e8326a43a9b034b96f6be11`

`wake` is a reserved native kernel command. An absent, refused, corrupt, or hostile optional plugin
MUST NOT intercept it. This document separates observed alpha behavior from the target provider
boundary; proposed hardening is not silently described as current behavior.

## Source authority

| Concern | Current owner |
|---|---|
| dispatch and orchestration | `crates/maw-cli/src/core_impl/wake.rs` |
| argv and validation | `core_impl/wake_argv.rs` |
| target/repository/session resolution | `core_impl/wake_target_resolution.rs` |
| engine/config composition | `core_impl/wake_engine_command.rs` |
| worktree mutation | `core_impl/workon.rs`, called by `wake_prepare_worktree` |
| pane readiness/launch confirmation | `core_impl/wake_pane_launch.rs` |
| peer routing | `core_impl/wake_peer_route.rs` |
| focused behavior tests | `core_impl/wake_tests.rs`, `tests/native_wake_cutover.rs` |

Serena was discovery tooling; the named source snapshot is the authority.

## Grammar

Observed usage text is:

```text
maw wake <target|all> [--task <slug>|--wt <slug>] [--repo <org/repo>]
  [--prompt <text>] [--on-ready <cmd>]
  [--all --all-local --attach|-a --no-attach --dry-run --fresh
   --from-snapshot --kill --layout <nested|legacy> --list --main --new
   --parent <session> --peer <node> --pick --resume --snapshot <id>
   --solo --split --yes|-y]
```

The parser additionally accepts these source-frozen forms even though usage omits some of them:

| Class | Accepted forms |
|---|---|
| value | `--task`, `--wt`, `--prompt`, `--repo`, `--issue`, `--pr`, `--incubate` |
| routing | `--parent|--session`, `--peer|--from` |
| launch | `-e|--engine`, `--engine-cmd`, `--name`, `--repo-path`, repeated `--on-ready` |
| other value | `--layout`, `--snapshot` |
| boolean | `--all`, literal `all`, `--all-local`, `--attach|-a`, `--no-attach`, `--dry-run`, `--fresh`, `--from-snapshot`, `--kill`, `--list`, `--main`, `--new`, `--pick`, `--resume`, `--solo`, `--split`, `--bud`, `--channels`, `--wait`, `--yes|-y` |

Every listed value form except `--repo-path` and `--session` also has the implemented `--flag=value`
form where `wake_equals_setters` defines one. Unknown flags fail. Except for `--all` without a
positional, exactly one target is required. `--main` also sets `solo`; the last attach/no-attach
flag wins. Help is success output at the CLI boundary even though the internal parser carries usage
as an error value.

## Input validation

- Target-like values reject empty/control-bearing values and unsafe tmux target forms.
- `--wt` and `--name` use slug validation; issue/PR values use positive issue validation.
- `--engine-cmd` is an operator command surface and remains native authority. It MUST never become
  a guest-supplied string.
- `--prompt`, `--task`, and hook text remain exact operator/config content; any future plugin handoff
  uses opaque content/command receipts, not reparsed guest text.
- `--repo-path` is an explicit filesystem override used by native callers. Guests never receive this
  path authority.

## Dispatch and route selection

1. Parse argv once.
2. Peer routing is considered only when the request is not list/all/dry-run/repo/incubate, the target
   is not a GitHub slug, and the target contains `:` or `--peer` is present.
3. The peer path loads signed federation configuration, lists local sessions, resolves local/self/
   peer/collision, and fails closed on route errors. A peer request uses the native authenticated
   transport; a plugin cannot select an arbitrary URL, token, or sender identity.
4. The local path lists sessions. List/all use the regular listing; creation uses the cold-start-aware
   listing. Listing failure exits `1` before worktree/tmux mutation.
5. Picker/fleet expansion happens before single-target resolution.

## Resolution precedence

For the normal local path, native resolution is:

1. exact live fleet-registry session;
2. typed registry target/window;
3. typed repository candidate;
4. explicit `--repo-path`;
5. explicit `--repo`;
6. explicit `--incubate`;
7. target that is a GitHub slug, `.`, relative path, or absolute path;
8. fleet/ghq repository lookup with ambiguity diagnostics.

The session is selected by explicit `--parent|--session`, detected live Oracle session, typed/fleet
hint, registered repo session, then collision-safe generated name. Window naming, worktree slug,
layout, `--main`, `--new`, and matched registry window are resolved before mutation.

## Engine and config precedence

Configuration is merged **in the resolved directory**, so repo/worktree layers matter.

- Engine identity: explicit `-e|--engine` > repo-layer config > user config > built-in `codex`.
- Resume: explicit `--resume`, or `wake.resume=true` unless `--fresh`.
- Channels: explicit `--channels`, otherwise `wake.channels=true`.
- Prompt: explicit `--prompt` > merged `wake.prompt` > absent.
- Resume command: complete `commands.<engine>-resume` wins, even over `--engine-cmd`, and emits a
  warning; otherwise `--engine-cmd` wins for cold start, then the resolved `commands.*` entry.
- Family fallback: Claude appends `--continue`; Codex/OMX inject `resume` after the binary; unknown
  engines receive the historical naive ` resume` plus a warning.
- The in-pane line is bare, not `exec`: `MAW_SESSION_WINDOW=<window> <engine command>`. The shell is
  expected to survive engine exit.

## Read-only and mutation modes

| Mode | Required behavior |
|---|---|
| `--list` | render inventory only; no repo/worktree/tmux mutation |
| `--all` | render/dispatch the frozen all-plan behavior; preserve `--yes` and dry-run rows |
| `--pick` | use the frozen picker/fleet behavior before target mutation |
| `--dry-run` | resolve and render; no worktree, session, pane, fleet, or hook mutation |
| live registry attach shortcut | with exact target/session, attach, no task/worktree, select active/first window only |
| normal | resolve, optionally prepare worktree, then apply |

Current `--split` parsing is effectively inert in this native flow. Changing it requires a RED test
and explicit behavior approval.

## Mutation sequence and truthful partials

The observable order is normative unless separately approved:

1. resolve target, repo, session, window, initial config, and command;
2. append the `resolve` phase audit;
3. if requested, plan worktree and refuse a live-window cwd mismatch;
4. create or reuse the worktree; then re-resolve directory-layer command/config from the final cwd;
5. verify the final repo directory and render warnings/fuzzy match;
6. probe session;
7. create/reuse a window or create a session; start the pane in the final cwd;
8. wait for shell readiness, submit the engine line, and confirm launch/trust-prompt state;
9. optionally select/attach, joining a deferred sender before returning;
10. upsert fleet state;
11. run post-wake hooks.

Failures after a confirmed earlier mutation report failure without claiming rollback. A non-idempotent
ambiguous pane/worktree result MUST NOT be retried automatically.

## Output and exit contract

- Success is exit `0`; parse, resolution, listing, path, worktree, tmux, confirmation, and route
  failures are exit `1` with `wake:` diagnostics on stderr unless a frozen picker/fleet row says
  otherwise.
- Dry-run/list/all rendering, ANSI warning/fuzzy/worktree lines, phase-audit visibility, and silence
  on the live attach shortcut are compatibility bytes and need golden fixtures.
- Phase timing is append-only audit state; slow pre-attach phases may render diagnostics. Audit write
  ordering relative to refusal is part of the provider amendment below.
- “Submitted and confirmed” means pane composer/launch confirmation, not end-to-end agent work.

## Optional provider boundary

Vendor policy may move behind a generic native provider contract, but `wake` remains owner of target,
config, worktree, process, tmux, secrets, and mutation ordering.

Before any provider-dependent mutation, native code MUST:

1. resolve the exact prospective final repo/worktree branch, commit, and directory-layer config;
2. bind provider/executable identity and prompt/config inputs to host-issued receipts;
3. invoke exactly one fresh plan-only provider instance with **zero workflow host imports**;
4. validate the complete plan and refuse missing/refused/malformed/under-capable providers with an
   actionable package/source repair message;
5. revalidate the branch/ref/config snapshot under a CAS or lock spanning exact-object worktree
   creation; and
6. only then append the normal dispatch audit and enter the existing mutation sequence.

An explicitly selected missing Codex provider has no native fallback and causes no worktree/tmux/
workflow mutation. Non-Codex engines continue without the Codex artifact. Provider output supplies
vendor arguments only: native config still owns base command, resume override, channels, prompt, and
final composition.

## Known gaps requiring RED plus approval

- Usage text is incomplete relative to the parser.
- Current flow resolves a command before worktree creation and re-resolves after mutation. Provider
  planning requires a pure prospective target-tree config snapshot before mutation.
- Current `workon_create_worktree` can add by mutable branch name; a check before that call alone does
  not close ref-move TOCTOU. Exact-object creation plus ref CAS/locking is required.
- Dispatcher audit currently occurs before wake execution. A provider refusal/no-mutation contract
  requires deliberately deferring only the provider-dependent wake audit while preserving exactly
  one audit on accepted and all non-provider paths.
- Any change to `--split`, output, retry, phase timing, or partial-mutation semantics is independent
  of extraction and must not be normalized opportunistically.

## Acceptance matrix

At minimum, fixtures cover:

- help, omitted/extra positional, every canonical/equals flag, unknown/control-bearing input;
- exact registry, sleeping registry, repo/ghq/path/fuzzy/ambiguous/missing, peer/local/collision;
- list/all/pick/dry-run/live-attach and no-mutation snapshots;
- task/worktree reuse/create/fresh/collision/cwd mismatch and branch-move race;
- session/window create/reuse, shell timeout, trust prompt, confirmed/failed/ambiguous send;
- every engine/config/resume/channels/prompt precedence row, including final-worktree config;
- attach/no-attach, fleet upsert, hooks, and partial failures after each confirmed mutation;
- optional provider accepted/missing/refused/malformed/stale/under-capable/no-import behavior; and
- all five RFC-0001 commands remaining callable with the plugin directory absent or corrupt.
