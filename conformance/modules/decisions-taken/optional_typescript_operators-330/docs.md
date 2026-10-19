# Decision 330 — `?T` is read with TypeScript's operators and has no methods; `result` goes; a type declared in a type is its associated type.

The example reads optional values the way the engine reads its scratch directory:
`env.read("BOTOPINK_TEST_TMPDIR") ?? os.tmpdir()` takes the host's tmpdir when the variable is
absent, `??` being the whole fallback. `suiteOrNull` answers `?string` — `null` for a test name
with no `": "` — and `main` reads it with the operators alone: `suite?.length() ?? 0` calls the
method only when there is a suite (the answer `?i32`, flattened, never `??i32`), and `suite!`
unwraps a value the program knows is there; on a `null` it would abort with the location and the
expression, the same on every target. The second name has no suite, so `none ?? "(no suite)"`
and `none?.length() ?? 0` take their fallbacks.

The last lines read a `Result`, not an optional: `fs.readText` of a snapshot file that does not exist
answers `Error`, and the program asks it with its own methods, `recorded.isError()` and
`recorded.unwrapOr("(no snapshot)")` — the methods stay on `Result`, the operators on `?T`, and no
`result.*` namespace is involved. The test records the fallback `none ?? "(no suite)"` gives as its
snapshot.

The test writes the assertion as every consumer will, `snap.matches(<value>)`, with no location argument: `matches` takes its argument as `syntax` (decision 719), so the call is expanded where it is written and the snapshot's file and line are read from that argument, `compiler.source`, at the call; the argument is not executed there (decision 720) but spliced into the answer, which calls the engine at run time. snap's own entry points are still `assertText` / `assertAs` taking a hand-written `SourceLocation`, with no `snap` namespace for `import {snap} from "snap";` to bind, and a `syntax` parameter of a type other than `string` is not yet accepted, so the draft does not build: it compiles once 144 B-156 (machine 1) and front 160's step that writes `matches` land — that is its departure.

Proving cells: none in this repository yet; snap's own cells come with front 160's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)
