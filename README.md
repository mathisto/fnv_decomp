# Mapping for the Xbox 360 PDB names

This branch holds the machine-readable mapping behind the pull request
[momusdigital/fnv_decomp#42](https://github.com/momusdigital/fnv_decomp/pull/42), "Names from the Xbox 360 development
build PDBs". [PDB_NAMES.md](https://github.com/mathisto/fnv_decomp/blob/pdb-names/PDB_NAMES.md) on the pull request's
branch describes how the names were linked and applied.

The files are JSON Lines, one object per line, and match commit [`62d07faf`](https://github.com/mathisto/fnv_decomp/commit/62d07faf18f0c7c2a5d094f712a98193dc6dfe80), the head of the pull request when they were written. They hold names, addresses,
offsets and link methods; no PDB or executable data. Addresses are PC addresses in `FalloutNV.exe` unless the key says
`x360_address`. `old` is the name on `main` before the pull request.

| File | Rows | One row is |
|---|---|---|
| `functions_renamed.jsonl` | 3,231 | A function whose own name changed: `address`, `old_symbols` and `new_symbols` (every decorated name `build/units.json` gives it), `pdb_name`, `build` (`retail` or `debug`), link `method`, `tier`, and the `// src:` `marker` in the source |
| `functions_linked.jsonl` | 37,460 | Every PC function linked to a PDB function, renamed or not: `status` (`renamed`, `already named`, `not renamed`), and `reason` when not renamed. The 7 conflicts found in review have `tier: disputed` and `was_tier`; the 5 functions that keep a disputed name have a `dispute` note |
| `fields.jsonl` | 4,852 | A renamed field: `class`, `offset`, `old`, `new`, `pdb_type`, and the `file`, `line` and `marker` of its declaration. A class can be declared in more than one file, so 117 `class` + `offset` pairs repeat; `file` + `class` + `offset` is unique |
| `classes.jsonl` | 81 | A renamed class: `old`, `new`, `source` (PC RTTI, tagged `EXE`, or linked PDB methods, tagged `XBOX_RETAIL_PDB`), `file`, `line`, `marker` |
| `globals.jsonl` | 517 | A renamed global: `address`, `old` (every name units used for it), `new`, `pdb_symbol`, `source`, `marker`, and every `declarations` line that carries the marker |
| `parameters.jsonl` | 20,730 | A renamed parameter: `function` address, `index` (0-based), `old` (`(unnamed)` when it had none), `new`, `source`, and the function's link: `pdb_name`, `build`, `method`, `tier` |
| `symbols_changed_by_class_or_type_renames.jsonl` | 686 | A function whose decorated names changed only because a class or type in its signature was renamed |
| `not_applied_summary.jsonl` | 768 | Counts of what was not applied, by kind and reason |

## Taking a group of names out again

Every renamed function, field, global and class has a `// src: XBOX_RETAIL_PDB` or `// src: XBOX_DEBUG_PDB` marker
(class names from RTTI: `// src: EXE`), so a search finds it. Field, global and class markers also give the member
offset or address and the old name; a function's marker gives its address, and its old names are in
`functions_renamed.jsonl`. To reverse one, give the definition its old name from the marker or from these files, rename its uses, and put the old
decorated names back in `build/units.json` (`old_symbols` for functions). The check has to stay byte-identical.
