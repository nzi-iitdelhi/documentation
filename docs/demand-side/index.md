# Demand-side

**Repo:** [`nzi-iitdelhi/demand-side`](https://github.com/nzi-iitdelhi/demand-side)

Computes **how much energy is needed** — by sector, region, year, time slice — from
surveys, appliance stock, usage and efficiency trends. Outputs feed the supply side as an
approved native case.

Sectors implemented: **residential** and **transport**, same CLI shape.

---

## The flow

```text
  seed Excel ──seed──▶ DuckDB ──run──▶ parameters ──export──▶ PIER CSVs
                                                                  │
                                                          pier_db run
                                                                  ▼
                                                  demand output (RUMI format) ──▶ central server
```

`pier_db/` reimplements [RUMI](https://github.com/prayas-energy/Rumi) and must stay
numerically identical. That constraint drives most of the design.

---

## Repository map

| Path | What |
|---|---|
| `nzi_pipeline/` | DuckDB pipeline: seed Excel → parameters → PIER CSVs |
| `pier_db/` | Demand computation engine — RUMI-identical |
| `frontend/` | FastAPI + web UI: edit inputs, run steps, snapshots, undo |
| `tests/` | pytest — unit + integration, incl. golden-output manifest checks |
| `scripts/` | Dev/CI wrappers — the same entry points GitHub Actions uses |
| `General/Residential_Sector_Data/`, `PIER/` | Professor's reference data. **Read-only.** |
| `.github/workflows/ci.yml` | lint + unit + integration + docker-build |

!!! danger "Read-only reference folders"
    `General/Residential_Sector_Data/Parameters/` and `PIER/Scenarios/` are the only
    independent check this project has. Never write to them.

---

## Commands

```bash
python -m nzi_pipeline seed --sector residential   # Excel → DuckDB
python -m nzi_pipeline run  --sector residential   # the 7 pipeline steps
python -m pier_db run --sector D_RES               # compute demand output
python -m frontend                                 # FastAPI UI on :8000
```
```bash
bash scripts/test.sh all           # lint + unit + integration (~55s)
bash scripts/test.sh unit          # 33 tests, <1s
bash scripts/test.sh integration   # 17 tests, ~55s
```

Typical remote workflow: **pull a seed DB → run locally → push to central → browse and
pull others' runs.** The central server sleeps when idle; the CLI wakes it. You never run
it yourself.

---

## Environment constraints

| Constraint | Why |
|---|---|
| Python **3.10–3.12, not 3.13** | `rumi` pins `numpy==1.26.4` / `pandas==2.2.1`, no 3.13 wheels → numpy compiles from source and fails |
| **R + `survey`** for residential | `Rscript` fits three survey-weighted logistic regressions. Transport has no R dependency. |
| DuckDB file locking | Frontend releases its connection after each job so the CLI can run between. A crashed frontend leaves it locked — kill the process, restart. |

---

## Known open discrepancies

Read before reporting a "bug". **Authoritative version lives in the repo's `CLAUDE.md`** —
if this disagrees, `CLAUDE.md` wins and [this page needs fixing](../reference/contributing.md).

| Area | Status |
|---|---|
| Parameters (residential) | 32/34 identical |
| Demand (residential) | 37/56 within 1% |
| Cooling demand | AC +67%, FAN −25%, COOLER +42% — needs the 3-archetype `FAN_ES_Demand` port from `chcdh_calc.R` / `cooling_demand.py` into `s04_es_demand.py` |
| Cook fuel mix | BIOGAS −99%, NATGAS −80% per fuel, total within 0.1% — TSR vs `modern_fuel_split`; real fix is data calibration |
| `EfficiencyLevelSplit` | LIGHT_ELEC INCAND 0.75% residual, 2031–33 |
| `ST_SEC` | 126K value diffs from stock-flow FP drift |
| Emissions | 9.37× — needs supply-side `DemandMet` modelling, not implemented |

---

## Next

| You are | Go to |
|---|---|
| Running an experiment | [Run an experiment](run-an-experiment.md) |
| Changing pipeline or engine code | [Developer](developer.md) |
| Merging, releasing | [Maintainer](maintainer.md) |
| Signing off results | [PI sign-off](../pi-signoff.md) |
