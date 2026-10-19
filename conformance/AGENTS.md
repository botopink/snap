# snap — conformance/

- `modules/decisions-taken/<slug>-<N>/` — front 166's decision cells (decision 651: the subtree is that front's alone): one
  per code-bearing row of `../decisions-taken.md`, a `docs.md` (first line generated) and, in phase 1, `draft/main.bp` (decision
  649; no `src/`, no manifest, so `botopink cell check` finds no cell there); kept by `scripts/decision-cells.py` of the meta
  repository and its CI check 7.
- `modules/botopink.json` — the workspace manifest (`"workspaces": ["*"]`, snap's targets, `snap` by `"path": "../.."`) that
  `decision-cells.py compile` reads for the drafts' dependencies.
- `docs.md` — the library's reference, in botopink-lang's `conformance/docs.md` pattern (no fenced `botopink`; a table, a
  grammar or a file layout is fenced as `text`; every example is a cell or a decision cell's draft, linked). One section each:
  *The library* (what `snap` holds, `root.bp`, its dependencies), *The path rule* (`suiteOf`, `slugOf`, `path`, `pathNamed`),
  *The snapshot file* (the `botopink-snap 1` format and the body comparison), *The entry points* (`assertText`, `assertAs`,
  `assertNamed`, `assertNamedAs` and their `snapshots:` errors), *Where the test's location comes from*
  (`comptime compiler.src()`, `snap.matches`; the departure of `src/root.bp`), *Decisions* (the ledger's index).
