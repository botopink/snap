# Decision 337 — The shorthand import goes; `mod m;` binds the namespace `m` in the module that declares it.

The example is refused. `main.bp` writes two shorthand imports, items with no module path and no
`from`. `import {slugOf};` names a function the dependency `snap` exports, but a bare name never
reaches a dependency's modules (nor std's): no module of this package declares `slugOf`, so the
item is refused as an unresolved import, located at it — the program has to write
`import {slugOf} from "snap";`, as `naming.bp` does. `import {snapshotFile};` names a function
one module of this package declares `pub`, so the refusal at that item writes its fix,
`import {naming.snapshotFile};`.

Both refusals are `shorthand-import`, one per item, and the program builds once each item names
where its name comes from.

Proving cells: none in this repository yet; snap's own cells come with front 160's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)

[draft/naming.bp](draft/naming.bp)
