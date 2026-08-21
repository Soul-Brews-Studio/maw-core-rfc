# Dispatcher route ledger: index, method, and blocking rule

## Frozen scope

- Source commit: `775c709b67add2a69c7c2dd3ecee18bbb5fcdb6e`.
- Source tree: `2e66f58be215d5151e8326a43a9b034b96f6be11`.
- Registry construction: `crates/maw-cli/build.rs::collect_core_files` and `generate`; runtime iteration: `core_impl/dispatcher.rs::dispatcher_entries`.
- Part 1: [A–O](dispatcher-route-ledger-1.md).
- Part 2: [P–Z](dispatcher-route-ledger-2.md).

## Count proof

- Build-mirroring narrow parser scanned only direct `core_impl/*.rs` files at the frozen commit, omitted `mod.rs` and first-10-line `//maw:noauto`, and used the same first `DISPATCH_NN` / `TMUX_SUB_NN` discovery rule as `build.rs`.
- `227` auto-included Rust files; `152` generated dispatcher fragments (`133` non-empty, `19` empty).
- `200` source-declared `DispatcherEntry.command` spellings, all unique, grouped into `176` identical file/fragment/handler rows.
- Production registry has `199` of those spellings; `__async-dispatch-test` is the one `#[cfg(test)]` entry and is retained in the ledger so source coverage is literally exhaustive.
- Nested `maw tmux` has `4` built-in groups plus `7` generated `TMUX_SUB` groups: `11` groups, `14` unique spellings.
- The parser asserted every source spelling occurs once in these two parts, every group preserves its exact handler/file/fragment, and both top-level and tmux nested spelling sets have no duplicates.

### Proposed category totals (source spellings)

| Category | Spellings | Status basis |
|---|---:|---|
| **KERNEL5** | 6 | RFC-0001 exact retained public set (`attach/a` counts as two spellings). |
| **HOST-ADMIN** | 41 | Direct CLI/plugin/config/auth/trust/consent/transport administration named by RFC-0001 native-infrastructure requirements. |
| **RFC0002-TMUX** | 25 | Direct tmux inventory/read/mutation facades and the nested tmux namespace; top-level `split` is overridden by #963. |
| **RFC0003-SERVE** | 3 | Serve daemon and serve-named native helper routes; `messages` is deliberately not inherited. |
| **EXTERNAL-963** | 13 | Only routes explicitly assigned by `specs/001-lean-core-plugins/contracts/cli-ownership.md` and its frozen inventories. |
| **COMPAT/UNRESOLVED** | 112 | No accepted target assignment found; this is a blocker, not a keep decision. |
| **APPROVED-REMOVE** | 0 | Requires linked approval; none exists in the frozen evidence. |

## Classification constraints

- Categories are proposed ownership buckets, not proof that behavior may change. Current behavior stays compatibility authority until a frozen RED and explicit approval say otherwise.
- Shared files do not transfer ownership. Thus `serve` is RFC-0003 while co-dispatched `messages` remains unresolved; `hey` is KERNEL5 while `send`, `health`, `reply`, and `rp` remain unresolved.
- A direct tmux facade is RFC0002-TMUX, but a workflow that happens to call tmux is not automatically tmux administration. `new`, `done/finish`, `sleep`, `run`, and similar lifecycle/workflow routes remain unresolved.
- Aliases are grouped only when source points to the identical file fragment and handler. Similar names or shared helpers do not create alias equivalence.
- `APPROVED-REMOVE` has zero rows: no route is removable merely because it is hidden, test-facing, old, unsafe, or missing from help.

## Blocking rule

RFC-0001 route acceptance is **blocked** while any production spelling is `COMPAT/UNRESOLVED`, any generated spelling is absent/duplicated, any nested router lacks a reviewed disposition, or any owner can be reached both natively and through a plugin. `APPROVED-REMOVE` additionally requires a linked decision, frozen compatibility/RED evidence, and downstream source-removal proof. Re-run the parser whenever the frozen source commit changes; count equality alone is insufficient because handler/file ownership must also match.

## Nested router truth

