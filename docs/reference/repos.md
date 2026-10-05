# Repository map

The work is spread over three repositories:

- [`documentation`](https://github.com/nzi-iitdelhi/documentation) is this handbook: the
  principles, roles and workflows.
- [`supply-side`](https://github.com/nzi-iitdelhi/supply-side) holds the scenario database
  (scenarios, versions and sensitivities), input generation, MACRO runs and results, with its
  API and web interface.
- [`demand-side`](https://github.com/nzi-iitdelhi/demand-side) holds the residential and
  transport demand pipelines, the RUMI-compatible engine, and the client for the central
  server.

Two outside projects matter too. [MacroEnergy.jl](https://github.com/macroenergy/MacroEnergy.jl)
is the model the supply side builds cases for, and [RUMI](https://github.com/prayas-energy/Rumi)
is the framework that the demand side's `pier_db/` must match exactly.

## Where does this belong?

| If it is | It goes in |
|---|---|
| A supply-side scenario version or sensitivity | The supply-side database, through the web interface or `yaml import`. Use `yaml export` if you want a copy as a file. |
| A demand-side experiment | `overrides:` in `run.yaml` in the demand-side repository |
| A base case, seed database or bulk input data | Neither repository. It is staged into ignored folders and tracked by its content hash or version. |
| A generated model input or a model output | Neither repository, because it can be regenerated from its definition |
| A run manifest | Alongside the results. Never deleted ([FAIR A2](../principles/fair.md)). |
| A design decision or an argued trade-off | `docs/` in the code repository it concerns |
| A role, workflow, principle or checklist | This handbook |
| A meeting decision that changes how the team works | This handbook, as an edit to the page it affects rather than a page of notes |

A simple test helps: if it describes how the team works, it belongs here. If it describes how
one module works, it belongs next to that module, where the person changing the code will see
it and keep it up to date.

## What is deliberately not in git

Some things are kept out of git on purpose, and each has its own way of being tracked or
recreated:

- **Base cases and the staged `data/` folder** are tracked by their content hash, which is
  recorded on `import`.
- **Supply-side run folders** (`runs/`) can be rebuilt with
  `python -m supply_side recreate <run_id>`.
- **Supply-side databases** (`db2.sqlite` and `results.duckdb`) are backed up on the server at
  every deploy, and built locally with `make init`.
- **Model outputs** are described by their run manifest, which is small and kept forever.
- **Demand-side `.duckdb` files** are fetched with `nzi-pipeline pull-seed`.
- **Virtual environments, `.deps/` and `node_modules/`** are recreated with `uv` on the supply
  side, `pip install -e ".[dev]"` on the demand side, and `npm install` for the web interface.

!!! danger
    Never fix a wrong generated file by committing a corrected copy. The fix belongs in the
    generator or in the definition.

## Sources this handbook was built from

The design principles and the FAIR page come from the FAIR principles (Wilkinson et al. 2016),
the team's FAIR/NZI deck, and the concept note on scalable scenario and sensitivity analysis
for MACRO. Role boundaries, the validation strategy and deployment targets come from the team
meetings on 24 August, 26 August and 1 September 2026. The supply-side workflows come from that
repository's README, Makefile, `docs/nomenclature.md` and `docs/specs/`, and were rewritten on
5 October 2026 for the scenario database. The demand-side workflows, tests, CI and known
discrepancies come from that repository's README, `CLAUDE.md` and `.github/`.

When one of those sources changes and this handbook does not, the handbook is wrong. Please
[fix it](contributing.md).
