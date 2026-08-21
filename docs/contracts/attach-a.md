# `maw attach` / `maw a` native command contract

- **Status:** Proposed compatibility contract
- **Parent:** [RFC-0001](../RFC-0001-minimal-maw-kernel.md)
- **Source baseline:** `maw-rs@775c709b67add2a69c7c2dd3ecee18bbb5fcdb6e`
- **Source tree:** `2e66f58be215d5151e8326a43a9b034b96f6be11`

`attach` and `a` are one reserved native capability. Plugins MUST NOT shadow either spelling, and a
plugin discovery/refusal failure MUST NOT prevent attach. Current behavior has two implementations:
a binary pre-dispatch fast path and the normal CLI handler. Their differences are part of this audit,
not silently treated as equivalent.

## Source authority

| Concern | Current owner |
|---|---|
| binary fast path | `crates/maw-cli/src/main.rs::{maybe_exec_attach,attach_exec_tmux_args}` |
| CLI parse/resolve/render/execute | `crates/maw-cli/src/core_impl/attach.rs` |
| shared action decision | `maw-tmux` `decide_tmux_attach_action` and attach resolution |
| nested, distinct command | `core_impl/tmux_attach.rs` (`maw tmux attach`) |
| focused tests | `core_impl/attach_private_tests.rs`, `tests/attach_alias_cli.rs`, `main.rs::tests` |
| fixtures | `tests/fixtures/{epic56,native-interactive}` attach rows |

## Grammar

Observed help advertises only:

```text
maw-rs attach <target> [--print] [--readonly|-r]
maw-rs a      <target> [--print] [--readonly|-r]
```

The actual normal parser accepts exactly one positional target and:

| Flag | Meaning |
|---|---|
| `--print` | render the local action instead of executing it |
| `--readonly`, `--read-only`, `-r` | request readonly local attach |
| `--plan-json`, `--dry-run` | render the JSON plan |
| `--yes`, `-y` | forward confirmation only in the remote attach-ssh plan |
| `--ssh-alias VALUE`, `--ssh-alias=VALUE` | override the rendered remote SSH alias |
| repeated `--alive VALUE`, `--alive=VALUE` | inject a caller-supplied live-session set |
| `--help`, `-h` | usage on stdout, exit `0` |

Missing target, extra positional, missing flag value, unknown flag, control-bearing/leading-dash
unsafe target, SSH alias, or alive token fails before attach execution. Parser failures exit `2`
with usage; later validation failures exit `1` without usage. Scalar flags are last-value wins;
alive values form a sorted set.

The fast path accepts only canonical/alias + target + readonly spelling. Help, print, JSON/dry-run,
non-TTY stdout, extra positional, unknown flags, ambiguous target, or missing target fall through to
the normal dispatcher. Valid normal flags such as `--yes`, `--alive`, and `--ssh-alias` are unknown to
the fast path and therefore force plan/dispatcher behavior.

## Binary fast path

`main_code_async` calls `maybe_exec_attach` before the CLI dispatcher. Observed order:

1. construct a local `TmuxClient` and run `list-session-names`;
2. on list failure, fall through;
3. only then check whether argv begins with `attach|a`, stdout is a TTY, flags are eligible, and the
   target resolves to one live session;
4. readonly always executes `tmux attach -r -t <session>`;
5. otherwise, inside tmux executes `tmux switch-client -t <session>` and outside executes
   `tmux attach -t <session>`;
6. return the child status directly without entering normal dispatch.

Thus every maw invocation currently probes `tmux` from `PATH` before verb eligibility. Moving the
verb/TTY checks before tmux I/O is desirable least authority but is a behavior/order correction that
requires a frozen RED and explicit approval.

Readonly takes precedence over inside-tmux switching. Preserve or explicitly adjudicate that row.

## Normal target resolution

1. Validate argv and target.
2. A target containing `:` is classified as explicit remote when both halves are nonempty and the
   suffix is not a numeric `window` or `window.pane` form.
3. Explicit remote returns a **plan only**; it does not run SSH. Default alias is the node unless
   `--ssh-alias` was supplied. `--yes` is forwarded. Current remote rendering ignores readonly.
4. Otherwise, when no `--alive` values were supplied, run native `list-sessions` and build the set.
5. Resolve typed candidates across live session/window names, registry aliases, sleeping registry,
   Oracle identity, and repo basename.
6. Prefer a unique raw live match or a single exact live candidate. Missing, bridge, or ambiguous
   results use the frozen non-TTY table/picker behavior; sleeping/Oracle rows may call native wake.
7. Validate the resolved session again and decide Print, SwitchClient, Attach, or Recover.

