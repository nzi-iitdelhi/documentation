# Run an experiment (supply-side)

Define a scenario change, run it, read results. No backend internals needed.

Two kinds of experiment, kept apart on purpose:

| | Exploration | Sensitivity |
|---|---|---|
| Is | A new **version** of a scenario, being tried out | Variations **around the active version** |
| Built from | Exploration params — one parameter, one operation, **one amount**, a scope | Sensitivity params — one parameter, one operation, **a list of amounts**, a scope |
| Gives | One MACRO input | One MACRO input per combination of amounts |
| For | Convincing the PIs the scenario is right | Showing robustness in the paper |

If your change is not expressible as a parameter, an operation (scale, add or set) and a
scope, go to [Developer](developer/index.md).

---

## Before you start

- [ ] `make test` passes on a clean checkout
- [ ] `make run-local` is up: UI at <http://127.0.0.1:3000>, API at `:8002`
- [ ] You know which scenario and which version you are deriving from

!!! warning "Runs are switched off in the UI right now"
    `RUNS_ENABLED = false` in `web/lib/api.ts`: every Run button opens a *Runs are
    disabled* dialog. You can still create and edit everything. To solve, use the CLI
    (step 4) or ask your maintainer.

---

## 1. Pick or create the scenario

**Scenarios** lists them; **New scenario** creates one. A scenario is a storyline (the
reference, a net zero path); the project has four. Most experiments are a new version or
a sensitivity of an existing one, not a new scenario.

---

## 2a. Exploration: a new version

On the scenario page, **Add exploration** starts a draft version from a locked version or
from the base — never from another draft. Then **Add exploration param**:

| Field | Meaning | Getting it wrong |
|---|---|---|
| Parameter | The knob, from the **Parameters** library (e.g. *Coal investment cost*) | — |
| Operation | `scale`, `add` or `set` | `add 1.05` where you meant `scale 1.05` |
| Amount | One number | Units: the library shows the parameter's unit |
| Scope | File pattern, id pattern, periods | Too broad matches cells you did not mean — the output still looks plausible |

**Review →** compares the version with its parent cell by cell. Read it.

- [ ] Only the cells you meant changed, by the amount you meant
- [ ] The cell count is what you expected (scope too broad or too narrow shows up here)
- [ ] The version's message says what changed in one sentence

## 2b. Sensitivity: around the active version

On the scenario page, **New sensitivity on v*N*** attaches it to the active version. Add
sensitivity params from the library (or **Create sensitivity param** if none fits).
**Review & run →** shows every combination before anything runs.

- [ ] Number of combinations is what you expected — it multiplies
- [ ] Each sensitivity param is named for what it varies (`standard_coal_cost`)

---

## 3. The lock rule

A draft version or sensitivity is editable, and every edit is recorded. It **locks** when
its first run is queued (a version also when first marked active), and never changes
after that. To change something locked: a new version from it, or **Duplicate** the
sensitivity.

This is what makes a result traceable — the thing that ran is exactly the thing on record.

---

## 4. Run

| Want | How |
|---|---|
| Run a version or sensitivity | **Run** in the UI (when enabled) — choose concurrency |
| Same, from the terminal, API up | `uv run python -m supply_side run <scenario-number>` — the API's worker solves it |
| Solve right here, API stopped | `uv run python -m supply_side run <n> --wait` |
| Small, fast check | add `--periods 2 --subperiods 2` |

`run` queues the scenario's **active** version. Runs are independent
([principle 7](../principles/index.md)); each gets `runs/<run_id>/` with its MACRO folder
and solver log. **Runs** shows the newest 200 with status, log, cancel and retry.

Solving needs Julia + MacroEnergy.jl: see `docs/how_to_run/run-macro-with-highs.md`, and
point `SUPPLY_JULIA` / `SUPPLY_JULIA_ENV` at them if they are not in `../.deps/`.

**Mark active** a version once it has a successful run and the PIs agree. At most one
active version per scenario; the old one becomes *retired*.

---

## 5. Read the results

Each version page shows its runs and a chart of its results. For your own analysis:

```sql
SELECT * FROM r_capacity JOIN run_context USING (run_id)
```

While the API is up it owns `results.duckdb`, so query through `GET /api/results/<name>`
instead; open DuckDB directly only with the API stopped.

- [ ] Every expected run is `done` — a `failed` run is a result, read its log
- [ ] Objective values finite and plausible (see `INF_FIX_CHANGELOG.md`)
- [ ] The **base case reproduces its known reference value** — if the base moved, every
      sensitivity is measured from the wrong origin
- [ ] Each result joined to `run_context` before charting, never matched by file name

---

## 6. Record

The database is the record; you mostly need to not break the chain.

- [ ] Version message and sensitivity name say what changed
- [ ] Export what you are reporting: `uv run python -m supply_side yaml export <n> --out defs/`
- [ ] Run IDs kept with any figure — `recreate <run_id>` rebuilds the exact input from them
- [ ] You can state in one sentence what changed vs the base case, matching the review

Going to a slide, partner, or paper? → [PI sign-off](../pi-signoff.md).

---

## YAML instead of the UI

A version or sensitivity can be written as YAML and imported; importing an unchanged file
is a no-op.

```yaml
# examples/yaml/nz_a_v1.yaml — a version
scenario: nz_a
description: Net zero A
message: coal investment cost up 5%
exploration_params:
  - parameter: Coal investment cost
    scale: 1.05
    where: {id: "*_coal"}
```

```yaml
# examples/yaml/nz_a_robustness.yaml — a sensitivity; v1 must be active or retired first
scenario: nz_a
version: 1
name: robustness
sensitivity_params: [standard_coal_cost, standard_phwr_cost]
```

```bash
uv run python -m supply_side yaml import examples/yaml/nz_a_v1.yaml
```

---

## Common mistakes

| Symptom | Likely cause |
|---|---|
| Results identical across sensitivities | Scope matched nothing, or the parameter does not affect the objective. Check the review's cell count. |
| Cannot edit a version | It is locked. Make a new version from it. |
| Cannot start a version from a draft | By design — lock the parent first (run it) or start from the base. |
| **Mark active** refused | The version has no successful run yet. |
| `run --wait` or DuckDB says the file is locked | The API is running and owns `results.duckdb`. Stop it, or drop `--wait`. |
| Solver reports infeasible | The scope or amount produced a physically meaningless case. A validation rule is missing — file it. |