These are source-recognized multi-action selector tokens beneath a generated top-level entry. Universal help flags, ordinary positional/flag grammars, and a lone plan-harness `constants` fast path are not promoted into a family; passthrough/default branches are called out because they are not registered child routes. A parent disposition does not silently decide a mixed or independently routed child.

### Nested `agents`
- `gc` (default path is list)

### Nested `artifacts`
- `ls|list` (default), `get|show`

### Nested `auth`
- `sign-v1`, `sign-headers`, `verify-v1`, `verify-legacy-from`, `verify-v3-from`, `from-sign-payload`, `hmac-sign`, `hmac-verify`, `constants`, `sign-v3`, `verify-request`, `loopback`, `from-address`, `hash-body`

### Nested `bg`
- `ls|list`, `tail`, `attach`, `kill`, `gc`; any other first token is treated as the spawned program, not a registered child route

### Nested `channel`
- `ls|list` (default), `providers`, `test`, `add`, `rm|remove`, `setup`, `migrate`

### Nested `check`
- `tools` (also the default; any other token is an error result from `check_run`)

### Nested `codex`
- `accounts` only

### Nested `completions`
- `commands` (`--describe` variant), `subs`, `oracles`, `windows`, `squads`, `fleet`, `oracle`, `zsh`, `bash`, `fish`

### Nested `config`
- `show` (default), `sources`, `explain`, `set`

### Nested `consent`
- `list` (default), `list-trust`, `approve`, `reject`; `trust|untrust` are recognized but explicitly refused native branches

### Nested `federation`
- `status`, `sync`

### Nested `feed`
- `parse-line|--parse-line`, `describe|--describe`, `active|--active`; `constants` is the plan-contract fast path

### Nested `fleet`
- `token`; roster intercepts `create`, `show`, `status`, `remove`, `leave`, `join`; main parser accepts `add`, `ls|list|census`, `doctor`, `gc|garbage-collect`, `init`, `health`, `consolidate`, `resume`, `sync`, `wake|wake-all`, `sleep`, `gather`, `renumber`

### Nested `fuzzy`
- `distance|--distance`, `match|--match`; `constants` is the plan-contract fast path

### Nested `identity`
- `session-name|--session-name`, `node|node-identity|--node-identity`; `constants` is the plan-contract fast path

### Nested `inbox`
- `pending|queue`, `show-pending|pending-show`, `approve`, `reject`, `list|ls` (default), `read`, `show`, `write`, `status`, `drain`

### Nested `more`
- `codex`, `status`

### Nested `oracle`
- `ls|list` (default), `fleet` (deprecated alias path), `scan`, `stale`, `prune`, `register`, `recruit`, `search|find`, `about`, `set-nickname|nickname`, `get-nickname`

### Nested `pair`
- `generate`, `accept`; otherwise `<url> <code>` is the pair action grammar, not a child name

### Nested `pair-api`
- `generate`, `probe`, `accept`, `status`; `constants` is the plan-contract fast path

### Nested `pair-code-store`
- `register`, `lookup`, `consume`; `constants` is the plan-contract fast path

### Nested `pane`
- `swap` only

### Nested `peer-probe`
- `classify`, `constants`, `format`, `handshake`, `handshake-constants`

### Nested `peers`
- `add`, `list|ls`, `info`, `remove|rm`, `forget`, `probe`, `probe-all`, `map`, `accept`

### Nested `plugin`
- `ls|list`, `init`, `install`, `infer-capabilities`, `create|scaffold`, `info`, `enable`, `disable`, `remove|rm|uninstall`, `build`, `dev`

### Nested `plugin-artifact`
- `contract`, `plan`

### Nested `plugin-manifest`
- `parse`, `load`, `discover`, `import-symbol`, `invoke`

### Nested `plugin-scaffold`
- `validate-name`, `manifest`; `constants` is the plan-contract fast path

### Nested `plugins`
- `ls|list` (default), `info`, `remove|rm`, `nuke`, `enable`, `disable`, `lean`, `standard`, `full`

### Nested `policy`
- `constants`; otherwise action is selected by flags, not a child name (`plugin-policy` uses the same handler)

