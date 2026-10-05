# Repository map

| Repository | Owns |
|---|---|
| [`documentation`](https://github.com/nzi-iitdelhi/documentation) | This handbook — principles, roles, workflows |
| [`supply-side`](https://github.com/nzi-iitdelhi/supply-side) | Scenario database (scenarios, versions, sensitivities), input generation, MACRO runs, results; API + UI |
| [`demand-side`](https://github.com/nzi-iitdelhi/demand-side) | Residential + transport demand pipelines, RUMI-compatible engine, central-server client |

**External:** [MacroEnergy.jl](https://github.com/macroenergy/MacroEnergy.jl) (the solver
supply-side generates cases for) · [RUMI](https://github.com/prayas-energy/Rumi) (what
`pier_db/` must stay identical to).

---

## Where does this belong?

| It is | Goes in |
|---|---|
| A scenario version / sensitivity | The supply-side database (UI or `yaml import`); `yaml export` for a file copy |
| A demand experiment definition | `demand-side` — `overrides:` in `run.yaml` |
| A base case, seed DB, bulk input data | **Neither** — staged into ignored dirs, tracked by content hash / version |
| A generated model input or output | **Neither** — reproducible from the definition |
| A run manifest | With the results. Never deleted ([FAIR A2](../principles/fair.md)). |
| A design decision or argued trade-off | `docs/` in the relevant code repo |
| A role, workflow, principle, checklist | **Here** |
| A meeting decision that changes how the team works | **Here** — as an edit to the affected page, not a notes dump |

!!! tip "The test"
    Describes *how the team works* → here. Describes *how this module works* → next to the
    module, where the person changing the code will update it.

---

## Deliberately not in git

| Not committed | Tracked instead by |
|---|---|
| Base cases, staged `data/` | Content hash, recorded on `import` |
| Supply run folders (`runs/`) | `python -m supply_side recreate <run_id>` |
| Supply databases (`db2.sqlite`, `results.duckdb`) | Server backups on each deploy; `make init` locally |
| Model outputs | The run manifest — small and permanent |
| `.duckdb` files | `nzi-pipeline pull-seed` |
| Virtualenvs, `.deps/`, `node_modules/` | `uv` (supply), `pip install -e ".[dev]"` (demand), `npm install` |

!!! danger
    Never fix a wrong generated artefact by committing it. The fix is in the generator or
    the definition.

---

## Sources this handbook was built from

| Source | Gave us |
|---|---|
| FAIR principles (Wilkinson et al. 2016) + the team's FAIR/NZI deck | [Design principles](../principles/index.md), [FAIR for models](../principles/fair.md) |
| Concept note: scalable scenario and sensitivity analysis for MACRO | The nine design principles |
| Meeting decisions, 24 Aug / 26 Aug / 1 Sep 2026 | Role boundaries, validation strategy, deployment targets |
| `supply-side/` README, Makefile, `docs/nomenclature.md`, `docs/specs/` | Supply-side workflows (rewritten 2026-10-05 for the scenario database) |
| `demand-side/` README, `CLAUDE.md`, `.github/` | Demand-side workflows, tests, CI, known diffs |

When one of those changes and this handbook does not, the handbook is wrong.
[Fix it](contributing.md).
