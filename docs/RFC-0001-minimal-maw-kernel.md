# RFC-0001: Minimal maw Native Kernel

- **Status:** Proposed
- **Tracking:** [maw-core-rfc#1](https://github.com/Soul-Brews-Studio/maw-core-rfc/issues/1)
- **Implementation program:** [maw-rs#963](https://github.com/Soul-Brews-Studio/maw-rs/issues/963)
- **Source baseline:** `maw-rs alpha@775c709b67add2a69c7c2dd3ecee18bbb5fcdb6e`
- **Source tree:** `2e66f58be215d5151e8326a43a9b034b96f6be11`

## Summary

Reduce `maw-rs` to a trusted orchestration kernel centered on five operator capabilities:

1. wake an Oracle;
2. attach to it (`attach`/`a`);
3. create or enter isolated work (`work`);
4. send it a message (`hey`); and
5. inspect pane output (`peek`).

The native binary also retains the plugin runtime, artifact verification, capability enforcement,
transport authentication, and bounded platform adapters needed to implement those capabilities.
Optional workflow UX and product/vendor policy move to source-proven external plugins.

This repository records the proposal. It does not duplicate the implementation task graph already
reviewed in `maw-rs#963`. Current-state claims are source-derived; the extraction and target
multi-route ABI remain planning work, not code already shipped on `alpha`.

## Why

`maw-rs` currently combines a trusted host with optional workflow products. That makes the native
binary larger, grants product policy ambient native authority, and makes independent plugin delivery
look complete while native dispatch still shadows it. A lean kernel makes the security boundary
legible: native code owns authority; plugins own optional policy.

The change is an extraction, not a rewrite. Current behavior remains the compatibility authority
unless an individually frozen and approved correction says otherwise.

## Current source map

The source snapshot is the authority; Serena's Rust index was used only to discover and cross-check
the repo-relative symbols below.

| Surface | Current native owner | Target |
|---|---|---|
| dispatch/plugin fallback | `core_impl/dispatcher.rs` | native |
| `wake` | `core_impl/wake.rs` | native |
| `attach` / `a` | `core_impl/attach.rs` | native |
| `work` and worktree/session orchestration | `core_impl/workspace_scaffold_commands.rs::work_run_command`, `core_impl/workon.rs` | native |
| `hey` | `core_impl/send_federation.rs` | native |
| `peek` | `core_impl/tmux_peek.rs` | native |
| `bring` / `b` | `core_impl/session_list_plan.rs::run_bring_plan` | plugin |
| `split` | `core_impl/split.rs` | plugin |
| `team` / `t` | `core_impl/team_core.rs` and Team modules | plugin |
| `codex accounts` | `core_impl/codex_accounts.rs` | plugin |
| `more` / `wave` Codex workflows | `core_impl/more.rs`, More/Wave modules | plugin |

The audited workspace contains 690 Rust files: 472 in `maw-cli`, including 229 `core_impl` files;
61 in `maw-plugin-manifest`; and 63 in `maw-tmux`. Of those, 295 are under test directories.

The merged #963 Spec Kit protects `wake`, `attach/a`, `hey`, `peek`, and `serve`, but does not yet
name `work` as a first-class retained invariant or separate serve from the minimal public inventory.
Implementation requires a reviewed upstream amendment to its constitution, ADR, ownership contract,
requirements, regressions, task graph, quickstart, and canary. Open PR #967 is not merged authority
and its canary also omits `work`.

## Native kernel contract

### `maw wake`

- Preserve picker/target selection, local/peer routing, list/all/dry-run, worktree plan/create/reuse,
  phase audit, session/window reuse/create, shell readiness, engine confirmation, attach/select,
  fleet registration, and hooks.
- Retain generic provider invocation but move vendor-specific planning behind accepted providers.
- Invoke a selected provider exactly once in a fresh plan-only instance before phase, filesystem,
  worktree, or tmux mutation; revalidate its target/config snapshot before execution.
- Refuse a missing, invalid, or refused explicitly selected provider before workflow mutation.
- Preserve non-Codex behavior when optional plugins are absent.
- Preserve currently inert `--split` behavior unless a separate RED and human approval changes it.

### `maw attach` / `maw a`

- Retain typed local/remote target resolution and the binary attach fast path.
- Preserve picker, sleeping wake plan, remote SSH plan, `--print`, `--readonly|--read-only|-r`,
  `--plan-json|--dry-run`, `--yes|-y`, `--ssh-alias`, repeated `--alive`, and exact errors.
- Freeze `main.rs::maybe_exec_attach`: only an interactive, unambiguous live target may attach fast;
  list failure falls through. **Observed conflict:** top-level `attach/a` then collapses list failure
  to an empty set, while nested `maw tmux attach` reports unreachable. Resolution needs RED + approval.
- **Observed ordering debt:** `main_code_async` currently constructs the tmux client and lists
  sessions before `maybe_exec_attach_with` checks the verb, TTY, or bypass flags, so even unrelated
  commands touch `tmux` from `PATH`. A verb/eligibility-before-I/O correction needs RED + approval.
- Never permit a plugin to shadow or intercept attach.

### `maw work`

- Preserve grammar `repo [task]`, `--wt`, `--fresh`, `--name`, `-e|--engine`, `--layout`, repo/ghq
  forms, create/reuse/collision, session/window, fleet, engine, attach, output, and error behavior.
- Expose only opaque, validated worktree references to plugins that need lifecycle composition.
- Keep raw Git, filesystem, and tmux operations native.
- Keep public `workon` until a separate frozen disposition is approved: `work` delegates to it and
  `wake` shares helpers, so shared-source deletion cannot silently remove it.

### `maw hey`

- Preserve positional/file/stdin exclusivity, picker/forced-peer routing, local-peer collision
  refusal, local/inbox/peer delivery, signatures/tokens, audit, trust/approval, dry-run ordering,
  and empty-body/implicit-self refusal.
- State-backed Team mailbox policy may be external, but transport authority stays native.
- Never claim delivery across an ambiguous or unconfirmed boundary.
- Co-dispatched `send`, `health`, `reply`, and `rp` require independent dispositions.

### `maw peek`

- Preserve default 30 lines, history/line validation, one-target maximum, help versus argument exits,
  overview/target resolution, canonical/raw fallback, duplicates, blank capture, unreachable/missing
  distinctions, and argv-vector injection safety.
- Plugins may request reviewed typed observations; they do not receive unrestricted capture access.
- **Observed debts:** any positive `u32` line count is accepted and `--history` captures full history;
  remote capture ignores parsed lines/history; response/target bytes have no finite cap.
- Peer/local collision guidance advertises unsupported `--peer`; `./...` forces local but no working
  forced-remote escape exists. Local-list failure maps to empty and may choose remote without proving
  absence. Each correction requires a frozen RED and approval.
- Native `peek` may return requested pane text; plugin observation returns only declared boolean
  marker results and never captured text.

### Native infrastructure

The kernel additionally owns plugin discovery, manifest parsing, immutable artifact/pin verification,
ABI negotiation, intent-scoped capabilities, audit, consent/trust decisions, secrets, and bounded
tmux/process/filesystem/network adapters.

## External plugin contract

The external `Soul-Brews-Studio/maw-plugins` repository owns:

- `bring`/`b` as a capability-free plan/render plugin;
- `split` over one closed typed split operation;
- `team`/`t` and confirmed Team-only companion routes;
- Codex account presentation, `more`, `wave`, health/profile/resume policy, and provider planning; and
- future optional workflows that do not establish host authority.

A plugin may own parsing, rendering, selection, and workflow policy. Inert validated plan/render
data may remain plain data, but every authority-bearing effect parameter consumes an intent-bound
opaque reference issued from reviewed operator input or host-owned state.

## Security requirements

1. No guest receives arbitrary process execution, raw environment, secrets, `/proc`, unrestricted
   filesystem paths, unrestricted tmux, arbitrary Git roots, or a generic CLI escape hatch.
2. Each effectful invocation receives the intersection of static manifest capabilities and a
   hash-covered route/subcommand intent with finite call budgets.
3. Authority-bearing targets, payloads, engines, workspaces, members, content, Git refs, and
   consent/trust outcomes are host-issued references, not trusted guest strings.
4. Non-idempotent actions have explicit confirmed/failed/ambiguous outcomes and cannot be replayed.
5. Guest-writable state cannot launder later execution authority; authority-bearing changes use
   closed semantic native mutations and durable provenance records.
6. Provider-planning instances receive no workflow mutation imports or route re-entry.
7. Missing, stale, malformed, hash-invalid, SDK-incompatible, or under-capable artifacts fail closed
   with actionable repair guidance.

## Compatibility requirements

- Freeze argv, aliases, help, stdout, stderr, exit codes, JSON, mutation ordering, and error rows
  before each extraction.
- Preserve maw-js fixtures; do not delete fixtures to make a cutover pass.
- Use client-first ordering for every cross-surface ABI change: land an additive compatible client,
  prove it against the preceding host, then enable or tighten the host.
- Never leave both native and plugin owners reachable after cutover.
- Help, completions, plugin list, doctor, registry, and issue ledger must agree on ownership.
- Maintain an exhaustive route ledger: every current dispatcher entry is classified as one of the
  five public commands, host administration, RFC-0002 tmux, RFC-0003 serve, external plugin,
  compatibility alias, or approved removal. An unclassified route blocks completion.
- Current alpha has legacy `cli.command` plus native-first fallback; planned route tables and
  canonical `invokedCommand` are future ABI work and must follow client-first ordering.

## Artifact-first migration

For each surface:

1. inventory native callers, behavior, side effects, capabilities, and fixtures;
2. land narrow generic host/SDK prerequisites without product policy;
3. build the external source with a pinned toolchain and committed lockfile;
4. require two clean byte-identical builds matching the committed `plugin.wasm` and SHA-256 pin;
5. run external native fake-host tests, WASM tests, policy checks, and direct committed-byte invocation;
6. merge the artifact and record its immutable source/tree/artifact evidence;
7. create a fresh downstream branch from current `origin/alpha`;
8. install the accepted artifact in parity tests, atomically switch ownership, and run full gates; and
9. delete inert native product source in bounded mechanical follow-ups.

`wake`, `attach/a`, `work`, `hey`, and `peek` regression failures block every cutover.

## Adjacent specifications

This RFC intentionally does not make every privileged native surface part of the small public
kernel:

- **RFC-0002** specifies the complete `maw-tmux` session/window/pane substrate and the bounded host
  operations used by both native commands and plugins.
- **RFC-0003** specifies `maw serve`, the God HTTP/WebSocket gateway, daemon lifecycle, exposure,
  authentication, and browser-equivalent canary. Serve remains native, but it is reviewed and
  released as a separate product/security boundary rather than a sixth kernel command.

## Release and canary

The exact final trees are frozen before evidence collection. Before a CalVer alpha tag:

- install the exact candidate and verify `maw --version` reports its commit;
- exercise retained native commands and installed/missing/refused plugin paths;
- bind all results to the candidate tree, registry, and artifact hashes.

The RFC-0001 canary exercises exactly the five public commands. A combined product release may
compose RFC-0002 and RFC-0003 evidence, but this RFC's acceptance never depends on serve. Promotion
must map candidate SHA to tagged merge SHA and prove identical trees plus the installed version.

## Non-goals

- Moving the plugin host, raw tmux adapter, transport authentication, or secrets into WASM.
- Reimplementing `wake`, `attach/a`, `work`, `hey`, or `peek` in plugins.
- Defining serve/God HTTP/WS behavior or the full public/raw tmux surface.
- Creating a second implementation tracker that competes with `maw-rs#963`.
- Bundling unrelated feature redesigns into extraction PRs.

## Acceptance

The proposal is complete when current-alpha source evidence supports the ownership table, independent
review approves the capability and compatibility boundaries, and `maw-rs#963` remains the sole
implementation program. The implementation is complete only when all extracted routes have one
reachable external owner, prohibited product code is absent from the native boundary, every artifact
and gate is verified, retained native commands pass, and the exact-candidate canary succeeds before
the alpha tag.
