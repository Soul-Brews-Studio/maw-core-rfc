# `maw work` native command contract

- **Status:** Proposed compatibility contract
- **Parent:** [RFC-0001](../RFC-0001-minimal-maw-kernel.md)
- **Source baseline:** `maw-rs@775c709b67add2a69c7c2dd3ecee18bbb5fcdb6e`
- **Source tree:** `2e66f58be215d5151e8326a43a9b034b96f6be11`

`work` is a reserved native kernel command for resolving a repository, creating or reusing an
isolated Git worktree and tmux window/session, launching its engine, registering eligible Oracle
sessions, and attaching or rendering the attach instruction. Optional plugins MUST NOT intercept it.

## Source authority and ownership

| Concern | Current owner |
|---|---|
| public `work` wrapper/help | `crates/maw-cli/src/core_impl/workspace_scaffold_commands.rs::work_run_command` |
| parse, repo/worktree/tmux/engine implementation | `core_impl/workon.rs` |
| shared engine config | `core_impl/wake_engine_command.rs::wake_resolve_command_from_config` |
| pane submission | native `sendtext_send_text` / `maw-tmux` runner |
| focused tests | `core_impl/{work_bundle_tests,workon_kind_tests}.rs`, `workon.rs::workon_tests` |
| integration/goldens | `tests/native_workon_plugin.rs`, `fixtures/native-workon/*` |

The public `workon` route currently reaches the same implementation and is not an alias declaration
of `work`. Its eventual retain/deprecate/externalize disposition is a separate compatibility decision.
It cannot be deleted merely because `work` is the desired public surface; `wake` also reuses helpers.

## Grammar

```text
maw work <repo|.|path|url> [task]
  [--wt [slug]] [--fresh] [--name <stable>]
  [-e <engine>] [--layout nested|legacy]
```

Observed parser contract:

| Input | Behavior |
|---|---|
| repo | required first positional; `.`, `./path`, absolute path, GitHub URL/slug, or ghq query |
| task | optional second positional; mutually exclusive with `--wt` |
| `--wt` | optional value; without a non-flag value chooses name, engine, then `codex` as auto slug |
| `--wt=SLUG` | named worktree request |
| `--fresh`, hidden alias `--new` | skip reusable worktree/window behavior where implemented |
| `--name VALUE`, `--name=VALUE` | stable name; requires task or `--wt` |
| `-e VALUE`, `--engine VALUE`, `--engine=VALUE` | engine selection; `-e=` is not accepted |
| `--layout VALUE`, `--layout=VALUE` | case-insensitive `nested` (default) or `legacy`; empty equals currently means nested |
| `--help`, `-h` | usage on stdout, exit `0` |

The wrapper rejects an empty argv and any literal `--` separator. More than two positionals, unknown
flags, leading-dash/`..` repo or engine queries, empty/unsafe slugs, task plus `--wt`, or a bare repo
combined with `--fresh|--name` fail. **Observed gap:** repo/engine validation does not reject control
characters despite its error vocabulary. Repeated scalar options are last-value wins.

The positional task is only a worktree/window slug seed. It is not fetched as an issue and is not
passed to the engine as prompt/context; documentation claiming otherwise is stale.
Because `--wt` has an optional value, placing it before the repo may consume the intended repo as the
worktree name and then fail for a missing positional repo; preserve this parse row.

All non-help errors exit `1` with stderr. As a synchronous native route, the dispatcher best-effort
appends its generic audit row before the handler, including invalid/help invocations.

## Repository resolution

1. `.` / relative / absolute paths resolve through the Git top-level.
2. A recognized GitHub slug or URL maps under `GHQ_ROOT/github.com/<slug>`; if missing, native code
   runs `ghq get` before continuing.
3. Other input searches ghq by the final path segment; duplicate basenames across organizations are
   sorted and the first is silently selected rather than reported ambiguous.
4. Absence reports `repo not found`; path/git/ghq errors are terminal.

There is no dry-run mode: repository resolution may clone. Raw filesystem, Git, ghq configuration,
and path canonicalization remain native authority and are never delegated to a guest.

## Worktree naming and layout

Task/name inputs are lowercased, whitespace-normalized to hyphens, restricted to ASCII alnum plus
`.`, `_`, `-`, collapsed for repeated dots, trimmed, and capped at 50 characters.

- Positional task supplies the worktree slug when `--wt` is absent.
- Named `--wt` supplies its value; value-less `--wt` uses `--name`, engine, then `codex`.
- A stable `--name` may be combined with a different `--wt`, producing `<name>-<requested>`.
- Branches are `agents/<worktree-name>`.
- Nested path is `<repo>/agents/<name>`; legacy path is sibling `<repo>.wt-<name>`.
- Without `--fresh`, one exact reusable slug is reused; ambiguous reusable rows fail.
- A stable named request may reuse an existing branch. An unnamed collision allocates a numbered
  `<N>-<slug>` by scanning worktrees/branches, with a bounded 1,000-candidate loop.

