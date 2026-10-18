# Decision 206 — `import {…} from "<name>"` resolves only to a package — std, a bundled package or a declared dependency.

The example is a package with two modules, and one of them is named `snap` — the name of the
dependency every `<lib>-test` writes through. `main.bp` imports `suiteOf` and `slugOf` with
`from "snap"`: that source string names the package `snap` and nothing else, so both names come
from the engine's `src/root.bp`, not from the local module that shares the name. The local
module is imported by its path in braces with no `from` — `import {snap.describe};` — and that
is the only way to reach it. `snap.bp` itself does the same in the other direction: it takes
`pathNamed` from the dependency with `from "snap"` and `SourceLocation` from std with
`from "std"`.

Run, the program prints the suite `css`, the slug of the whole test name, and the path of a
named second snapshot as the dependency computes it — three answers that show the two `snap`s
never mix. Writing `from "<this package>"` for the local module is refused
(`module-import-with-from`, decision 309), its fix the brace form used here.

The `SourceLocation` here is built by hand so that the paths are fixed. In a real test the caller's location reaches the engine through `comptime compiler.src` — a reading of decision 606 (3), which no row states for snap's consumers yet.

Proving cells: none in this repository yet; snap's own cells come with front 160's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)

[draft/snap.bp](draft/snap.bp)
