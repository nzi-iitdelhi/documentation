# Run an experiment (supply-side)

Define a scenario, run it, read results. No generator internals needed.

If your change is not expressible in a YAML file, go to [Developer](developer.md).

---

## Before you start

- [ ] `make test` passes on a clean checkout
- [ ] `make tracking-check` passes
- [ ] You can state which base-case version you are deriving from

!!! danger "If `tracking-check` fails, stop"
    Your base case has drifted; every number will be incomparable with everyone else's.
    Run `make tracking-report` and take it to your maintainer. Do **not** run
    `tracking-lock` to make the error go away.

---

## 1. Write the experiment YAML

Lives in `experiments/`, **committed**. It is the permanent record
([principle 3](../principles/index.md)).

```yaml
label: reference_demand_supply_sensitivities
scenario_label: reference-demand

# Demand remains an opaque part of the approved native case.
data_demand_side: data/base-case
data_supply_side: data/base-case/supply

sweep_params:
  - nr_onshore_wind_investment_cost
  - nr_coal_max_capacity
validate: supply_case
```

| Field | Meaning | Getting it wrong |
|---|---|---|
| `label` | Names the experiment; becomes `runs/<label>/` | Collisions overwrite someone's results |
| `scenario_label` | The demand scenario this pairs with | Sensitivities attributed to the wrong demand future |
| `data_demand_side` / `data_supply_side` | Paths into the staged case | — |
| `sweep_params` | **Names of registered sweeps**, not inline values | Unregistered name fails at generate time — that is the point |
| `validate` | Which validator set runs | Invalid cases reach the solver and waste hours |

**Checklist**

- [ ] `label` unique and descriptive of the experiment, not the author
- [ ] Every `sweep_params` name is registered (if not → [Developer](developer.md))
- [ ] `validate` is set
- [ ] Committed **before** the run, not after

Worked examples: `experiments/sensitivity_01_coal_plus5pct.yaml` through `16`.

---

## 2. Generate configs

```bash
make configs EXPERIMENT=experiments/your_experiment.yaml
```

Wipes `runs/`, writes one complete config per sensitivity point.

!!! tip "Make small ones to read them"
    A config carries the whole case, so it is as large as the case.
    ```bash
    make configs CONFIG_PERIODS=2 CONFIG_SUBPERIODS=2
    ```
    Regenerate at full size before running for real.

**Check before burning compute**

- [ ] Expected number of configs under `runs/<label>/`
- [ ] `make check-case` passes
- [ ] Diff two neighbouring sensitivities — **only** the swept cells differ

That last check catches most mistakes.

---

## 3. Run

| Want | Command |
|---|---|
| Cheap local validation, no licence | `make sample` |
| Full local run | `make run` |
| One scenario | `make sensitivity-run SCENARIO=01` |
| All 16 + base, 3 at a time | `make run-scenarios PARALLEL=3` |

Logs and profiling land in `batch_runs/<run id>/`. Runs are independent
([principle 7](../principles/index.md)), so nothing is lost by running fewer at once.

---

## 4. Read the results

- [ ] Every expected run completed — a missing run is a result, find out why
- [ ] Objective values finite and plausible (see `INF_FIX_CHANGELOG.md`)
- [ ] The **base case reproduces its known reference value** — if the base moved, every
      sensitivity is measured from the wrong origin
- [ ] Each result joined to its manifest before charting

Start from the team's tornado workflow (`docs/images/tornado.py`) rather than new plotting code.

---

## 5. Record

- [ ] Experiment YAML committed and pushed
- [ ] Base-case version recorded with the results
- [ ] Run IDs and manifests preserved even if bulk outputs are deleted ([FAIR A2](../principles/fair.md))
- [ ] You can state in one sentence what changed vs the base case, matching the YAML diff

Going to a slide, partner, or paper? → [PI sign-off](../pi-signoff.md).

---

## Common mistakes

| Symptom | Likely cause |
|---|---|
| Results identical across sensitivities | Sweep matched nothing, or does not affect the objective. Diff the configs. |
| `tracking-check` fails after a run | Something wrote into the staged base case. Re-stage, then find what wrote to it. |
| Configs are gigabytes | Expected. Use `CONFIG_PERIODS`/`CONFIG_SUBPERIODS` to read them. |
| A hand-edited config gives a nicer number | Not a result. Fix the YAML or generator, regenerate. |
| Solver reports infeasible | A validator should have caught it — that is a missing validator, file it. |
