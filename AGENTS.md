# snap

> Repository: `botopink/snap` (`git@github.com:botopink/snap.git`) · in the meta checkout: `repository/snap/`

The snapshot engine every `<lib>-test` writes through (decisions 72, 390, 391): the path rule
(`<dir of loc.file>/__snapshots__/<suite>/<slug>.snap`), the `botopink-snap 1` format and the
`<path>.new` flow — a person renaming `.new` accepts a snapshot; there is no update flag (decision
67). It is moved here from std's `testing/snapshots.bp` by `specs/1.0.12-beta/20-snap` step 0, and
until that step lands this repository holds only its scaffold.

A library of its own (decision 326's rule; 115: five consumers). It imports std only and declares
no host cell: std provides the general primitives it is written over (`@src()` / `SourceLocation`,
`io/fs`, `path`, `io/env`, the generic text methods of 391 (2)).

## Hooks

Every clone runs the shared pre-commit gate:

```sh
git config core.hooksPath scripts/git-hooks
```

`scripts/git-hooks/pre-commit` and `scripts/git-hooks/lib/runner-standalone.sh` are byte-identical
across the library repositories (the meta repository's CI check 4); edit them in one place and copy.
