# RFC-0001: Minimal maw Native Kernel

- **Status:** Proposed
- **Tracking:** [maw-core-rfc#1](https://github.com/Soul-Brews-Studio/maw-core-rfc/issues/1)
- **Implementation program:** [maw-rs#963](https://github.com/Soul-Brews-Studio/maw-rs/issues/963)
- **Source baseline:** `Soul-Brews-Studio/maw-rs` `alpha@775c709b`

## Summary

Reduce `maw-rs` to a trusted orchestration kernel centered on five operator capabilities:

1. wake an Oracle;
2. attach to it (`attach`/`a`);
3. create or enter isolated work (`work`);
4. send it a message (`hey`); and
5. inspect bounded output (`peek`).

The native binary also retains the plugin runtime, artifact verification, capability enforcement,
transport authentication, and bounded platform adapters needed to implement those capabilities.
Optional workflow UX and product/vendor policy move to source-proven external plugins.

This repository records the proposal. It does not duplicate the implementation task graph already
reviewed in `maw-rs#963`.

## Why

`maw-rs` currently combines a trusted host with optional workflow products. That makes the native
binary larger, grants product policy ambient native authority, and makes independent plugin delivery
look complete while native dispatch still shadows it. A lean kernel makes the security boundary
legible: native code owns authority; plugins own optional policy.

The change is an extraction, not a rewrite. Current behavior remains the compatibility authority
unless an individually frozen and approved correction says otherwise.

## Current source map

The map below was rechecked with Serena's Rust symbol index.

| Surface | Current native owner | Target |
|---|---|---|
| dispatch/plugin fallback | `core_impl/dispatcher.rs` | native |
| `wake` | `core_impl/wake.rs` | native |
| `attach` / `a` | `core_impl/attach.rs` | native |
| `work` and worktree/session orchestration | `core_impl/workon.rs`, workspace/worktree modules | native |
| `hey` | `core_impl/send_federation.rs` | native |
| `peek` | `core_impl/tmux_peek.rs` | native |
| `bring` / `b` | `core_impl/session_list_plan.rs::run_bring_plan` | plugin |
| `split` | `core_impl/split.rs` | plugin |
| `team` / `t` | `core_impl/team_core.rs` and Team modules | plugin |
| `codex accounts` | `core_impl/codex_accounts.rs` | plugin |
| `more` / `wave` Codex workflows | `core_impl/more.rs`, More/Wave modules | plugin |

The audited workspace contains 690 Rust files: 472 in `maw-cli`, including 229 `core_impl` files;
61 in `maw-plugin-manifest`; 63 in `maw-tmux`; and 295 Rust test files.

The merged #963 Spec Kit protects `wake`, `attach/a`, `hey`, `peek`, and `serve`, but does not yet
name `work` as a first-class retained invariant. Implementation therefore requires a reviewed
upstream amendment that adds `work` to the constitution, ownership contract, requirements,
regression matrix, task graph, quickstart, and exact-candidate canary before any extraction cutover.

## Native kernel contract

### `maw wake`

- Resolve identity, repository/worktree, engine, prompt, session, and window through native policy.
- Retain generic provider invocation but move vendor-specific planning behind accepted providers.
- Refuse a missing or invalid explicitly selected provider before workflow mutation.
- Preserve non-Codex behavior when optional plugins are absent.

### `maw attach` / `maw a`

- Retain typed local/remote target resolution and the binary attach fast path.
- Preserve picker, plan, readonly, port, and error behavior.
- Never permit a plugin to shadow or intercept attach.

### `maw work`

- Retain trusted repository discovery, branch/worktree creation, collision handling, session/window
  creation, and attachment.
- Expose only opaque, validated worktree references to plugins that need lifecycle composition.
- Keep raw Git, filesystem, and tmux operations native.

### `maw hey`

- Retain local/federated routing, authentication, audit, explicit delivery outcomes, and refusal.
- State-backed Team mailbox policy may be external, but transport authority stays native.
- Never claim delivery across an ambiguous or unconfirmed boundary.

### `maw peek`

- Retain bounded target validation, pane capture, window overview, truncation, and failure behavior.
- Plugins may request reviewed typed observations; they do not receive unrestricted capture access.

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

A plugin may own parsing, rendering, selection, and workflow policy. It cannot mint host authority.
Every effect consumes an intent-bound opaque reference issued from reviewed operator input or
host-owned state.

## Security requirements

1. No guest receives arbitrary process execution, raw environment, secrets, `/proc`, unrestricted
   filesystem paths, unrestricted tmux, arbitrary Git roots, or a generic CLI escape hatch.
2. Each effectful invocation receives the intersection of static manifest capabilities and a
   hash-covered route/subcommand intent with finite call budgets.
3. Targets, payloads, engines, workspaces, members, content, Git refs, and consent/trust outcomes are
   host-issued references, not trusted guest strings.
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
- run RFC-0003's browser-equivalent serve/God WebSocket canary independently; and
- bind all results to the candidate tree, registry, and artifact hashes.

An HTTP `101` or a bare `PASS` line is insufficient evidence. Evidence produced by changing the
candidate after review does not prove the reviewed candidate.

## Non-goals

- Moving the plugin host, raw tmux adapter, transport authentication, or secrets into WASM.
- Reimplementing `wake`, `attach/a`, `work`, `hey`, or `peek` in plugins.
- Creating a second implementation tracker that competes with `maw-rs#963`.
- Bundling unrelated feature redesigns into extraction PRs.

## Acceptance

The proposal is complete when current-alpha source evidence supports the ownership table, independent
review approves the capability and compatibility boundaries, and `maw-rs#963` remains the sole
implementation program. The implementation is complete only when all extracted routes have one
reachable external owner, prohibited product code is absent from the native boundary, every artifact
and gate is verified, retained native commands pass, and the exact-candidate canary succeeds before
the alpha tag.
