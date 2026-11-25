# Developer (pre-scenario)

This document tells you where each part of the pre-scenario pipeline lives and where to make a change. The pipeline reads input files into Bronze, makes one clean value per key in Silver, and writes a MACRO case in Gold. Each input file has one module, and each kind of MACRO asset has one translator. A change for one file or one asset kind therefore stays in its own file, and pull requests that stay open for a long time do not conflict. For a short checklist, see [Add an input file](add-an-input-file.md). Paths on this page are under `supply_side/prescenario/` in the supply-side repository unless they say otherwise.

## 1. Layout

```
supply_side/prescenario/
├── inputs.py                 input sets: upload, allowed files, checks
├── pipeline.py               runs Bronze, Silver and Gold in order
├── bronze/
│   ├── sources/              one module per input file
│   │   ├── __init__.py       gathers the modules into the tables below
│   │   ├── pier/             PIER files (input set folder pier/)
│   │   └── projections/      NZI projection files (input set folder projections/)
│   ├── schemas/pier.py       the column types every schema uses
│   ├── pier_to_db.py         loads the PIER files
│   └── other_to_db.py        loads the projection files, NZI base rows and units
├── silver/
│   ├── db_to_db.py           Bronze -> one value per key (PRIORITY, technology lists)
│   └── derive.py             calculated values (RULES)
└── gold/
    ├── settings.py           the Gold page's choices
    └── macro/
        ├── __init__.py       translate: availability, nodes, case files; runs every translator
        ├── common.py         shared: FIELDS, CASE, Case, writers
        └── assets/           one translator per asset kind
```

## 2. Where to make a change

| You want to | Change |
| --- | --- |
| Read a new PIER or projection file | Add a module in `bronze/sources/pier/` or `bronze/sources/projections/` (section 3) |
| Change the columns or checks of a file | That file's module: `COLUMNS` or `Schema` |
| Give a column its Silver name | That file's module: `SAME_AS` |
| Give a column a unit | That file's module: `UNITS` or `UNITS_PER_TECHNOLOGY` |
| Choose which source wins when two give a value | `silver/db_to_db.py: PRIORITY` |
| Add a technology | `silver/db_to_db.py`: the technology lists (`THERMAL`, `VRE`, `HYDRO`, `STORAGE`) |
| Calculate a value from other values | `silver/derive.py: RULES` (section 4) |
| Map a Silver value to a MACRO field | The asset kind's translator in `gold/macro/assets/`, or `FIELDS` in `gold/macro/common.py` for thermal |
| Add a kind of MACRO asset | A new translator in `gold/macro/assets/` (section 5) |
| Change the case layout, file names or nodes | `gold/macro/common.py: CASE` and `nodes` |
| Change what the Gold page offers | `gold/settings.py` |
| Add a value that no file gives | A `bronze_nzi_base` row in a new migration in `migrations/versions/` |

## 3. Input file modules

### 3.1 What a module holds

Each module describes one file. Name the module after the file in snake case: `ECT_CapAddBounds.csv` is `ect_cap_add_bounds.py`.

