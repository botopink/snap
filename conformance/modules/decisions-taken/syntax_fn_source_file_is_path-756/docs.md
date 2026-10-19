# Decision 756 — (a): `compiler.source(value).file` is the file, relative to the package root, as diagnostics print it — `src/main.bp` — never the module path.

The example is a `<lib>-test`-style helper over snap. `placeOf` takes its argument as `syntax`, so it is expanded where it is called, and reads `compiler.source(value)`: the call's file and line. The file is the one diagnostics print, relative to the package root — `src/main.bp` —, never the module path `main`; the expansion writes it, with the line, into the run-time call `placeAt` it answers. `placeAt` hands that file to snap's `path`, which puts a snapshot in `<dir of the file>/__snapshots__/<suite>/<slug>.snap`; so the program prints `src/main.bp:23` and `src/__snapshots__/css/modifiers_hover.snap`, the snapshot next to the source that took it. With the module path the directory would be lost and the snapshot would land at the package root.

Cells that prove it: botopink-lang's `syntax_fn_source_file_is_path` under its decision 756 (`src/main.bp:32` on four targets). In this repository snap's own tests in `src/snap.bp` write their snapshots through the same expansion (`src/__snapshots__/snapshots/`); no conformance cell yet — they come with front 160's Conformance step.

[draft/main.bp](draft/main.bp)
