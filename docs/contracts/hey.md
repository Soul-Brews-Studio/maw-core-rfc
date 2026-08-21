# `maw hey` native command contract

- **Status:** Proposed compatibility contract
- **Parent:** [RFC-0001](../RFC-0001-minimal-maw-kernel.md)
- **Source baseline:** `maw-rs@775c709b67add2a69c7c2dd3ecee18bbb5fcdb6e`
- **Source tree:** `2e66f58be215d5151e8326a43a9b034b96f6be11`

`hey` is the reserved native message capability. It resolves a local or federated target, binds sender
identity, optionally applies native ACL/trust policy, submits exact content to a pane or inbox, and
records message evidence. Optional plugins MUST NOT intercept it or receive transport secrets.

## Source authority and ownership

| Concern | Current owner |
|---|---|
| public route/parser/linear dispatch | `crates/maw-cli/src/core_impl/send_federation.rs` |
| target picker/resolution | `core_impl/hey_route_picker.rs` and native route helpers |
| identity, signature, display prefix | `core_impl/sender_identity.rs` |
| peer ACL/approval/trust | `core_impl/send_acl_gate.rs` |
| merged node/peer/agent config | `core_impl/federation_config.rs` |
| `hey log` | `core_impl/hey_log.rs` |
| local pane transport | `maw-tmux` through native `TmuxClient::send_text` |
| peer transport | native signed `/api/send` client and `maw-auth` |
| focused tests | `core_impl/send_acl_hotpath_tests.rs`, send/route/federation tests |

The same source file also dispatches `send`, `health`, `reply`, and `rp`. They require independent
route dispositions; retaining `hey` does not retain every sibling by source-file accident.

## Grammar

```text
maw hey <target> <message...> [--inbox|--no-inbox]
  [--from <oracle:node>] [--approve] [--trust] [--dry-run]
maw hey <target> -f <file>
maw hey <target> -
maw hey log [--since T] [--from X] [--suspicious] [-n N]
```

Observed message parser:

| Input | Behavior |
|---|---|
| target + words | joins all words after target with one ASCII space |
| `-f FILE` | exact UTF-8 file content, no shell expansion |
| lone `-` | exact UTF-8 stdin content |
| `--inbox`, `--no-inbox` | last one wins; inbox-only versus pane/default |
| `--from V`, `--from=V` | claimed wire identity, later checked against signer |
| `--approve` | bypass/approve peer ACL for this send |
| `--trust` | requires approve; attempts persistent sender-target trust |
| `--dry-run` | resolve and render route before delivery gate |
| `--help`, `-h` | usage on stdout, exit `0` |

File/stdin and positional message sources are mutually exclusive. Missing target/message/value,
multiple sources, unreadable/non-UTF-8 input, unknown flags, `--trust` without approve, and
whitespace-only content fail. There is no `--` separator. File/stdin reads currently have no explicit
byte cap; bounding them is a deliberate security change.

Help advertises deprecated `--force`, but the actual parser does not accept it. Freeze that conflict
as a RED before correcting either side.

## Message content and identity

- A positional message is joined; file/stdin content otherwise passes unchanged.
- Empty-after-trim content is refused before any delivery.
- Text beginning `[` is reserved for signed transport prefixes and is refused.
- Text beginning `/` is sent without a human prefix; other text becomes `[node:oracle] <text>`.
- `--from` must validate as the exact configured signing identity `oracle:node`; it cannot forge a
  different sender.
- Native code may add an Ed25519 signature when a peer key exists. Missing local signing key can
  yield an unsigned local record; remote delivery separately requires federation credentials.

Message bytes are operator authority. Any future internal composition uses an exact host-issued
ContentRef; guest code cannot substitute, prefix, truncate, or replay them.

Sender-oracle precedence is explicit `--from`, current `TMUX_PANE` window, cwd Oracle marker,
`MAW_SESSION_WINDOW`, configured Oracle, focused in-tmux window, then headless cwd fallback. Peer
wire identity additionally honors `MAW_SENDER`; current local display formatting sees CLI `--from`
but not `MAW_SENDER`, so local display and peer/audit identity can disagree. This is frozen debt.

## Route selection

1. Load merged `node`, `oracle`, `peers`, `namedPeers`, and agent-to-node mappings.
2. Inventory tmux sessions; failure is terminal and distinct from an empty inventory.
3. For bare, non-path, non-self targets, run the typed picker. Ambiguity uses frozen TTY/non-TTY
   picker behavior rather than an arbitrary winner.
4. `peer:<node>:<target>` forces peer routing before local matching.
5. Otherwise resolve `local:<target>`, exact tmux forms, self aliases, configured agents/peers, and
   canonical `<node>:<session>[:<window>]` forms.
6. For `hey`, a target missing from namedPeers may fall back to native `peers.json`.
7. Prefer pane zero for the frozen ambiguous-agent row.
8. If a node name collides with a local session and peer, refuse **before** dry-run. The operator must
   choose explicit local or `peer:` form.
9. A cross-node name that resolves back to this node is refused unless the query explicitly used
   `local:`. Bare intentional local/self aliases remain local.

Route/peer URLs, current session, local pane ids, config, peer keys, tokens, and network addresses stay
native. A plugin cannot invent or enumerate them.

## Dry-run

Dry-run resolves inventory, picker, peer fallback, collision, and route errors. It then prints exactly
one local, self-node, or `peer <node> <target> via <url>` line and performs no pane/inbox/network/
message-sink mutation. Collision refusal still fails. Current dry-run runs before the empty-body and
self-node delivery gate, so those rows can render success-shaped route plans; preserve until approved.

