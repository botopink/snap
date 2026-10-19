# snap — conformance

> Path: `repository/snap/conformance/`
> Sibling (AGENTS): [`./AGENTS.md`](AGENTS.md)

`snap` is the snapshot engine every `<lib>-test` writes through: the path rule, the `botopink-snap 1`
format and the `<path>.new` flow (decisions 72, 390, 391). It left std's `testing/snapshots.bp` for a
repository of its own, and `import {testing.snapshots} from "std"` answers the plain unknown-module error
(decisions 391, 407). This reference describes the library as its `src/` declares it. Every example is a
conformance cell (decision 461): a decision's example is its directory under `modules/decisions-taken/`,
linked from the section it shows, and the cells that prove `snap` itself come with front 160's Conformance
step (decision 465). A table, a grammar or a file layout is not a program and is fenced as `text`.

| Section | What |
|---|---|
| *The library* | what `snap` holds, the one module `snap`, its dependencies |
| *The path rule* | `suiteOf`, `slugOf`, `path`, `pathNamed` |
| *The snapshot file* | the `botopink-snap 1` format and how a body is compared |
| *The entry points* | `matches`, `matchesAs`, `matchesNamed`, `matchesNamedAs`, `matchesRefused`, the location-taking `assert*`, and how they fail |
| *Where the test's location and name come from* | `snap.matches` expanded at its call, the running test's name |
| *Decisions* | `modules/decisions-taken/`: the ledger's index |

## The library

`snap` holds the path rule, the `.snap` format, the check and the `<path>.new` flow. std provides only the
general primitives it is written over: `compiler.src()` and `SourceLocation`, `io/fs`, `path`, `io/env`
and `io/os`, `mkdirRecursive`, and the text methods `normalizeNewlines`, `trimTrailingNewlines` and
`firstDifferingLine`. A primitive the engine lacks is std surface first, never a host cell in `snap`
(decision 391 (2)). The library has one module, `src/snap.bp` (module `snap`, decision 592), and declares no host cell, so it runs where
`io/fs` runs: commonJS and erlang, its manifest's two targets. It is no loss on wasm, where `io/fs` has no
binding. A consumer declares `"snap"` in its `"dependencies"` and writes `import {snap} from "snap";`
(decision 407). The module's `import` of std binds `io.process` as `host`, because a module-level
`process` would shadow Node's global in the emitted file; `io/random` is not imported at all.

Nothing accepts a snapshot but a person renaming a `.new` file: there is no update flag, no environment
variable and no manifest key (decision 67), and no code path in `snap` ever writes the `.snap` itself.

The `.snap` files that exist stay and change only with their test; a new snapshot exists only where its exact
bytes are read by another package (decision 390).

## The path rule

Four pure functions, the same on every target, read nothing but their arguments:

- `suiteOf(testName: string) -> string` is the text before the first `": "` of a test name, and `""` when
  the name has none.
- `slugOf(text: string) -> string` is the compiler's own slugifier, byte for byte
  (`codegen/tests/helpers.zig`, decision 72): lowercase, every maximal run of `[A-Za-z0-9]` kept, every
  other run one `_`, no leading and no trailing `_`. A text with no such character slugs to `""`.
- `path(loc: SourceLocation) -> string` is `<dir of loc.file>/__snapshots__/<suite>/<slug>.snap`, where the
  suite is `suiteOf(loc.fnName)` slugged and the slug is that of the rest of the name after `": "`. The
  path is package-relative because `loc.file` is: the engine opens it relative to the process's working
  directory, and `botopink test` runs from the package root. A file with no directory gives
  `__snapshots__/<suite>/<slug>.snap`.
- `pathNamed(loc: SourceLocation, name: string) -> string` is a second snapshot of the same test,
  `…/<slug>.<slugOf(name)>.snap`.

```text
test name   css: modifiers ---- hover on md breakpoint
suite       css
slug        modifiers_hover_on_md_breakpoint
path        <dir>/__snapshots__/css/modifiers_hover_on_md_breakpoint.snap
named       <dir>/__snapshots__/css/modifiers_hover_on_md_breakpoint.after_merge.snap
```

The path rule is pure botopink, so under `comptime` each call runs at build on the wasm comptime runtime
whatever `--target` the build has (decision 594); the example is
[libraries_run_at_build_on_wasm-594](modules/decisions-taken/libraries_run_at_build_on_wasm-594/draft/main.bp).

## The snapshot file

A `.snap` is a header, a blank line and the body (decision 72):

```text
botopink-snap 1
test: <loc.fnName>
subject: <subject>

<body, verbatim, one trailing newline appended>
```

`test:` is the full test name, suite included, and `subject:` the name the helper gives (`css`, `html`,
`route`, `text`). The check is exact on the header lines. Bodies compare after `\r\n` becomes `\n` and
with trailing newlines trimmed on both sides, the two normalisations the compiler's `utils/snap.zig`
applies, and nothing else. A recorded file whose first line is not `botopink-snap 1`, or that is too
short to hold a header, is not a snapshot file.

## The entry points

Five functions are expanded at their call (decision 719) and answer the run-time call that checks the
snapshot; a failure panics with the message below, so it is the test's failure:

- `snap.matches(value)` records or compares `value`, a `string`, as the test's one snapshot with subject
  `text`.
- `snap.matchesAs(subject, value)` is what a `<lib>-test` helper `assert<Subject>` answers: the same, with
  the helper's subject in the header.
- `snap.matchesNamed(name, value)` is a named second snapshot of the same test, subject `text`.
- `snap.matchesNamedAs(name, subject, value)` is a named snapshot with a subject. A name that slugs to
  `""` is refused.
