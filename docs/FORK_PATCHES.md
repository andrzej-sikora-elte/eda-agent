# Fork patches

This fork tracks upstream (`upstream/main`) and keeps its own additions on a
separate branch (`elte`). `main` is always a plain mirror of upstream. This file
lists what the branch adds, so that a merge of upstream can be checked quickly.

## Schematic component: database-library fields (`obj_query` / `obj_modify` / `obj_batch_modify`)

Upstream already reads `DatabaseTableName`, `DatabaseLibraryName` and the managed
part identity (`LibraryIdentifier`, GUIDs). The branch adds the rest.

| Property | Read | Write | Notes |
|---|---|---|---|
| `UseDBTableName` | yes (added) | yes (added) | `true` / `false` |
| `UseLibraryName` | yes (added) | yes (added) | `true` / `false` |
| `FootprintModel` | yes (added) | yes (added) | the current PCB footprint of the placed part; see below |
| `LibraryIdentifier` | upstream | yes (added) | derived value, see behaviour notes |
| `DatabaseTableName` | upstream | yes (added) | changing it changes the suffix of `LibraryIdentifier` |

`FootprintModel` is applied through the PCBLIB implementation of the component
(`CurrentFootprintModelName` itself is read-only). Value format: `Name` or
`Name@Library.PcbLib`. When a library is given it is written into every datafile
link of the current PCBLIB implementation.

Applies to schematic components only; on other objects the read is reported as
unreadable.

## Where the code is

All changes are in `scripts/altium/Generic.pas`:

- `GetCompFootprint` and `SetCompFootprint`: new helper functions placed just
  before `GetSchProperty`.
- `GetSchProperty`: one extra branch right after the upstream
  `DatabaseTableName` / `DatabaseLibraryName` branch (reads the three fields above).
- The property setter: five extra branches right after the `DesignItemId` case.

Nothing is removed from upstream code.

## Behaviour observed on a live check

- With `UseLibraryName=false`, `LibraryIdentifier` reads as an empty string;
  setting it back to `true` restores the original value.
- Setting `DatabaseTableName` also changes the table part of `LibraryIdentifier`;
  setting the original table name restores it.
- `FootprintModel` reads back exactly what was written.

## Updating from upstream

```
git fetch upstream
git checkout elte
git merge upstream/main
# resolve conflicts, keeping the additions listed above
python scripts/altium/lint.py
git push origin elte
```

If upstream starts to provide any of these properties itself, drop the matching
branch from this branch rather than keeping two copies.

## Checking after a merge

On a scratch copy of a schematic sheet that contains a component placed from a
database library (never on a working project):

1. Read the five fields with `obj_query` on one component and note them.
2. Change `UseDBTableName`, `UseLibraryName` and `FootprintModel` with
   `obj_batch_modify`, then read them back.
3. Restore the original values and confirm the read matches step 1.
4. Close the copy with the changes discarded.

## Contributing upstream

Upstream asks for an issue before a non-trivial change (see `CONTRIBUTING.md`).
The additions above are small and self-contained and could be proposed that way.