A nonnumeric local `session:window` can be classified as remote before local resolution. This is a
known ambiguity, not a supported forced-remote/local contract.

When any `--alive` value is supplied it replaces, rather than augments, the probed inventory.
Picker execution can bridge to native wake/fleet. The displayed Oracle wake row may include a
resolved `--repo`, but current execution drops that disambiguating repo; preserve as a known defect.

## Execution versus rendering

The normal handler passes `is_tty=false` to `decide_tmux_attach_action`. Therefore normal live local
invocations generally render a `Run: tmux ...` plan; actual interactive attach/switch normally occurs
only in the binary fast path. `--print` and `--plan-json|--dry-run` always render.

| Action | Text/JSON behavior | Exit |
|---|---|---|
| fast live attach/switch | exec child; no normal stdout | child status |
| normal Attach/Switch without render flag | execute only if action reaches live executor; otherwise frozen plan | child/action status |
| Print | `Run: tmux ...`, resolved row, detach hint | `0` |
| Recover | missing-session message + `maw wake <target> --attach` | `1` |
| explicit remote | Tier-3 dry-run + `maw-rs attach-ssh ...`, or JSON | `0` |
| parser failure | diagnostic + usage on stderr | `2` |
| validation/executor failure | diagnostic on stderr | `1` |

Local JSON fields are `command`, `alias`, `target`, `session`, `action`, and `tmuxArgs`. Remote adds
`tier`, `node`, `sessionName`, `sshAlias`, `yes`, and `attachSshArgs`. Current JSON always reports
`"command":"attach","alias":"a"`, even when the canonical spelling was invoked, because invoked
route identity is not passed to the handler. Preserve bytes until a separately approved correction.

Remote text/JSON is a plan, not evidence that SSH or tmux attached. Current remote readonly is not
represented in `attachSshArgs`; do not claim otherwise. With `--yes`, text appends `-y` but the JSON
`attachSshArgs` omits it even while `yes:true`.

## Error and ambiguity semantics

- `attach_list_sessions` currently maps runner failure to an empty set. The fast-path comment claims
  fallthrough preserves an unreachable error, but top-level normal attach can instead report missing/
  recovery. Nested `maw tmux attach` has stricter unreachable handling. This conflict requires RED
  tests and a human decision before fail-closed normalization.
- Ambiguous non-TTY resolution prints candidate/picker rows rather than choosing arbitrarily.
- A sleeping registry/Oracle candidate bridges through native `run_wake_command`; plugin absence must
  not affect it.
- Live execution success means the tmux attach/switch process accepted the action. It does not prove
  a later Oracle action.
- No mutation is retried after an ambiguous child/process outcome.
- As a synchronous native route, every invocation best-effort appends the generic dispatcher audit,
  including help and invalid argv; it records the actual `attach` or `a` spelling.

## Security boundary

Attach keeps raw tmux, PATH/process execution, registry paths, fleet state, SSH material, and wake
composition native. A plugin may receive inert display facts but cannot invoke attach, choose raw
argv, supply the live-session set, resolve an SSH alias, or obtain captured pane content.

The `--alive` input is a native compatibility/testing surface; it MUST NOT be exposed as guest-owned
truth. Any future typed caller receives an intent-scoped host-issued target ref and one-shot action.

## Known correction decisions

Each requires a native RED fixture and explicit approval:

1. check verb/TTY/flags before the fast path touches tmux;
2. make top-level listing failure terminal rather than empty/missing;
3. reconcile fast and normal resolvers/TTY behavior;
4. distinguish invoked `attach` versus `a` in JSON without breaking frozen consumers;
5. adjudicate local named-window versus explicit remote parsing;
6. define or intentionally reject remote readonly; and
7. complete help so it reflects the accepted parser.

Extraction work MUST NOT bundle these corrections implicitly.

## Acceptance matrix

- Both spellings with plugin directory absent, corrupt, refused, and containing a shadow attempt.
- Help/missing/extra/unknown/control-bearing inputs and every flag/equals spelling.
- Fast path: wrong verb, non-TTY, bypass flags, exact/ambiguous/missing, inside/outside tmux, readonly,
  list failure, attach spawn failure, and exact child status.
- Normal path: live, sleeping registry, Oracle/repo alias, raw-exact tie break, ambiguous picker,
  missing/recover, list failure, wake bridge, and executor failure.
- Explicit remote: numeric local target, nonnumeric remote target, alias/default, yes, readonly quirk,
  text and exact JSON bytes.
- Golden stdout/stderr/exit fixtures plus a no-unrelated-tmux-I/O RED for the proposed ordering fix.
