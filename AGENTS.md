# snap

`decisions-taken.md` is generated — from the meta repository's `specs/<milestone>/decisions-taken.md` by `scripts/decisions-split.py`; edit it there, never here.

> Repository: `botopink/snap` (`git@github.com:botopink/snap.git`) · in the meta checkout: `repository/snap/`

The snapshot engine every `<lib>-test` writes through (decisions 72, 390, 391): the path rule
(`<root>/<dir of loc.file>/__snapshots__/<suite>/<slug>.snap`, `<root>/` only for a loaded package, decision 1003), the `botopink-snap 1` format and the
`<path>.new` flow — a person renaming `.new` accepts a snapshot; there is no update flag (decision
67). Moved here from std's `testing/snapshots.bp` by `specs/1.0.12-beta/20-snap` step 0, with std's
four `.snap` files.

A library of its own (decision 326's rule; 115: five consumers). It imports std only and declares
no host cell: std provides the general primitives it is written over (`compiler.source`, the `code` template, `SourceLocation`,
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
├── botopink.json     "name": "snap", "targets": ["erlang", "beam", "commonJS"] (no wasm: std `io.fs.mkdirRecursive` has no `.Wasm(…)` binding, decision 993 (4)), "files": ["snap.bp"] (what a consumer loads), std only
├── src/
│   ├── snap.bp       the whole engine, module `snap` — `path`, `pathNamed`, `suiteOf`, `slugOf` (pure), the four
│   │                 expanded functions `matches`, `matchesAs`, `matchesNamed`, `matchesNamedAs` over their run-time
│   │                 `matches*At(…, at: Place)`, the location-taking `assert*` kept until their consumers move, and its eighteen inline tests
│   └── __snapshots__/snapshots/   the `.snap` files the path-rule and naming tests record
├── conformance/      `docs.md` (the reference), `modules/decisions-taken/` (front 166's), `test/snapshot_named_by_running_test/` (772), `test/snap_matches_refused/` (784)
└── scripts/git-hooks/   the shared pre-commit gate
```

## The engine

Consumers write `import {snap} from "snap";` and call a function that is **expanded at its call**
(decision 719): the test writes nothing but the expression, and the expansion writes the call's
`Source` (file, line, column) into the run-time call it answers:

```bp
import {snap} from "snap";

test "release: text ---- the vm.args file" {
    snap.matchesAs("vm.args", releaseText(app));   // __snapshots__/release/text_the_vm_args_file.snap
}
```

Each is `pub fn matches…(…, syntax value: unknown) -> anyvalue { val at = compiler.source(value); return code @`matches…At(…, ${value}, placeAt(${at.file}, ${at.line}, ${at.col}))`; }`
over a run-time `matches…At(…, value: unknown, at: Place) -> void` — code written with holes, never `compiler.build` over text
(decision 625); `Place(file, line, col)` is snap's own record, since `compiler.Source` cannot be built at run
time on erlang or commonJS yet (B-147). The snapshot's **name** is the running test's, read from
`BOTOPINK_TEST_NAME`, which `botopink test`'s runner sets while each test body runs (decision 772); the
location only places the file.

- `path(loc)` = `<root>/<dir of loc.file>/__snapshots__/<suite>/<slug>.snap` — `loc.root` (decision 1003) joined first when it is not `""`, the directory `compiler.load` loaded the package from; `suite` is the text before the
  first `": "` of `loc.fnName`, `slug` the slugified rest (the compiler's own rule, byte for byte:
  lowercase, a maximal run of `[A-Za-z0-9]` kept, anything else one `_`, none leading or trailing).
  `pathNamed(loc, name)` is `…/<slug>.<slugOf(name)>.snap`.
- The `.snap` is `botopink-snap 1` / `test: <name>` / `subject: <subject>` / blank / body. Header
  lines compare exactly; bodies compare after `\r\n` -> `\n` and with trailing newlines trimmed.
- A missing file or a mismatch writes `<path>.new` (its parents made with `fs.mkdirRecursive`) and
  answers `Error`; a match removes a stale `.new`. Nothing here writes `<path>` itself.
- A name with no suite (`""`, or no `": "`) and an empty `name` of `matchesNamed*` are refused.
- Entry points: `matches(value)` (subject `text`), `matchesAs(subject, value)`,
  `matchesNamed(name, value)`, `matchesNamedAs(name, subject, value)`, each over its `…At` run-time
  function; `assertText(loc, actual)`, `assertAs`, `assertNamed`, `assertNamedAs` over a caller-written
  `SourceLocation` stay until the `<lib>-test` helpers (rakun-bpp's and the others) call `matchesAs`. The engine proper is private: `placeAs(loc, subject, actual)` and
  `placeNamedAs(loc, name, subject, actual)`, `-> Result<void, string>`, over a `SourceLocation`.

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
