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

Proving cells: none in this repository yet; the language's is botopink-lang's `conformance/modules/fs_error_message`.

[draft/main.bp](draft/main.bp)
