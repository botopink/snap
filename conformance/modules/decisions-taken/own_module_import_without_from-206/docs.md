# Decision 206 — `import {…} from "<name>"` resolves only to a package — std, a bundled package or a declared dependency.

The example is a package with two modules, and one of them is named `snap` — the name of the
dependency every `<lib>-test` writes through. `main.bp` writes `import {snap} from "snap";`: that
source string names the package `snap` and nothing else, so `snap.matches` in the test is the
engine's, not the local module's that shares the name. The local module is imported by its path
in braces with no `from` — `import {snap.describe};` — and that is the only way to reach it.
`snap.bp` itself does the same in the other direction: it takes `suiteOf` and `slugOf` from the
dependency with `from "snap"`.

Run, the program prints `describe`'s line, the suite `css` and the slug of the whole test name as
the dependency computes them, and the test records that line as its snapshot through the
dependency — answers that show the two `snap`s never mix. Writing `from "<this package>"` for the local module is refused
(`module-import-with-from`, decision 309), its fix the brace form used here.

The test writes the assertion as every consumer will, `snap.matches(<value>)`, with no location argument: `matches` takes its argument as `syntax` (decision 719), so the call is expanded where it is written and the snapshot's file and line are read from that argument, `compiler.source`, at the call; the argument is not executed there (decision 720) but spliced into the answer, which calls the engine at run time. snap's own entry points are still `assertText` / `assertAs` taking a hand-written `SourceLocation`, with no `snap` namespace for `import {snap} from "snap";` to bind, and a `syntax` parameter of a type other than `string` is not yet accepted, so the draft does not build: it compiles once 144 B-156 (machine 1) and front 160's step that writes `matches` land — that is its departure.

Proving cells: none in this repository yet; snap's own cells come with front 160's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)

[draft/snap.bp](draft/snap.bp)