**Observed mismatch:** after a collision chooses path/branch `N-<slug>`, the tmux window still uses
the unnumbered requested slug. Name-only window reuse can therefore select a different cwd.

Creation currently makes the nested `agents/` parent and invokes `git worktree add` by branch name,
using `-b` when the branch is absent. Confirmed creation is not rolled back by later tmux/engine/
fleet/attach failure.

## Session and window behavior

The window name is the repo basename, or `<repo>-<resolved-worktree-slug>` for isolated work.

### Inside tmux

1. Read the current session through `display-message`.
2. List its windows.
3. If an exact window exists and `--fresh` is false, select it and return a reuse line; do not
   revalidate its cwd or relaunch its engine.
4. Otherwise create a window in the target cwd, resolve the command from that cwd, and submit it.
5. Register fleet state only for a taskless path recognized as an Oracle repo and only after a new
   session/window; selecting an existing window neither registers nor updates it.
6. Return progress text; no attach process is started.

### Outside tmux

1. Prefer an exact session named for the repo.
2. Otherwise resolve a unique numbered fleet-stem match; ambiguity fails rather than creating a
   sibling. Current list-session failure collapses to empty and may lead to creation.
3. Create a detached session in the target cwd or ensure the window in the existing session.
4. Submit the engine command and conditionally register taskless Oracle fleet state.
5. If stdout is a TTY, print accumulated progress then execute native `tmux attach -t <session>` with
   inherited stdio. Otherwise return `run: tmux attach -t <session>` in stdout.

The interactive attach bypasses the public `maw attach` resolver by design today. Changing that
composition is a separate behavior/security decision.

## Engine/config precedence

The final engine command is resolved against the **final repo/worktree cwd** using the shared weighted
merged config algorithm. Selection uses explicit engine, exact window/Oracle command keys, glob keys,
default config, then the `work`/`workon` built-in fallback `claude` (unlike wake's `codex`). Safe
`tokenPool` configuration may add the native pool prefix. The command is sent via the native bounded
pane-submission helper; a plugin never supplies executable text or environment.

`work` does not expose wake's prompt/resume/channels options. Any future shared provider extension
must preserve this distinction and use host-issued engine/command receipts.

## Output and truthful partials

Observable ANSI worktree/reuse/fleet/success lines, absolute display paths, lowercase `run:` attach
hint, silence while interactive attach owns the terminal, stderr wording, and exit codes are frozen
fixture material. There is no JSON mode.

Most downstream errors retain the `workon:` prefix and usage, and fleet JSON records
`created_by:"maw workon"` even when invoked as `work`; this vocabulary is observed compatibility
debt. There is currently no end-to-end public `maw work` success golden—only wrapper guards and
`workon` integrations—so direct public fixtures are required before refactoring.

Mutation order is:

1. generic dispatcher audit;
2. resolve/possibly clone repo;
3. inspect and create/reuse worktree;
4. inspect/create/select session/window;
5. submit engine command;
6. write fleet record when eligible;
7. attach or render attach hint.

A later failure reports error without claiming rollback. In particular: worktree may exist with no
window; window/session may exist with a failed engine submit; engine may run before fleet write fails;
and all may succeed before interactive attach fails. Never automatically retry an ambiguous Git,
tmux, or pane mutation.

## Security boundary

Native code retains repository roots, Git refs, branch allocation, filesystem creation, merged config,
engine command composition, tmux argv, pane send, fleet paths, environment, process execution, and
attach. Optional plugins may receive inert display facts only. Any typed composition uses exact
host-issued repo/workspace/engine/member refs, intent-scoped one-shot budgets, and truthful outcomes.

## Known decisions requiring RED + approval

1. public `workon` disposition and deprecation policy;
2. fail-closed session-list behavior instead of empty/create;
3. exact-object/ref-CAS worktree creation rather than mutable branch-name TOCTOU;
4. existing-window cwd/engine revalidation versus current select-only reuse;
5. composing interactive attach through the reserved attach contract;
6. help exposure of hidden `--new` and accepted equals forms; and
7. any JSON/dry-run addition or output normalization.
8. cross-organization ghq ambiguity, repo/engine control-byte validation, and numbered-path/window
   reconciliation.

## Acceptance matrix

- Canonical route remains callable with optional plugins absent/corrupt/refused/shadowing.
- Help, empty/extra/unknown/`--`, every flag/equals/last-wins, validation, and combination errors.
- Repo path/Git top-level/GitHub/ghq clone/search/miss/duplicate cases.
- Task/auto/named/stable/fresh slug, nested/legacy paths, branch reuse, worktree reuse/ambiguity/
  collision/number exhaustion, and ref-move race.
- Inside/outside tmux, exact/numbered/ambiguous/list-error session, window reuse/create/fresh.
- Final-cwd config and every engine precedence row; send confirmed/failed/ambiguous.
- Fleet created/existing/skipped/failure; TTY attach success/spawn/nonzero and non-TTY hint.
- Failure injection after every confirmed mutation with exact remaining state and output fixtures.
