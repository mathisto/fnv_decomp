# Mapping for the Xbox 360 PDB names

This branch holds the machine-readable mapping behind the pull request
[momusdigital/fnv_decomp#42](https://github.com/momusdigital/fnv_decomp/pull/42), "Names from the Xbox 360 development
build PDBs". [PDB_NAMES.md](https://github.com/mathisto/fnv_decomp/blob/de63646ca1dce39db99fb0ef7488f99441011505/PDB_NAMES.md) describes how the names
were linked and applied.

The files match commit [`de63646c`](https://github.com/mathisto/fnv_decomp/commit/de63646ca1dce39db99fb0ef7488f99441011505), the head of the pull
request when they were written; file and line numbers refer to that commit. They are JSON Lines, one object per line.
They hold extracted names and metadata (addresses, offsets, link methods); no PDB files or executable bytes. Addresses
are PC addresses in `FalloutNV.exe` unless the key says `x360_`. `old` is the name on `main` before the pull request.

| File | Rows | One row is |
|---|---|---|
| `functions_renamed.jsonl` | 3,146 | A function whose own name changed: `address`; `old_symbols` and `new_symbols`, the decorated names `build/units.json` gives it, C names included, that changed; `pdb_name`; `build` (`retail` or `debug`); link `method`; `tier`; and the `// src:` `marker` in the source |
| `functions_linked.jsonl` | 37,460 | Every PC function linked to a PDB function, renamed or not: `status` (`renamed`, `already named`, `not renamed`) and `reason`. Review conflicts have `tier` `disputed` (2) or `rejected` (1) and `was_tier`. Nine rows have an `overload_note` (the four overload pairs and the function that keeps `0x004d5a90`'s name), and two an `x360_address_matching_overload` where the link's own address is the sibling overload. The 25 name-only links where only the bare method name matches, on another class, have `parameters: not applied` |
| `fields.jsonl` | 4,852 | A renamed field: `class`, `offset`, `old`, `new`, `pdb_type`, and the `file`, `line` and `marker` of its declaration. A class can be declared in more than one file, so 117 `class` + `offset` pairs repeat; `file` + `class` + `offset` is unique |
| `classes.jsonl` | 77 | A renamed class: `old`, `new`, `source` (how the first pass found it), `tag` (`EXE` for the 54 from PC RTTI; for the 23 from high-tier links to their own methods here, `XBOX_RETAIL_PDB` (17) or `XBOX_DEBUG_PDB` (6) by the build most of those links come from, retail on a tie), `address` (the vtable, or the first such link), `evidence`, `file`, `line`, `marker` |
| `globals.jsonl` | 517 | A renamed global: `address`, `old` (every name units used for it), `new`, `pdb_symbol`, `source`, `tier`, and `declarations`: the `file`, `line` and exact `marker` of each declaration |
| `parameters.jsonl` | 20,699 | A renamed parameter: `function` address, `index` (0-based), `old` (`(unnamed)` when it had none), `new`, `source`, and the function's link (`pdb_name`, `build`, `method`, `tier`). 49 rows have `name_found_in`: the name is not a symbol of the linked Xbox 360 function itself but of a sibling overload (34) or of the other build's copy (15) |
| `symbols_changed_by_class_or_type_renames.jsonl` | 691 | A function whose decorated names changed only because a class or type in its signature was renamed |
| `not_applied_summary.jsonl` | 768 | Counts of what the first pass did not apply, by kind and reason (before review) |

## Taking a group of names out again

Every renamed function, field, global and class has a `// src: XBOX_RETAIL_PDB` or `// src: XBOX_DEBUG_PDB` marker
(class names from RTTI: `// src: EXE`), so a search finds it. Field, global and class markers also give the member
offset or address and the old name. A function's marker gives its address; its old names are in
`functions_renamed.jsonl`. To reverse one, give the definition its old name from the marker or from these files, rename
its uses, and put the old decorated names back in `build/units.json` (`old_symbols` for functions). The check has to
stay byte-identical.