## Delivery branches

### Local pane (default or `--no-inbox`)

1. Optionally warn when the resolved pane does not look agent-shaped.
2. validate/sign and format the outbound text;
3. call native `TmuxClient::send_text`, including its confirmation semantics;
4. on confirmed send, record configured message sinks/audit;
5. print `delivered → <target>: <outbound>`; warnings use stderr.

No receiver inbox is written by default. “Delivered” proves native pane submission confirmation, not
that an agent consumed or acted on the message.

### Local `--inbox`

Resolve the target Oracle repository, write a timestamped file under `<repo>/ψ/inbox`, skip pane
injection, and print `queued inbox <oracle> <filename>`. Unknown local Oracle or write failure is
terminal. This branch bypasses prefix/signature validation and success sinks, so bracket-prefixed
text may be stored even though pane/peer delivery rejects it. The exact path and filesystem authority
remain native.

### Peer

1. Evaluate native scope ACL; proceed, queue pending, or reject.
2. Resolve/validate wire sender, signature, peer key, federation token, timestamp, URL/address set.
3. POST the exact request to `/api/send` through the native 5-second no-secret-to-guest client.
4. Only after accepted response, record message sinks and print `<state|queued> <node:target>`.
5. Surface receiver non-agent warnings on stderr; transport/auth/receiver failure is exit `1`.

`inbox:true` asks the receiver for inbox-only delivery; default/false asks pane delivery. A successful
HTTP response is the receiver's stated outcome, not proof of agent consumption.

## ACL, approval, and trust

Peer ACL is native authority. With no scopes, it allows. A Queue decision writes a pending message and
returns a queued result without network delivery. `--approve` proceeds; `--approve --trust` attempts
to persist the exact sender-target trust pair before delivery.

**Observed fail-open debt:** ACL read/evaluation failure, pending-queue write failure, and trust-add
failure warn and proceed. This is current behavior, not the target security posture. Any move to
fail-closed requires RED fixtures, explicit approval, and preservation of truthful queued/partial
state. Plugins never receive a trust-store mutation capability or an ACL bypass.

## Audits and mutation ordering

Async `hey` does not use the generic synchronous dispatcher audit. Its success path records message
sink evidence containing command, argv audit view, normalized sender, target, outbound text, route,
and signature presence. Failed/refused/dry-run sends do not claim a success record. `hey log` reads
and correlates audit JSONL with message events; it does not deliver.

Observed sinks are best effort: raw argv (including message or file path) enters audit JSONL,
`maw-log.jsonl` truncates outbound content to 500 characters, optional MQTT publication is
synchronous, and the SQLite ledger may be disabled. Sink errors never change delivery success.

Mutation order is route-specific:

- local pane: tmux confirmation, then sink records;
- local inbox: inbox file only;
- queued peer ACL: pending file only;
- approved trust: trust write attempt, network delivery, then sink records;
- normal peer: network delivery, then sink records.

A later sink failure is best effort and does not revoke delivery. Never retry an ambiguous pane or
network mutation automatically.

## `hey log` contract

Defaults to the last 20 correlated rows. `--since` and `--from` accept separated/equals values;
`--suspicious` filters rows with reasons; `-n N|-n=N` selects a validated limit. Unexpected positional
or unknown flags fail with exit `2` and usage; help is exit `0`. It silently drops unreadable/invalid
JSONL, compares `--since` timestamps lexically, and renders TSV or `No hey log entries.`. Bare target
`log` is reserved for this subcommand; use an explicit qualified target to message an agent named
log. No delivery or JSON mode is implied.

## Output and exit contract

- Help: `0`, stdout. Normal confirmed/queued/dry-run: `0`, stdout, optional warning stderr.
- `hey` parse, route, identity, ACL reject, tmux, inbox, auth, network, or receiver failure: `1`.
- Candidate/refusal/queued/delivered strings, ANSI warnings, sender prefix, target normalization, and
  newline placement are frozen fixture bytes. `hey` has no JSON output mode.

## Known decisions requiring RED + approval

1. reconcile documented `--force` with the rejecting parser;
2. cap file/stdin/message and route-display bytes;
3. change ACL/trust/queue errors from fail-open to fail-closed;
4. define message-sink failure reporting after confirmed delivery;
5. distinguish submission confirmation from end-to-end consumption in wording without breaking bytes;
6. adjudicate dry-run before empty/self delivery gate; and
7. classify `send`, `health`, `reply`, and `rp` independently before source movement.

## Acceptance matrix

- Kernel availability with plugins absent/corrupt/refused/shadowing.
- Help, positional/file/stdin exact bytes and exclusivity, empty/non-UTF-8/unreadable/unknown inputs,
  inbox/no-inbox, from, approve/trust, dry-run, and stale force claim.
- Picker exact/fuzzy/ambiguous/cancel, local/pane-zero/self/local-prefix/canonical peer/forced peer,
  peers.json fallback, local-peer collision, tmux-unreachable, and unknown target.
- Local confirmed/failed/ambiguous pane send, non-agent warning, slash/bracket prefix, signature/from.
- Local inbox resolve/write and no-pane proof.
- ACL allow/queue/approve/trust plus each fail-open failure; peer key/token/address/HTTP/receiver rows.
- Audit/message-sink ordering, no-success-record failures, `hey log` filters/limits/correlation.
- Exact stdout/stderr/exit goldens and failure injection after each confirmed mutation.
