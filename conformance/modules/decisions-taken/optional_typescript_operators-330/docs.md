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

Proving cells: none in this repository yet; the language's are botopink-lang's `conformance/modules/optional_operators`, `optional_call_operator`, `nullish_optional_call_default`, `optional_bang_aborts`, `optional_has_no_methods`, `nullish_never_null`, `nullish_beside_logical` and `result_namespace_removed`.

[draft/main.bp](draft/main.bp)