### Nested `profile`
- `list|ls`, `current|active`, `show|info`, `use|set`

### Nested `project`
- `learn`, `incubate`, `find|search`, `list`

### Nested `schedule`
- `add`, `ls`, `rm`, `sync`, `peek`, `pause`, `resume`, `logs`, `cost`, `run`, private `fire`, private `exec`

### Nested `scope`
- `list|ls`, `create|new`, `show|info`, `delete|rm|remove`

### Nested `serve`
- default start; `status|--status`; `stop`

### Nested `setup`
- `auto-wake` only

### Nested `snapshots`
- `list` (default), `create`, `show` (a lone other name is show-by-name)

### Nested `squad`
- `token` → `set`; `import`

### Nested `tab`
- `new`; other leading positionals are tab indices/list-or-message grammar, not registered child names

### Nested `team`
- 43 accepted names: `create new list ls status tasks oracle-members members lives history plan preflight check load spawn spawn-from send msg broadcast inbox invite adopt release up bring apply reassign liveness live-check down remove delete rm prune gc shutdown resume enter send-enter add task done assign`

### Nested `tmux`
- 14 spellings / 11 groups: `ls|list`, `peek`, `split`, `attach`, `break`, `close|unsplit`, `kill`, `layout`, `open`, `pipe|pipe-pane`, `sync`

### Nested `token`
- `list|ls|tokens`, `current`, `resolve`, `status`, `use`, `apply`, `save`, `load`, `scan`

### Nested `tonk`
- `gh`, `say`, `status`; `gh` → `whoami|discuss`; `discuss` → `create|read|post|reply`

### Nested `transport`
- `constants`; otherwise action is selected by flags

### Nested `trust`
- `list|ls`, `add|pin|trust`, `remove|rm|delete|revoke` (`trusts` is a top-level wrapper to the same handler)

### Nested `user-setup`
- `projects audit` is the explicit nested audit branch; otherwise the default prune plan runs

### Nested `wave`
- `status` (default), `start`, `dispatch`, `teardown|down`

### Nested `workspace`
- `ls|list` (default; `ws` is the top-level alias)

### Nested `worktree`
- `ls` (default), `clean`, `add`

### Nested `x`
- `ls`, `gc`, `rm`, `trust` → `ls|revoke`; any other first token is parsed as a plugin spec, not a registered child route

### Nested `xdg`
- `paths`, `core-paths`, `validate-instance`; `constants` is the plan-contract fast path

### Nested `zai`
- `status`, `mon`, `test`

## Exact nested `tmux` ownership rows

| Spellings | Handler | Registry owner |
|---|---|---|
| `ls` / `list` | `run_tmux_ls` | `core_impl/tmux_dispatch.rs` `built-in` |
| `peek` | `run_tmux_peek` | `core_impl/tmux_dispatch.rs` `built-in` |
| `split` | `run_tmux_split` | `core_impl/tmux_dispatch.rs` `built-in` |
| `attach` | `run_attach_plan` | `core_impl/tmux_dispatch.rs` `built-in` |
| `break` | `run_tmux_break_command` | `core_impl/tmux_break.rs` `TMUX_SUB_280` |
| `close` / `unsplit` | `run_tmux_close_command` | `core_impl/tmux_close.rs` `TMUX_SUB_281` |
| `kill` | `run_tmux_kill_command` | `core_impl/tmux_kill.rs` `TMUX_SUB_282` |
| `layout` | `run_tmux_layout_command` | `core_impl/tmux_layout.rs` `TMUX_SUB_283` |
| `open` | `run_tmux_open_command` | `core_impl/tmux_open.rs` `TMUX_SUB_284` |
| `pipe` / `pipe-pane` | `run_tmux_pipe_command` | `core_impl/tmux_pipe.rs` `TMUX_SUB_285` |
| `sync` | `run_tmux_sync_command` | `core_impl/tmux_sync.rs` `TMUX_SUB_286` |

The nested registry is separate from similarly named top-level commands. In particular, nested `peek`, `split`, `attach`, `break`, and `kill` are not assumed aliases of their top-level spellings.
