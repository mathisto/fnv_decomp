# Mapping for the Xbox 360 PDB names

This branch holds the machine-readable mapping behind the pull request
[momusdigital/fnv_decomp#42](https://github.com/momusdigital/fnv_decomp/pull/42), "Names from the Xbox 360 development
build PDBs". [PDB_NAMES.md](https://github.com/mathisto/fnv_decomp/blob/967b001e56f6e1ebcb5615db2ac58942983a5ddd/PDB_NAMES.md) describes how the names
were linked and applied.

The files match commit [`967b001e`](https://github.com/mathisto/fnv_decomp/commit/967b001e56f6e1ebcb5615db2ac58942983a5ddd), the head of the pull
request when they were written; file and line numbers refer to that commit. They are JSON Lines, one object per line.
They hold extracted names and metadata (addresses, offsets, link methods); no PDB files or executable bytes. Addresses
are PC addresses in `FalloutNV.exe` unless the key says `x360_`. `old` is the name on `main` before the pull request.

| File | Rows | One row is |
|---|---|---|
| `functions_renamed.jsonl` | 3,141 | A function whose own name changed: `address`; `old_symbols` and `new_symbols`, the decorated names `build/units.json` gives it, C names included, that changed; `unchanged_symbols` (192 rows), other declarations of the same address that keep `main`'s name (constructor and destructor symbols not listed); `pdb_name`; `build` (`retail` or `debug`); link `method`; `tier`; and the `// src:` `marker` in the source |
| `functions_linked.jsonl` | 37,460 | Every PC function linked to a PDB function, renamed or not: `status` (`renamed`, `already named`, `not renamed`) and `reason`. Review conflicts have `tier` `disputed` (2) or `rejected` (1) and `was_tier`. Nine rows have an `overload_note` (the four overload pairs and the function that keeps `0x004d5a90`'s name), and two an `x360_address_matching_overload` where the link's own address is the sibling overload. The 25 name-only links where only the bare method name matches, on another class, have `parameters: not applied`. Five functions with a virtual declaration are `not renamed` (reverted in review) |
| `fields.jsonl` | 4,852 | A renamed field: `class`, `offset`, `old`, `new`, `pdb_type`, and the `file`, `line` and `marker` of its declaration. A class can be declared in more than one file, so 117 `class` + `offset` pairs repeat; `file` + `class` + `offset` is unique |
| `classes.jsonl` | 72 | A renamed class: `old`, `new`, `pdb_name` for the 9 nested classes (named `Outer_Inner`), `source` (how the first pass found it), `tag` (`EXE` for the 50 from PC RTTI; for the 22 from high-tier links to their own methods here, `XBOX_RETAIL_PDB` (16) or `XBOX_DEBUG_PDB` (6) by the build most of those links come from, retail on a tie), `address` (the vtable, or the first such link from that build), `evidence`, `file`, `line`, `marker` |
| `globals.jsonl` | 517 | A renamed global: `address`, `old` (every name units used for it), `new`, `pdb_symbol`, `source`, `tier`, and `declarations`: the `file`, `line` and exact `marker` of each declaration |
| `parameters.jsonl` | 20,683 | A renamed parameter: `function` address, `index` (0-based), `old` (the name on `main` at the first definition this PR renamed, anchored definitions first, read from the source), `new`, `source`, and the function's link (`pdb_name`, `build`, `method`, `tier`). 30 rows have `applied_at` where the rename stands only at declarations. 31 rows in 23 functions have `name_found_in`: the name is not a symbol of the linked Xbox 360 function itself but of a sibling overload of the same name (27) or of the other build's copy (4) |
| `parameters_other_count_declarations.jsonl` | 453 | A parameter name on a declaration whose parameter count differs from the definition's (another view of the function), so its position may not match: `function`, `index`, `name`, `file`, `line`, `declaration_symbol`, `definition_parameters`, `declaration_parameters`. A known risk; see `PDB_NAMES.md` |
| `symbols_changed_by_class_or_type_renames.jsonl` | 682 | A function whose decorated names changed only because a class or type in its signature was renamed |
| `not_applied_summary.jsonl` | 768 | Counts of what the first pass did not apply, by kind and reason (before review) |

## Taking a group of names out again

Every renamed function, field, global and class has a `// src: XBOX_RETAIL_PDB` or `// src: XBOX_DEBUG_PDB` marker
(class names from RTTI: `// src: EXE`), so a search finds it. Field, global and class markers also give the member
offset or address and the old name. A function's marker gives its address; its old names are in
`functions_renamed.jsonl`. To reverse one, give the definition its old name from the marker or from these files, rename
its uses, and put the old decorated names back in `build/units.json` (`old_symbols` for functions). The check has to
stay byte-identical.
