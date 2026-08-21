# `maw peek` native command contract

- **Status:** Proposed compatibility contract
- **Parent:** [RFC-0001](../RFC-0001-minimal-maw-kernel.md)
- **Source baseline:** `maw-rs@775c709b67add2a69c7c2dd3ecee18bbb5fcdb6e`
- **Source tree:** `2e66f58be215d5151e8326a43a9b034b96f6be11`

`peek` is the reserved native pane-reading capability. It resolves a local or federated target and
returns pane text to the operator. Optional plugins MUST NOT shadow it or gain the same text-capture
authority through the narrow marker-observation API proposed for workflow guests.

## Source authority

| Concern | Current owner |
|---|---|
| public parse, local resolution, capture, overview | `crates/maw-cli/src/core_impl/tmux_peek.rs` |
| peer detection, signing, HTTP capture | `core_impl/peek_federation.rs` |
| typed local target resolution | native tmux target resolver and `maw-tmux::TmuxRunner` |
| peers and signed federation transport | native peers store, `maw-auth`, and native curl adapter |
| distinct nested command | `core_impl/tmux_dispatch.rs` (`maw tmux peek`) |
| focused tests | `tmux_peek.rs` and `peek_federation.rs` test modules |

Serena was used to discover the named symbols; this immutable source snapshot is the authority.

## Grammar and validation

```text
maw peek <tmux-target> [--lines N] [--history]
maw peek <peer>:<target> [--lines N]
maw peek ./<target>
maw peek [--lines N]
```

The parser accepts at most one positional target, `--lines N`, `--lines=N`, `--history`, and
`--help|-h`. Default line count is `30`. Repeated line counts are last-value wins; history is sticky.
`N` is any positive `u32`, including values larger than the default. Zero, negative, missing,
non-numeric, overflow, unknown flags, a literal `--` separator, or a second target fails with exit
`2`. Help exits `0` on stdout.

A local target must be nonempty, unpadded, not begin with `-`, and contain no whitespace, NUL, or
control character. Target strings and capture output currently have no explicit byte bound. The
`--history` form requests full tmux history with `capture-pane -S -`; describing current output as
bounded would be false.

As a synchronous native route, dispatch best-effort records the generic command audit **before**
help, parse, and routing, including all failures. No JSON output mode exists.

## Route selection

With no target, `peek` is a local all-window overview. With a target:

1. If it has no nonempty `prefix:rest`, route locally without reading peers or listing sessions.
2. Otherwise look up the exact, case-sensitive `prefix` in native `peers.json`. Invalid, missing,
   unreadable, malformed, or unsafe-URL peer data silently falls back to local interpretation;
   merged-config `namedPeers` is not consulted.
3. For a known peer, list local session names solely to detect a peer/session collision.
4. If that exact local session exists, refuse as ambiguous. Otherwise fetch the peer's target.
5. A leading `./` bypasses peer routing and is stripped before local validation.

**Observed fail-open routing:** local session-list failure becomes an empty list, so a known peer is
selected when native code cannot prove that a same-named local session exists. This differs from a
fail-closed absence/unreachable contract and requires a RED fixture plus approval before changing.

**Observed unusable escape hatch:** the collision diagnostic recommends
`maw peek <peer>:<target> --peer <peer>`, but the parser has no `--peer` option and rejects it. `./`
works for forced local; there is no working forced-remote spelling. Preserve the contradiction as a
test until a correction is approved.

## Local target capture

Native code first resolves the operator target through the shared local tmux resolver. If resolution
fails it attempts one raw capture; if that also fails, it renders the frozen window-not-found form.
After a successful resolution it validates the resolved target and captures it with argv-vector
execution:

```text
capture-pane -p -t <target> -S -<lines> -J
capture-pane -p -t <target> -S - -J       # --history
```

No shell is involved. If a normalized resolved target returns blank content and differs from the raw
input, native code tries the raw target and prefers it only when nonblank. If resolved capture fails,
it also tries raw input but preserves the resolved capture error when both fail. This canonical/raw
fallback and its error precedence are compatibility behavior.

Success output is an ANSI cyan header followed by captured text:

```text
--- <resolved-target> ---
<pane text>
```

Trailing blank lines are removed while the last nonblank line's newline is preserved. An empty pane
therefore returns the header and newline. Success proves only that tmux returned capture bytes.

## Local overview

