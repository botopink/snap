# snap

`decisions-taken.md` is generated — from the meta repository's `specs/<milestone>/decisions-taken.md` by `scripts/decisions-split.py`; edit it there, never here.

> Repository: `botopink/snap` (`git@github.com:botopink/snap.git`) · in the meta checkout: `repository/snap/`

The snapshot engine every `<lib>-test` writes through (decisions 72, 390, 391): the path rule
(`<dir of loc.file>/__snapshots__/<suite>/<slug>.snap`), the `botopink-snap 1` format and the
`<path>.new` flow — a person renaming `.new` accepts a snapshot; there is no update flag (decision
67). Moved here from std's `testing/snapshots.bp` by `specs/1.0.12-beta/20-snap` step 0, with std's
four `.snap` files.

A library of its own (decision 326's rule; 115: five consumers). It imports std only and declares
no host cell: std provides the general primitives it is written over (`@src()` / `SourceLocation`,
`io/fs` with `mkdirRecursive`, `path`, `io/env`, `io/os`, and the `String` methods
`normalizeNewlines`, `trimTrailingNewlines` and `firstDifferingLine` of decision 391 (2)). Pure
`.bp`: no `#[@External]` cell, no `declare fn`, no sidecar. It runs where `io/fs` does — commonJS,
erlang and beam (`botopink test` runs on commonJS and erlang); `io/fs` has no wasm binding.

A program that imports `from "snap"` declares it in `dependencies` (decision 242) —
`{ "snap": { "git": "https://github.com/botopink/snap.git", "branch": "feat" } }`; inside the meta
checkout that entry resolves by name through the `repository/` root (`repository/snap`), elsewhere
through the install store. `import {testing.snapshots} from "std"` is gone: std keeps no snapshot
engine and no re-export.

## Tree

```text
snap/
├── AGENTS.md
├── botopink.json     "name": "snap", "target": "erlang", "targets": ["erlang", "commonJS"], std only
├── src/
│   ├── root.bp       the whole engine — `path`, `pathNamed`, `suiteOf`, `slugOf` (pure), the four entry
│   │                 points `assertText`, `assertAs`, `assertNamed`, `assertNamedAs`, and its sixteen inline tests
│   └── __snapshots__/snapshots/   the four `.snap` the path-rule tests record (moved from std, byte for byte)
└── scripts/git-hooks/   the shared pre-commit gate
```

## The engine

Consumers write `import {assertAs} from "snap";` and call it with the CALLER's `@src()` — a
`@src()` taken inside a helper would name the helper:

```bp
import {assertAs} from "snap";

test "release: text ---- the vm.args file" {
    try assertAs(@src(), "vm.args", releaseText(app));   // __snapshots__/release/text_the_vm_args_file.snap
}
```

- `path(loc)` = `<dir of loc.file>/__snapshots__/<suite>/<slug>.snap`; `suite` is the text before the
  first `": "` of `loc.fnName`, `slug` the slugified rest (the compiler's own rule, byte for byte:
  lowercase, a maximal run of `[A-Za-z0-9]` kept, anything else one `_`, none leading or trailing).
  `pathNamed(loc, name)` is `…/<slug>.<slugOf(name)>.snap`.
- The `.snap` is `botopink-snap 1` / `test: <name>` / `subject: <subject>` / blank / body. Header
  lines compare exactly; bodies compare after `\r\n` -> `\n` and with trailing newlines trimmed.
- A missing file or a mismatch writes `<path>.new` (its parents made with `fs.mkdirRecursive`) and
  answers `Error`; a match removes a stale `.new`. Nothing here writes `<path>` itself.
- A name with no suite (`""`, or no `": "`) and an empty `name` of `assertNamed*` are refused.
- Entry points: `assertText(loc, actual)` (subject `text`), `assertAs(loc, subject, actual)`,
  `assertNamed(loc, name, actual)`, `assertNamedAs(loc, name, subject, actual)` — every one
  `-> @Result<void, string>`.

The inline tests make a scratch directory `bpsnap-<pid>-<n>` under `BOTOPINK_TEST_TMPDIR` (the host
tmpdir outside a run) and remove it again with `fs.removeTree`.

## Hooks

Every clone runs the shared pre-commit gate:

```sh
git config core.hooksPath scripts/git-hooks
```

The hook applies `botopink format` to the commit's staged source files before its gate (decision 524 — the
staged version, into the index; formatting never fails it) and runs the gate with `--jobs 2`.

`scripts/gate.sh [--staged] [--no-conformance] [--jobs <N>]` is the gate's one entry (one text in every
library repository, CI check 4; it runs `scripts/git-hooks/lib/runner-standalone.sh`): the pre-commit hook
runs `scripts/gate.sh --staged --no-conformance` (decision 470), and CI on a push to `feat` and a front's
Conformance step run it whole, which adds the last stage, `botopink cell check conformance/` over this
repository's `conformance/` cells (decisions 463, 464; the stage exists when the directory does). It knows no
job pool: `--jobs` is the total it may use, and the meta repository's `scripts/gate.sh run <repository>`
hands it its share.

`scripts/git-hooks/pre-commit` and `scripts/git-hooks/lib/runner-standalone.sh` are byte-identical
across the library repositories (the meta repository's CI check 4); edit them in one place and copy.
