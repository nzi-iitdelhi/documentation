# Nomenclature (supply-side)

!!! note "Source"
    Mirrors `docs/nomenclature.md` in the [`supply-side`](https://github.com/nzi-iitdelhi/supply-side/blob/main/docs/nomenclature.md) repo, which is authoritative. Change it there first, then here. Short definitions across both sides: [Glossary](../reference/glossary.md).

## 1. Domain terms

What the team, the PIs and the paper talk about.

1. **Base**: one import of DB1 (today a Macro input folder). Read-only.
2. **Scenario**: a named storyline built on the base, such as the reference or a net zero scenario. The project has four.
3. **Scenario version**: one state of a scenario, numbered v1, v2, and so on. It can start from any earlier version. Editable as a draft until its first run or activation, fixed after that.
   1. **Exploration**: a scenario version still being tried out to convince the PIs (status draft).
   2. **Active version**: the version the team currently stands behind (status active). At most one per scenario.
4. **Sensitivity**: a variation of a scenario around its active version, sweeping some parameters across several values to show robustness in the paper. It belongs to the scenario and stays tied to the version it was built on.
5. **Parameter**: a knob of the model, such as coal investment cost.
6. **Macro input**: one complete input handed to Macro. A scenario version gives one; a sensitivity gives one per sweep value.
7. **Run**: one solve of a macro input.
8. **Result**: what a run produced.

## 2. Specification terms

How a scenario version or a sensitivity is written down, in the UI or in YAML.

1. **Exploration param**: one precise change in a scenario version: a parameter, an operation, one amount, and a scope.
2. **Sensitivity param**: a named, reusable set of amounts for sensitivities: a parameter, an operation, a list of amounts, and a scope. Each amount gives a macro input.
3. **Operation**: the arithmetic applied: scale, add or set.
4. **Scope**: the file pattern, id pattern and periods that select which cells change.
5. **Version status**: draft, active or retired (was active, since replaced). Changed only by Mark active.
6. **Edit state**: editable or locked, on scenario versions and sensitivities. Locked once the first run is queued (a version also on first activation), with `locked_at`.
7. **Run status**: queued, running, done, failed or cancelled.

Note: renamed 2026-09-29 from adjustment / sweep setting (tables `adjustment`, `sweep_setting`, `sensitivity_sweep` are now `exploration_param`, `sensitivity_param`, `sensitivity_param_link`).

## 3. Implementation entities

Tables that exist for normalization, traceability or debugging. They are not things people talk about.

1. **base_file**: every file of a base, stored whole, so a macro input folder can be rebuilt.
2. **base_cell**: an index of a base's values, for scopes and dropdowns.
3. **sensitivity_param_link**: links a sensitivity to the sensitivity params it uses.
4. **macro_input_source**: which scenario version or sensitivity asked for a macro input, so identical inputs are stored once.
5. **cell_change**: the old and new value of each cell a macro input changes. A representation of the macro input, used for review and audit.
6. **manifest**: the record on a run of everything needed to recreate it.
7. **run_result**: the catalogue of a run's result tables.
8. **run_log**: a run's full solver log.
9. **run_batch**: the runs queued by one press of a Run button, with how many may run at once.
10. **event**: the append-only history of status changes and actions.

## 4. Conventions

1. **Base model**: every entity table has a UUID `id`, `created_at`, `created_by`, `updated_at`, `updated_by`, and `is_deleted`, `deleted_at`, `deleted_by`. Detail rows (base_file, base_cell, cell_change and the like) carry only `id` and `created_at` and follow their parent.
2. **Delete**: always a soft delete. It sets `is_deleted = true`; rows are never removed.

## 5. Stores

1. **Relational store**: the experiment record. Today SQLite.
2. **Columnar store**: results. Today DuckDB.

## 6. Terminologies for Pre-Scenario Pipeline

The pre-scenario pipeline makes the base model inputs from PIER and other data, before any scenario. It has three layers: bronze, silver and gold. The layer and step names are Andrew's, agreed on 26 October 2026. Example: `supply_side/medallion/README.md`.

### 6.1 Layers

1. **Bronze**: the values as each source gives them, with their units and their source. Bronze has no formulas.
2. **PIER2.1_Bronze**: the part of bronze that comes from PIER 2.1.
3. **NZI_Base_Bronze**: the part of bronze that does not come from PIER. It holds fixes to PIER data, extensions of PIER data (for example years after 2041), and data that is not in PIER.
4. **NZI_Base_Silver**: one value for each parameter, technology, state, vintage and year, selected from bronze. It has two parts:
   1. **NZI_Base_Silver.PIER_NZI**: in PIER names and PIER units.
   2. **NZI_Base_Silver.Macro_NZI**: in MACRO names and MACRO units, calculated from PIER_NZI.
5. **Gold**: the input files of one model (for example a MACRO case folder), written only from silver.

### 6.2 Steps

1. **PIER_TO_DB**: load PIER 2.1 files into bronze.
2. **OTHER_TO_DB**: load data that is not from PIER into bronze.
3. **DB_TO_DB**: select one bronze value for each silver key, and write it to PIER_NZI. No formulas.
4. **DB_TO_MACRO**: convert units and apply the workbook formulas, from PIER_NZI to Macro_NZI.
5. **Macro_TO_DB**: read MACRO's own definitions (for example default values and commodity types) from MacroEnergy.jl.
6. **TRANSLATE_MACRO_TO_MacroStructure**: write the MACRO case from Macro_NZI. This step applies the periods, the regions, the asset ids, the graph links and the file layout.

### 6.3 Other Terms

1. **Source kind**: the type of a bronze source: PIER, workbook, hand fix, assumption or projection.
2. **Source priority**: the order of source kinds that DB_TO_DB uses when two sources give the same value. The highest is first.
3. **Build**: one run that adds data to bronze or silver. Each build keeps who ran it, when, and from which commit. Earlier builds stay readable.
4. **Gap**: a value that silver cannot give yet, because a source or a unit is missing. Gold leaves a gap empty. The fix is a row in NZI_Base_Bronze.
5. **Base case**: the MACRO case that Andrew made by hand. The pipeline compares gold with it, but does not use it as a source.