| Name | Required | Meaning |
| --- | --- | --- |
| `PATH` | Yes | PIER: the path under the scenario folder (`Supply/Parameters/...`). Bronze reads the `Scenarios/Supply/8_REF_Unconstr` copy if it exists, else the `Default Data` copy. Projection: the file name. |
| `COLUMNS` | Yes | Which columns name the technology, vintage, year, place and time (3.2). Every other column is stored as a value. |
| `Schema` | Yes | One field per column (3.3). The input page checks each upload with it, and Bronze checks each row again when it loads. |
| `SAME_AS` | No | `{column: Silver parameter}` for a column whose name differs from the Silver name. A key can be `(column, technology)` when the name means something else for other technologies. |
| `UNITS` | No | `{column: (unit, description, where the unit comes from)}` for a column the file gives no unit for. |
| `UNITS_PER_TECHNOLOGY` | No | `{column: (template, description)}`. The template uses `{capacity}` (the technology's `CapacityUnit` in `EnergyConvTechnologies.csv`) and `{currency}` (`CurrencyUnit.csv`). |

`bronze/sources/__init__.py` gathers every module into `PIER_FILES`, `PROJECTION_FILES`, `SCHEMAS`, `SAME_AS`, `UNITS` and `UNITS_PER_TECHNOLOGY`. If two modules give the same key with different values, the import stops with an error.

### 3.2 `COLUMNS`

PIER files (`bronze/pier_to_db.py: ingest_pier`):

| Key | Value | Meaning |
| --- | --- | --- |
| `technology` | column name | The technology or carrier of the row. Required. |
| `vintage` | column name | The build year (`InstYear`). |
| `year` | column name | The year the value applies to. |
| `season`, `day_type`, `day_slice` | column name | The time of year and day. Silver has no day type: only an empty value or `ALLDAYS` is accepted. |
| `all` | text | The text that means "every one" in a vintage, year or place column, for example `"ALL"`. |
| `place`, `to` | column name | For a link: the region at each end. |
| `place_country`, `to_country` | column name | For a link between countries: the country at each end. |

Without `place`, the row's place is the finest of `ModelGeography`, `SubGeography1` and `SubGeography2` that is not empty.

Projection files (`bronze/other_to_db.py: ingest_projections`):

| Key | Value | Meaning |
| --- | --- | --- |
| `technology`, `vintage`, `year` | column name | As for PIER files. |
| `place` | tuple of column names | Coarsest first. The row's place is the last one that is not empty; none means the whole country. |
| `to` | column name | For a link: the other end. |
| `values` | tuple of column names | Keep only these columns as values. Without it, every other column is a value. |

### 3.3 `Schema`

A schema is a `Row` class (`bronze/schemas/pier.py`). It has one field per column, by the column's name. Every column is required, and a column the schema does not name stops the load. For a column name that is not a valid Python name, use `Annotated[type, Field(alias="the column name")]`.

| Type | Accepts |
| --- | --- |
| `Name` | Text that is not empty |
| `Place` | A place code, or empty for every place |
| `AYear` | A year |
| `EveryYear` | A year, `ALL`, or empty |
| `Every` | Any text, or empty (`ECT_Max_CUF`: empty or `ALL` is every one) |
| `Text` | Any text, or empty |
| `_value(Fraction)` | A number from 0 to 1, or empty |
| `_value(NonNeg)` | A number of 0 or more, or empty |
| `_value(Positive)` | A number above 0, or empty |
| `_value(NonNeg \| NoLimit)` | A number of 0 or more, or -1 for no limit |

An empty value cell is not stored.

### 3.4 Files that are not modules

1. `Common/Parameters/ModelGeography.csv`, `SubGeography2.csv` and `CurrencyUnit.csv` give the places and the currency. `pier_to_db.ingest_locations` and `ingest_parameters` read them by name, and `inputs.py: OWN_CHECKS` checks them.
2. The demand files (`demand/ELECTRICITY/ELECTRICITY_<year>.csv`) are copied into the case. They do not go through Bronze.
3. `run_HiGHS.jl` is copied into the case.
4. Values that no file gives (assumptions, hand fixes, MACRO case settings) are `bronze_nzi_base` rows, added by migrations.
5. MACRO's defaults come from MacroEnergy.jl (`bronze/macro_to_db.py`).

## 4. Silver

1. `silver/db_to_db.py` makes one value per parameter, technology, place and time.
2. When two sources give the same value, `PRIORITY` picks the source: hand fix, projection, assumption, macro case, PIER, macro default, base case copy.
3. Only the technologies in `TECHNOLOGIES` come into Silver. A new technology must be in one of the lists.
4. `silver/derive.py: RULES` holds the calculated values: `{Silver parameter: (function, unit, workbook cell)}`. The function takes `(p, state, vintage, year)` and reads other values through `p.number(...)`.
5. Silver keeps each value in the base unit of its dimension (MW, MWh, t, Million Rupees, year).

## 5. Gold: MACRO translators

### 5.1 A translator

Each module in `gold/macro/assets/` writes the files of one kind of asset. `gold/macro/assets/__init__.py` finds the modules by name, and `translate` runs each one. A module defines `write(ctx)`. `ctx` holds:

| Name | Meaning |
| --- | --- |
| `silver` | The Silver build: `silver["NZI"]` rows, places, ports |
| `case` | `Case(silver)`: the case rows, for example `case.by_year("SilverYear")` |
| `places` | `{state: region}` at the chosen number of regions |
| `names` | The region names, sorted |
| `periods` | `{period: Silver year}` |
| `first_period` | The first period |
| `tag` | `5r` or `25r`, for file names |
| `header` | `header(technology, year, place)`: the availability file and column of a build year |
| `like` | `{commodity: the commodity it acts like}` |
| `present` | The technologies left in the case after the Gold page's choices |
| `dest`, `base_case`, `report` | The output folder, the base case to compare with, and the comparison report |

To write a file, use `_write_assets(ctx.dest, ctx.base_case, path, assets, schema, ctx.report)` from `gold/macro/common.py`. It checks each asset against a schema in `gold/schemas/macro.py` and adds the comparison with the base case to the report.

### 5.2 Shared helpers in `gold/macro/common.py`

1. `asset(...)` and `builds(...)` build the legacy and new-build assets of a generator.
2. `_costs` and `_total` read costs and capacity totals.
3. `CASE` holds the case layout: file names, regions, own-asset rules.
4. `FIELDS` maps Silver parameters to MACRO fields for the thermal generators.

### 5.3 A new kind of asset

1. Copy `gold/macro/assets/links.py` to a new module.
2. Write `write(ctx)`, and add the asset schema to `gold/schemas/macro.py` if the kind has new fields.
3. To list the asset on the Gold page, add it to `ASSETS` in `gold/settings.py`.

## 6. Blast radius

The supply side sorts changes into rings ([Developer](../developer/index.md#blast-radius)). For
the pre-scenario pipeline they are:

| Ring | What you are touching | What you need to show |
|---|---|---|
| 1. Cosmetic | Docs, comments, the web pages' look | `make lint` passes |
| 2. Local | A test, a page, a check that only rejects bad input | `make test` passes |
| 3. Output | An input file module, `silver/`, `gold/macro/` | The full suite, plus the case before and after (section 7) |
| 4. Contract | A table, a migration, the manifest, the case layout | Ring 3, plus a tested migration and maintainer review |
| 5. Data | A `bronze_nzi_base` row, a hand fix, the base case copies | Ring 4, plus the list of changed values, and the PI told |

## 7. Tests and checks

1. Pre-scenario tests are in `tests/prescenario/` of the supply-side repository. Add one test that reads the new file and checks one value in the MACRO case.
2. `make test` runs every test, in about 5 minutes. A test that takes much longer points to a slow step: profile it and fix the cause.
3. `make macro_check` runs MACRO Diagnostics on a case.
4. A change that should not change the output: build the default case and one other case before and after, and compare the sha256 of every file. They must be identical.
5. A change that should change the output: say in the pull request which files change and why, and attach `make prescenario_diff` for the two Silver builds.

## 8. Pull requests

1. Add files. Do not edit the shared tables unless the change needs it, and say why in the pull request.
2. One input file or one asset kind per pull request.
3. Name the input file and its source (PIER version, projection set) in the pull request.

## 9. Shared files that a change can still need

These files are shared, so two pull requests can change the same lines:

1. `silver/db_to_db.py`: the technology lists and `PRIORITY`.
2. `silver/derive.py`: `RULES`.
3. `gold/macro/common.py`: `FIELDS` and `CASE`.
4. `gold/settings.py`: `ASSETS`.
5. `migrations/versions/`: a new migration must follow the last one. If two pull requests each add one, the second to merge must change its `down_revision`.
