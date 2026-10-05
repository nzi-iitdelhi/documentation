# Run an experiment (demand-side)

A demand-side experiment means pulling a seed database, changing some of the documented
constants, running a sector, and pushing the result. You do not need to change any code. If
the thing you want to vary is not one of the constants available under `overrides:`, read the
[Developer](developer.md) page instead.

## Before you start

You need a Python 3.10–3.12 virtual environment with `pip install -e ".[dev]"` done. If you are
running the residential sector you also need R with the `survey` package. Check that
`bash scripts/test.sh unit` passes, and work on the `main_dev` branch unless someone tells you
otherwise.

## 1. Get a seed database

```bash
nzi-pipeline pull-seed
# or build one yourself: python -m nzi_pipeline seed --sector residential
```

Write down which seed you pulled. If two people run the same experiment on different seeds,
they get different numbers, and it can take a day to work out why.

## 2. Describe the experiment in `run.yaml`

Your changes go under `overrides:`. This is a short, flat list of constants that the sector
allows you to tune, and they replace the sector's defaults when the run starts. The YAML file
is the record of what you did ([principle 3](../principles/index.md)).

```yaml
sector: residential
overrides:
  FRIDGE_DC_CAGR: 0.04
  LED_CAGR: 0.12
  INIT_AC3_SHARE: 0.35
```

Every key has to exist in the sector's `TUNABLE` block. A typo should make the run fail
straight away; if a misspelt key silently does nothing, please report it. Sanity-check the
values too. A share has to be between 0 and 1, and a growth rate (CAGR) of `1.5` is almost
certainly a misplaced decimal point.

Commit the file before you run it, and make sure you can say in one sentence what the
experiment is testing.

!!! note "The list of tunable constants is short on purpose"
    If the constant you need is not exposed, do not reach into `defaults.py` at run time. Ask
    for it to be added to `TUNABLE` properly; the [Developer](developer.md) page explains how.

## 3. Run it

```bash
python -m nzi_pipeline run --sector residential
python -m pier_db run --sector D_RES
```

For transport, use `--sector transport` and `D_TRA`.

The web interface (`python -m frontend`, on port 8000) is handy for editing inputs, stepping
through the pipeline, taking snapshots and undoing changes, and it uses the same operations as
the command line ([principle 8](../principles/index.md)). The command line is still the
reproducible path, and it is the one CI tests.

!!! warning "Locked database"
    If the web interface crashes, it can leave the DuckDB file locked. Kill the process and
    start again.

## 4. Check the result before you believe it

First run the integration tests, which compare the outputs with the golden files at a relative
tolerance of `1e-6`:

```bash
bash scripts/test.sh integration
```

For the residential sector, also run the validators that compare against the professor's
reference:

```bash
cd General/Residential_Sector_Data/Residential_Workflow_FromInput
python validate_parameters_keyed.py   # PIER parameters against the reference
python validate_demand_keyed.py       # demand output against the reference
python compare_outputs.py             # pier_db output against the reference
```

The match rate should be at least as good as the recorded baseline. Any new mismatch should be
explained by your overrides rather than by a regression; if you are not sure, run the baseline
with no overrides first and compare. Differences that are already on the
[known list](index.md#known-open-discrepancies) are expected. A difference that is not on the
list is a finding, so please report it.

!!! danger "The known list does not mean other differences can be ignored"
    Cooling, the cooking fuel mix and emissions are still open problems. We can only make
    sense of those three because everything else matches.

## 5. Push the run and keep a record

```bash
nzi-pipeline push
```

Make sure the run went up with its identifier and metadata, that the `run.yaml` that produced
it is committed and pushed, and that the seed version is recorded alongside the results. You
should be able to name the run, its seed and its overrides without opening anything.

If the result is going to be made public, it also needs [PI sign-off](../pi-signoff.md).

## When something looks wrong

**`pip install` tries to compile numpy and fails.** You are on Python 3.13. Rebuild the virtual
environment with 3.12.

**The residential run fails when it calls `Rscript`.** R or the `survey` package is missing.
Transport does not need R.

**"database is locked".** A crashed web interface is still holding the connection. Kill it and
restart.

**Your numbers differ from a colleague's.** Most likely you are using different seed databases,
or one of you has uncommitted overrides. Compare seeds first.

**The golden-output check fails even though you did not change any maths.** Your overrides are
meant to change the outputs. Run the baseline without overrides to separate the two effects.

**You found a "bug" in the cooling or cooking fuel numbers.** These are already tracked; see the
[known discrepancies](index.md#known-open-discrepancies).