Bare `maw peek` runs `list-windows -a` with session, index, name, and active state. Malformed lines
are silently dropped; duplicate window rows are retained. For each parsed window it captures the
last three lines and renders only the literal final line, truncated to 80 Unicode scalar values.
If that final line is blank it renders `(empty)` even when an earlier captured line had text. Active
rows carry a green marker. Parsed overview `--lines` and `--history` are ignored.

An individual window capture failure renders `(unreachable)` and the overall overview still exits
`0`; empty capture renders `(empty)`. Failure of the initial `list-windows` is terminal exit `1`.
An empty parsed inventory returns empty stdout with exit `0`. These distinctions MUST be frozen
before any fail-closed normalization.

## Federated capture

For a selected peer, native code:

1. validates the configured peer URL;
2. percent-encodes the target into `/api/capture?target=...`;
3. signs the query-stripped `/api/capture` path with native identity material when available;
4. invokes validated curl argv with a ten-second maximum and `--` before the URL;
5. separates the HTTP status marker and parses JSON `{content}` or `{error}`; and
6. returns a cyan `<peer>:<target>` header plus the trimmed content.

The query target is encoded, but URL, signing identity, keys, tokens, clock, process argv, and HTTP
response parsing remain native. An unsignable node currently sends a bare request and relies on the
peer to accept or reject it. Key loading may first generate and persist the native peer key, then
still downgrade to bare when a later signing prerequisite is absent. HTTP 401/403 yields the
credential diagnostic; other non-2xx, invalid JSON, explicit error, or missing content yields `1`.

**Observed option loss:** remote capture sends only `target`. Parsed `--lines` and `--history` never
cross the wire. The current receiver uses its own resolved capture with a fixed 80-line behavior,
`-e`, and no local peek `-J`, so wrapping and ANSI behavior can also differ. Any correction needs
frozen current rows and an explicitly versioned server request.

The remote response body and returned pane text are currently not byte-bounded. Adding limits is a
security change with explicit truncation/error semantics, not silent parity.

## Output, exits, and mutation

| Row | Stdout/stderr | Exit |
|---|---|---:|
| help | usage on stdout | `0` |
| local/remote capture | ANSI header + text on stdout | `0` |
| overview | one row per parsed window, possibly empty | `0` |
| parse failure | diagnostic on stderr | `2` |
| target/resolve/list/capture/peer/auth/HTTP failure | diagnostic on stderr | `1` |
| overview per-window capture failure | `(unreachable)` row, no stderr | `0` |

`peek` is observational at the workflow level. It reads tmux, peer configuration, signing material,
and the network; it writes the best-effort generic audit and may create a missing native peer key
during remote signing preparation. It performs no pane input or topology mutation and never retries
a mutation.

## Security boundary and distinct commands

Top-level native `maw peek` deliberately exposes pane text to the operator. This does **not** imply a
plugin capture capability. A workflow guest may receive only manifest-declared, intent-scoped,
finite boolean marker observations against host-issued pane references; it never receives capture
text, raw targets, peer URLs, credentials, or general substring-oracle access.

`maw tmux peek` is a separate permissive raw-capture command with different parsing, errors, routing,
and output. It is not an alias. Its retain/rename/retire decision belongs to RFC-0002 and must be
resolved before claiming one canonical peek surface.

## Known decisions requiring RED + approval

1. add a working forced-remote spelling and correct the impossible collision hint;
2. decide peer routing when local inventory is unreachable;
3. forward or explicitly reject remote `--lines` and `--history`;
4. define target, local history, HTTP body, and rendered output bounds;
5. decide whether per-window overview failure remains success-shaped;
6. distinguish empty versus malformed window inventory; and
7. disposition the nested raw `maw tmux peek` contract.

## Acceptance matrix

- Kernel availability with optional plugins absent, corrupt, refused, or attempting to shadow.
- Help, default/repeated/separated/equals lines, history, zero/negative/overflow/missing/unknown,
  separator, target count, whitespace/control/injection-safe argv, and unbounded-history fixtures.
- Local exact/resolved/raw fallback, blank/duplicate target, miss, capture failure, and newline bytes.
- Overview empty/malformed/duplicate/active/empty-pane/unreachable-pane/list-unreachable rows.
- Peer absent/invalid URL, exact peer, local collision, local-list failure, forced local, impossible
  forced remote, unsigned/signed request, target encoding, auth/status/JSON/content failures.
- Remote option-loss fixtures and explicit no-pane/no-topology-mutation proof.
