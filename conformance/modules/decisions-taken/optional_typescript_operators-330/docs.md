# Decision 330 — `?T` is read with TypeScript's operators and has no methods; `result` goes; a type declared in a type is its associated type.

The example reads optional values the way the engine reads its scratch directory:
`env.read("BOTOPINK_TEST_TMPDIR") ?? os.tmpdir()` takes the host's tmpdir when the variable is
absent, `??` being the whole fallback. `suiteOrNull` answers `?string` — `null` for a test name
with no `": "` — and `main` reads it with the operators alone: `suite?.length() ?? 0` calls the
method only when there is a suite (the answer `?i32`, flattened, never `??i32`), and `suite!`
unwraps a value the program knows is there; on a `null` it would abort with the location and the
expression, the same on every target. The second name has no suite, so `none ?? "(no suite)"`
and `none?.length() ?? 0` take their fallbacks.

The last lines read a `Result`, not an optional: `fs.readText` of a snapshot that does not exist
answers `Error`, and the program asks it with its own methods, `recorded.isError()` and
`recorded.unwrapOr("(no snapshot)")` — the methods stay on `Result`, the operators on `?T`, and no
`result.*` namespace is involved.

The `SourceLocation` here is built by hand so that the paths are fixed. In a real test the caller's location reaches the engine through `comptime compiler.src` — a reading of decision 606 (3), which no row states for snap's consumers yet.

Proving cells: none in this repository yet; snap's own cells come with front 160's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)
