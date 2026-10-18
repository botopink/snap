# Decision 563 — An `FsError` carries its own text: std's `io/fs` gives it `message()`

The example is the engine's read of a recorded snapshot, written as a consumer of `io/fs` would
write it. `readRecorded` answers `Result<string, string>` like every `assert*` entry point;
when `fs.readText` fails, its handler turns the `FsError` into the engine's `string` error with
`e.message()` and a prefix of its own — there is no `case` over the error's variants and no local
helper that spells them.

`main` reads two files. The first is the `.snap` path `path(loc)` computes for a test in
`src/missing/t.bp`, which does not exist, so the text printed is the not-found variant's,
`snapshots: cannot read not found: src/missing/__snapshots__/css/hover.snap`. The second is `.`,
a directory, so the text is the is-a-directory variant's. Each line is the variant's own text,
the path after it, and it is the same on every target.

The `SourceLocation` here is built by hand so that the paths are fixed. In a real test the caller's location reaches the engine through `comptime compiler.src` — a reading of decision 606 (3), which no row states for snap's consumers yet.

Proving cells: none in this repository yet; snap's own cells come with front 160's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)
