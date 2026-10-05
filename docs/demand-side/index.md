# Demand-side

**Repository:** [`nzi-iitdelhi/demand-side`](https://github.com/nzi-iitdelhi/demand-side)

The demand side works out how much energy is needed, broken down by sector, region, year and
time slice. It builds this up from surveys, appliance and vehicle stock, usage patterns and
efficiency trends. Its outputs become part of the approved base case that the supply side
runs on.

Two sectors are implemented so far, residential and transport, and both use the same command
line.

## How it fits together

```text
  seed Excel ──seed──▶ DuckDB ──run──▶ parameters ──export──▶ PIER CSVs
                                                                  │
                                                          pier_db run
                                                                  ▼
                                                  demand output (RUMI format) ──▶ central server
```

The pipeline starts from seed data in Excel, loads it into DuckDB, computes the model
parameters, and exports them as PIER files. The `pier_db` engine then turns those into demand
output in RUMI's format, which can be pushed to the shared central server.

`pier_db/` is our own implementation of [RUMI](https://github.com/prayas-energy/Rumi), and it
has to give numerically identical results. That one requirement explains most of the design
decisions on this side.

## What is in the repository

| Path | What it holds |
|---|---|
| `nzi_pipeline/` | The DuckDB pipeline, from seed Excel to parameters to PIER CSVs |
| `pier_db/` | The demand computation engine, which must match RUMI exactly |
| `frontend/` | A FastAPI web interface for editing inputs, running steps, taking snapshots and undoing changes |
| `tests/` | pytest unit and integration tests, including checks against golden outputs |
| `scripts/` | Wrappers for development and CI; GitHub Actions uses the same entry points |
| `General/Residential_Sector_Data/`, `PIER/` | The professor's reference data. **Read-only.** |
| `.github/workflows/ci.yml` | CI: lint, unit tests, integration tests and a Docker build |

!!! danger "Never write to the reference folders"
    `General/Residential_Sector_Data/Parameters/` and `PIER/Scenarios/` are the only
    independent check this project has on its numbers. If they are changed, we lose the
    ability to tell whether our pipeline is right.

## Everyday commands

```bash
python -m nzi_pipeline seed --sector residential   # loads the Excel seed data into DuckDB
python -m nzi_pipeline run  --sector residential   # runs the 7 pipeline steps
python -m pier_db run --sector D_RES               # computes the demand output
python -m frontend                                 # starts the web interface on port 8000
```

```bash
bash scripts/test.sh all           # lint, unit and integration tests (about 55 seconds)
bash scripts/test.sh unit          # 33 tests, under a second
bash scripts/test.sh integration   # 17 tests, about 55 seconds
```

The usual way of working with others is to pull a seed database, run locally, push your run
to the central server, and browse or pull other people's runs from there. The central server
goes to sleep when nobody is using it and the command line wakes it up, so you never need to
start it yourself.

## Things that trip people up

**Python version.** Use Python 3.10 to 3.12, not 3.13. `rumi` pins `numpy==1.26.4` and
`pandas==2.2.1`, which have no wheels for 3.13, so pip tries to compile numpy from source and
fails.

**R for the residential sector.** The residential pipeline calls `Rscript` to fit three
survey-weighted logistic regressions, so you need R with the `survey` package. Transport does
not need R.

**DuckDB file locking.** The web interface releases its database connection after each job so
the command line can use the file in between. If the interface crashes, it can leave the file
locked. Kill the process and restart it.

## Known open discrepancies

Some of our numbers are known not to match the reference yet. Please read this list before
reporting a bug. The authoritative version is in the repository's `CLAUDE.md`; if the two
disagree, `CLAUDE.md` is right and [this page needs fixing](../reference/contributing.md).

| Area | Status |
|---|---|
| Parameters (residential) | 32 of 34 identical |
| Demand (residential) | 37 of 56 within 1% |
| Cooling demand | AC +67%, FAN −25%, COOLER +42%. Needs the 3-archetype `FAN_ES_Demand` logic ported from `chcdh_calc.R` / `cooling_demand.py` into `s04_es_demand.py`. |
| Cooking fuel mix | BIOGAS −99% and NATGAS −80% per fuel, though the total is within 0.1%. The cause is TSR versus `modern_fuel_split`; the real fix is calibrating the data. |
| `EfficiencyLevelSplit` | A 0.75% residual for LIGHT_ELEC INCAND in 2031–33 |
| `ST_SEC` | 126K differences in values, caused by floating-point drift in the stock-flow calculation |
| Emissions | Off by a factor of 9.37. This needs supply-side `DemandMet` modelling, which is not implemented. |

## Where to go next

To run an experiment, read [Run an experiment](run-an-experiment.md). If you are changing
pipeline or engine code, read [Developer](developer.md). Merging and releasing are covered on
[Maintainer](maintainer.md), and approving results for publication on
[PI sign-off](../pi-signoff.md).
