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

Proving cells: none in this repository or in botopink-lang's `conformance/modules/` names this decision yet.

[draft/main.bp](draft/main.bp)
