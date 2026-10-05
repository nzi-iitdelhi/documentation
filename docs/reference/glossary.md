# Glossary

The words below come up across the handbook, listed alphabetically. The supply side has a
fuller, more precise vocabulary, including its table names and conventions, on the
[Nomenclature](../supply-side/nomenclature.md) page.

!!! note "TODO"
    This list was started from the repositories and is incomplete. When you meet a word that
    is not here, please add it; it is one of the quickest ways to make onboarding easier.

| Term | Meaning |
|---|---|
| **Active version** | The scenario version the team currently stands behind. A scenario has at most one. |
| **Approach A / B** | The two old supply-side implementations, removed on 29 September 2026. They are kept on the `main-archive` branch. |
| **Base** | One import of the base case into the supply-side database. It is read-only and identified by its content hash. |
| **Base case** | A complete, versioned model case that every experiment starts from. It is never edited in place. |
| **Blast radius** | How far a code change could reach, which decides what you need to prove before it merges. See the Developer pages. |
| **Central server** | The EC2 machine that stores demand-side runs. It sleeps when idle, and the command line wakes it. |
| **DOI** | Digital Object Identifier: a permanent, citable identifier for a published dataset or release. |
| **Experiment** | A general word for a change you are trying out. On the supply side it is an exploration or a sensitivity; on the demand side it is a `run.yaml`. |
| **Exploration** | A draft scenario version that is still being tried out. |
| **Exploration param** | One precise change in a version: a parameter, an operation, one amount and a scope. It used to be called an *adjustment*. |
| **FAIR** | Findable, Accessible, Interoperable and Reusable. See [FAIR for models](../principles/fair.md). |
| **Golden outputs** | Reference outputs committed to the demand-side repository. The tests compare against them at a relative tolerance of `1e-6`. |
| **HiGHS** | The open-source solver the supply side uses, so no Gurobi licence is needed. |
| **Lock rule** | A supply-side version or sensitivity locks when its first run is queued (a version also when it is first made active), and never changes after that. |
| **Macro input** | One complete input handed to MACRO. It is generated and fingerprinted, and never edited by hand. |
| **MACRO / MacroEnergy.jl** | The energy-system optimisation model, written in Julia, that the supply side builds cases for. |
| **Manifest** | Everything needed to recreate a run: the base hash, the input fingerprint, the code commit, the model versions and the options. |
| **Match rate** | The share of demand-side outputs that match the professor's reference. This is the number that tells us whether the pipeline is right. |
| **Operation** | How a param changes a value: `scale`, `add` or `set`. |
| **Parameter** | One knob of the model, such as coal investment cost. They are listed in the Parameters library. |
| **PIER** | The parameter and scenario file format used on the demand side. |
| **Retired version** | A scenario version that used to be active and has since been replaced. |
| **RUMI** | Prayas Energy's demand modelling framework. The demand side's `pier_db/` must give numerically identical results. |
| **Run** | One solve of a macro input. Its status is queued, running, done, failed or cancelled. |
| **Scenario** | A named storyline built on the base, such as the reference case or a net zero pathway. The supply side has four. |
| **Scenario version** | One state of a scenario, numbered v1, v2 and so on. It can be edited as a draft until its first run or activation, and is fixed after that. |
| **Scope** | The file pattern, ID pattern and periods that select which cells a param changes. |
| **Seed database** | The starting DuckDB file for a demand-side run, fetched with `nzi-pipeline pull-seed`. |
| **Sensitivity** | A set of variations around a scenario's active version, used to show how robust a result is. It produces one run per combination of amounts. |
| **Sensitivity param** | A named, reusable list of amounts for a sensitivity: a parameter, an operation, the amounts and a scope. It used to be called a *sweep* or *SweepParam*. |
| **Soft delete** | Deleting a row only marks it with `is_deleted`. Nothing is removed, and deleted items can be restored from the Trash. |
| **STC, ELS, TSR, SEC, NC, UP** | Terms in RUMI's demand formula. See [Developer (demand-side)](../demand-side/developer.md). |
| **Tunable (`TUNABLE`)** | The short list of constants a demand-side `run.yaml` is allowed to override. |

## Sector codes

| Code | Sector |
|---|---|
| `D_RES` or `residential` | Residential demand |
| `D_TRA` or `transport` | Transport demand |
