# Decision 563 — An `FsError` carries its own text: std's `io/fs` gives it `message()`

The example is the engine's read of a recorded snapshot, written as a consumer of `io/fs` would
write it. `readRecorded` answers `Result<string, string>` like every `assert*` entry point;
when `fs.readText` fails, its handler turns the `FsError` into the engine's `string` error with
`e.message()` and a prefix of its own — there is no `case` over the error's variants and no local
helper that spells them.

`main` reads two files. The first is `src/missing/__snapshots__/css/hover.snap`, which does not
exist, so the text printed is the not-found variant's,
`snapshots: cannot read not found: src/missing/__snapshots__/css/hover.snap`. The second is `.`,
a directory, so the text is the is-a-directory variant's. Each line is the variant's own text,
the path after it, and it is the same on every target. The test records the not-found text as its
snapshot.

The test writes the assertion as every consumer will, `snap.matches(<value>)`, with no location argument: `matches` takes its argument as `syntax` (decision 719), so the call is expanded where it is written and the snapshot's file and line are read from that argument, `compiler.source`, at the call; the argument is not executed there (decision 720) but spliced into the answer, which calls the engine at run time. snap's own entry points are still `assertText` / `assertAs` taking a hand-written `SourceLocation`, with no `snap` namespace for `import {snap} from "snap";` to bind, and a `syntax` parameter of a type other than `string` is not yet accepted, so the draft does not build: it compiles once 144 B-156 (machine 1) and front 160's step that writes `matches` land — that is its departure.

Proving cells: none in this repository yet; snap's own cells come with front 160's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)
