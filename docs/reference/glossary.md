# Glossary

!!! tip "Supply-side terms in full"
    The supply-side vocabulary, including table names and conventions, is in
    [Nomenclature](../supply-side/nomenclature.md).

!!! note "TODO"
    Seeded from the repos. Add terms as you hit ones this list does not cover — that is
    the fastest way to improve onboarding.

| Term | Meaning |
|---|---|
| **Base case** | A complete, versioned, immutable model case. Every experiment derives from one. |
| **Blast radius** | How far a code change can propagate. Determines what you must prove — see the Developer pages. |
| **Base** | One import of the base case into the supply-side database. Read-only; identified by its content hash. |
| **Scenario** | A named storyline built on the base — the reference, a net zero path. The supply side has four. |
| **Scenario version** | One state of a scenario: v1, v2 … Editable as a draft until its first run or activation, fixed after. |
| **Exploration** | A draft scenario version, still being tried out. |
| **Active / retired version** | Active: the one the team stands behind, at most one per scenario. Retired: was active, since replaced. |
| **Exploration param** | One precise change in a version: parameter, operation, one amount, scope. (Was *adjustment*.) |
| **Sensitivity** | Variations around a scenario's active version, to show robustness. One run per combination of amounts. |
| **Sensitivity param** | A named, reusable set of amounts: parameter, operation, list of amounts, scope. (Was *sweep / SweepParam*.) |
| **Parameter** | A knob of the model, such as coal investment cost. Listed in the Parameters library. |
| **Operation** | `scale`, `add` or `set`. |
| **Scope** | File pattern, id pattern and periods selecting which cells change. |
| **Macro input** | One complete input handed to MACRO, fingerprinted. Never hand-edited. |
| **Lock rule** | A version or sensitivity locks on its first run (a version also on first activation) and never changes after. |
| **Run** | One solve of a macro input: queued, running, done, failed or cancelled. |
| **Manifest** | Everything needed to recreate a run: base hash, input fingerprint, code commit, model versions, options. |
| **Soft delete** | Deleting sets `is_deleted`; rows are never removed and can be restored from Trash. |
| **Experiment** | Umbrella word: an exploration or a sensitivity (supply), a `run.yaml` (demand). |
| **Golden output / manifest** | Committed reference outputs the demand-side test suite regresses against, at `1e-6` relative tolerance. |
| **Match rate** | Fraction of demand-side outputs matching the professor's reference. The number that matters for correctness. |
| **FAIR** | Findable, Accessible, Interoperable, Reusable — see [FAIR for models](../principles/fair.md). |
| **DOI** | Digital Object Identifier. Persistent, citable identifier for a published dataset or release. |
| **MACRO / MacroEnergy.jl** | The Julia energy-system optimisation model the supply side generates cases for. |
| **RUMI** | Prayas Energy's demand modelling framework. `pier_db/` must stay numerically identical to it. |
| **PIER** | Parameter/scenario file format on the demand side. |
| **HiGHS** | Open-source solver used for cheap local smoke runs — no Gurobi licence needed. |
| **Approach A / B** | The two old supply-side implementations, removed 2026-09-29; on the `main-archive` branch. |
| **Seed database** | The starting DuckDB for a demand-side run, pulled with `nzi-pipeline pull-seed`. |
| **Central server** | EC2 host for demand-side runs. Sleeps when idle; the CLI wakes it. |
| **Tunable / `TUNABLE`** | The curated constants a demand-side `run.yaml` may override. |
| **STC / ELS / TSR / SEC / NC / UP** | RUMI demand-formula terms — see [Developer (demand)](../demand-side/developer.md). |

### Sector codes

| Code | Sector |
|---|---|
| `D_RES` / `residential` | Residential demand |
| `D_TRA` / `transport` | Transport demand |