- `snap.matchesRefused(comptime compiler.load(path))` records the refusals `compiler.load` answers for a file
  (decision 784), the call a refused case's test writes: `test "<case>" { snap.matchesRefused(comptime
  compiler.load("cases/<case>/main.bp")); }`. On `Error(ds)` the body is one block per `Diagnostic`, in
  order, separated by one blank line — `refused <message>`, `at <file>:<line>:<column>` and `label <text>`
  only when the diagnostic has a label — under subject `refused`; the snapshot is placed, named and
  compared as any other. On `Ok` the test fails, naming the file that compiled:
  `snapshots: <path> compiled - matchesRefused expects a refused file`. It is the run-time
  `matchesRefusedAt(loaded, written, at)` that does it, `written` the load call as the test wrote it.
  The cell is [snap_matches_refused](test/snap_matches_refused/test/snap_matches_refused.bp), shown by
  [snap_matches_refused-784](modules/decisions-taken/snap_matches_refused-784/draft/main.bp).

Each of the first four is `pub fn …(…, syntax value: unknown) -> anyvalue` over a run-time `matches…At(…, value: unknown,
at: Place) -> void`; `Place(file, line, col)` is snap's own record for the call's `compiler.Source`, which
no target builds at run time yet.

The four location-taking functions stay until the `<lib>-test` helpers move to `matchesAs`, each answering
`Result<void, string>` over a `SourceLocation` the caller writes (`comptime compiler.src()`, decision 718):
`assertText(loc, actual)`, `assertAs(loc, subject, actual)`, `assertNamed(loc, name, actual)` and
`assertNamedAs(loc, name, subject, actual)`. A helper takes the **caller's** location; a location taken
inside the helper would name the helper.

A test name without a suite is refused before any path is computed (decision 67: no `misc/` default):
a call made outside a running test has no name, and one whose name has no `": "` has no suite.

An entry point fails in these ways, each the text beginning with `snapshots:`:

```text
snapshots: missing <path> - candidate written to <path>.new; review it and rename it to record
snapshots: mismatch <path> - candidate written to <path>.new (header differs)
snapshots: mismatch <path> - candidate written to <path>.new (first difference at body line <n>)
snapshots: not a snapshot file: <path>
snapshots: test name needs a suite - write "<suite>: <description>"
snapshots: a named snapshot needs a name - pass a non-empty name to matchesNamed/matchesNamedAs
snapshots: cannot write <path>.new: <fs error>
```

A missing file and a mismatch each write `<path>.new`, creating the directories above it, and answer
`Error`; `<n>` counts lines of the body, from 1. A match answers `Ok` and removes a stale `<path>.new`
an earlier red run left. A host failure while writing is the test's failure, never a silent pass. When
`io/fs` fails, the engine carries the failure into its `string` error with the error's own text,
`e.message()`, one text per variant and the same on every target (decision 563); snap has no `case` of
its own over the variants. The example is
[fs_error_message_text-563](modules/decisions-taken/fs_error_message_text-563/draft/main.bp).

## Where the test's location and name come from

The form consumers write is `snap.matches(render(page))`, with no location argument: `matches` takes its
argument as a `syntax` parameter, so the call is expanded where it is written and its build body reads the
test's file and line with `compiler.source(value)` (decision 719). `compiler.source(value).file` is the
file relative to the package root as diagnostics print it, `src/main.bp`, never the module path
(decision 756). The argument is not executed at the call but spliced into the code the call answers
(decision 720): the body writes that code with std's `code` template, `${value}` a `syntax` hole spliced as the
expression was written and the place's file, line and column spliced as literals (decisions 625 (4), 637, 638).

The location only places the snapshot, beside its file. The snapshot's name is the running test's: the
test runner sets it where the run-time `matchesAt` reads it, `BOTOPINK_TEST_NAME` while a test's body runs
(decision 772), so a snapshot keeps its name when a line above it moves. The engine's own tests
`snapshots: name ---- the running test names the first snapshot` and `… the second snapshot` record one
snapshot each, their names in the headers. The cell [snapshot_named_by_running_test](test/snapshot_named_by_running_test/test/snapshot_named_by_running_test.bp)
proves it from a consumer's side: two tests, one `snap.matches` each, each `.snap` named by its own test.

A location can still be written as an explicit argument, `comptime compiler.src()`: `src` is a function
of `Compiler`, `fn src(self: Self) -> SourceLocation`, always written with its parentheses, and the
explicit argument is an ordinary parameter a call without it fails the arity check on (decision 718); the
`assert*` functions take it so. A default `at: SourceLocation = comptime compiler.src()` on a parameter
would name snap's own module and gets no exception (decisions 718, 719).

## Decisions

`modules/decisions-taken/` is the ledger's index: one `<slug>-<N>/` per code-bearing row of this
repository's generated `decisions-taken.md` (front 166, decisions 646-651), its `docs.md` walking through
one example program. The cells that show a point of a section above are linked from it:
[libraries_run_at_build_on_wasm-594](modules/decisions-taken/libraries_run_at_build_on_wasm-594/draft/main.bp) (the path rule at build),
[fs_error_message_text-563](modules/decisions-taken/fs_error_message_text-563/draft/main.bp) (an `io/fs` failure as the engine's error text),
[own_module_import_without_from-206](modules/decisions-taken/own_module_import_without_from-206/draft/main.bp) (a module of the package imported by its path, with no `from`) and
[shorthand_import_refused-337](modules/decisions-taken/shorthand_import_refused-337/draft/main.bp) (a bare name never reaches a dependency's modules).
The remaining row, [optional_typescript_operators-330](modules/decisions-taken/optional_typescript_operators-330/draft/main.bp), shows `?T` read with TypeScript's operators.
