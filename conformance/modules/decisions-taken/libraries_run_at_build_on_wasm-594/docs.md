# Decision 594 — Every library runs at build on the wasm comptime runtime (434), jhonstart first; a function only Erlang or Node can run is a located error there.

snap's path rule is pure botopink — `slugOf`, `suiteOf`, `path` and `pathNamed` read nothing but
their arguments — so it needs no wasm form of its own: written under `comptime`, each call in the
example runs at build on the wasm comptime runtime, whatever `--target` the build has, and the
program carries four string literals. It prints the slug
`modifiers_hover_on_md_breakpoint`, the suite `css`, the path
`src/__snapshots__/css/modifiers_hover_on_md_breakpoint.snap` and the named path
`test/__snapshots__/route/two_tables.after_merge.snap`, the same as the run-time calls would.

The engine's four `assert*` entry points are the other half of the rule: they reach `io/fs`,
which has no wasm binding, so one of them written under `comptime` would be refused at build,
located, naming the host function it reaches — never run on the target's VM instead.

The `SourceLocation` here is built by hand so that the paths are fixed. In a real test the caller's location reaches the engine through `comptime compiler.src` — a reading of decision 606 (3), which no row states for snap's consumers yet.

Proving cells: none in this repository yet; snap's own cells come with front 160's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)
