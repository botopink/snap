# Decision 594 — Every library runs at build on the wasm comptime runtime (434), jhonstart first; a function only Erlang or Node can run is a located error there.

snap's path rule is pure botopink — `slugOf`, `suiteOf`, `path` and `pathNamed` read nothing but
their arguments — so it needs no wasm form of its own: written under `comptime`, each call in the
example runs at build on the wasm comptime runtime, whatever `--target` the build has, and the
program carries the string literals. It prints the slug `modifiers_hover_on_md_breakpoint` and the
suite `css`, the same as the run-time calls would.

The assertion is the other half of the rule. `snap.matches`'s body is a build body (decision 719):
it runs at build, reads the call's location and returns the code that stands at the call. That
code reaches `io/fs`, which has no wasm binding, so it runs on the target's VM when the test runs;
written under `comptime`, the engine's run-time entry point would be refused at build, located,
naming the host function it reaches — never run on the target's VM instead.

The test writes the assertion as every consumer will, `snap.matches(<value>)`, with no location argument: `matches` takes its argument as `syntax` (decision 719), so the call is expanded where it is written and the snapshot's file and line are read from that argument, `compiler.source`, at the call; the argument is not executed there (decision 720) but spliced into the answer, which calls the engine at run time. snap's own entry points are still `assertText` / `assertAs` taking a hand-written `SourceLocation`, with no `snap` namespace for `import {snap} from "snap";` to bind, and a `syntax` parameter of a type other than `string` is not yet accepted, so the draft does not build: it compiles once 144 B-156 (machine 1) and front 160's step that writes `matches` land — that is its departure.

Proving cells: none in this repository yet; snap's own cells come with front 160's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)
