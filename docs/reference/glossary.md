# Glossary

!!! note "TODO"
    Seeded from the repos. Add terms as you hit ones this list does not cover — that is
    the fastest way to improve onboarding.

| Term | Meaning |
|---|---|
| **Base case** | A complete, versioned, immutable model case. Every experiment derives from one. |
| **Blast radius** | How far a code change can propagate. Determines what you must prove — see the Developer pages. |
| **Experiment** | A committed definition selecting a base case and naming sweeps + validation. |
| **Sensitivity** | One point in a sweep — e.g. "coal capex +5%". Expands into one run. |
| **Sweep / SweepParam** | A named, registered, reusable parameter variation referenced from an experiment YAML. |
| **Run config** | A generated, complete case with absolute values only. Never hand-edited. |
| **Manifest** | Metadata for a run: run ID, base version, transformations, tags, status. |
| **Lock (base-case)** | A recorded hash of the staged base case. `make tracking-check` verifies it. |
| **Golden output / manifest** | Committed reference outputs the demand-side test suite regresses against, at `1e-6` relative tolerance. |
| **Match rate** | Fraction of demand-side outputs matching the professor's reference. The number that matters for correctness. |
| **FAIR** | Findable, Accessible, Interoperable, Reusable — see [FAIR for models](../principles/fair.md). |
| **DOI** | Digital Object Identifier. Persistent, citable identifier for a published dataset or release. |
| **MACRO / MacroEnergy.jl** | The Julia energy-system optimisation model the supply side generates cases for. |
| **RUMI** | Prayas Energy's demand modelling framework. `pier_db/` must stay numerically identical to it. |
| **PIER** | Parameter/scenario file format on the demand side. |
| **HiGHS** | Open-source solver used for cheap local smoke runs — no Gurobi licence needed. |
| **Approach A / B** | The two independent supply-side implementations. One will be deleted. |
| **Seed database** | The starting DuckDB for a demand-side run, pulled with `nzi-pipeline pull-seed`. |
| **Central server** | EC2 host for demand-side runs. Sleeps when idle; the CLI wakes it. |
| **Tunable / `TUNABLE`** | The curated constants a demand-side `run.yaml` may override. |
| **STC / ELS / TSR / SEC / NC / UP** | RUMI demand-formula terms — see [Developer (demand)](../demand-side/developer.md). |

### Sector codes

| Code | Sector |
|---|---|
| `D_RES` / `residential` | Residential demand |
| `D_TRA` / `transport` | Transport demand |
