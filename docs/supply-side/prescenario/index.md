# Pre-scenario pipeline

**Code:** `supply_side/prescenario/` in [`nzi-iitdelhi/supply-side`](https://github.com/nzi-iitdelhi/supply-side)

The pre-scenario pipeline makes the base model inputs, before any scenario. It reads source
data (PIER 2.1, the NZI cost projections, hand fixes and MACRO's own defaults), keeps every
value with its source and unit, picks one clean value for each place and year, and writes a
complete MACRO case from it. That case is what the scenario database then starts from.

Every step records where each value came from, so a reviewer can trace any number in a MACRO
case back to the file and row it came from, and rebuild the case from its manifest.

## How it fits together

```text
  input set n        PIER folder, NZI projections, demand files (uploaded or the repository's inputs/)
      │  PIER_TO_DB, OTHER_TO_DB, MACRO_TO_DB
      ▼
  Bronze build       every value as its source gives it, with its unit and source
      │  DB_TO_DB (pick one value per key), derive (calculated values, base units)
      ▼
  Silver build       one value per parameter, technology, place, vintage and year
      │  settings chosen on the Gold page (regions, periods, solver, policy)
      ▼
  Gold: MACRO case   a case folder + manifest.json, checked with MACRO Diagnostics
      │
      ▼
  scenario database  (the base case every scenario starts from)
```

The three layers have fixed meanings, defined on the [Nomenclature](../nomenclature.md#6-terminologies-for-pre-scenario-pipeline)
page:

| Layer | What it holds | Rule |
|---|---|---|
| **Bronze** | Each input value exactly as given, with its unit and source file | No formulas. A file can be written back byte for byte. |
| **Silver** | One value per parameter, technology, place, vintage and year, in base units (MW, MWh, t, Million Rupees, year) | One vocabulary for every model. When two sources give a value, a fixed priority picks one. |
| **Gold** | The input files of one model, for example a MACRO case folder | Written only from Silver. The model's own names and units live here. |

## Builds keep history

Each build is stored, and nothing is overwritten:

- A **Bronze build** and a **Silver build** are made together from one input set.
- A build stores only the values that changed since the build it was made from. Unchanged
  values point back to the build where they last changed. This is a type 2 slowly changing
  dimension, keyed by build.
- A build can be made on top of any earlier build, and can apply just one input (PIER or the
  NZI projections). Builds therefore form a tree: two builds made from the same parent do not
  see each other's changes.
- Each MACRO case records its input set, Bronze and Silver builds, chosen settings, the code
  commit and the sha256 of every file in `manifest.json`.

## Where to go next

| You want to | Read |
|---|---|
| Upload inputs, build, and make or check a MACRO case | [Build a MACRO case](build-a-case.md) |
| Add a new input file, or a new kind of MACRO asset | [Add an input file](add-an-input-file.md) |
| Change the pipeline's code | [Developer](developer.md) |
| Deploy, back up or migrate the pre-scenario database | [Maintainer](../maintainer.md#pre-scenario-pipeline) |
