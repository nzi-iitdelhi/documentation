# Developer (demand-side)

This page is for anyone changing the demand-side pipeline, the engine, the web interface or
the tests.

One requirement shapes almost everything here: **`pier_db/` must give numerically identical
results to [RUMI](https://github.com/prayas-energy/Rumi).** Most of the rules below follow
from that.

## Blast radius

Before you start, work out how far your change could reach. That decides how much you need to
prove before it merges.

| Ring | What you are touching | Can it change a published number? | What you need to show |
|---|---|---|---|
| 1. Cosmetic | Docs, comments, lint fixes | No | `ruff check tests` passes |
| 2. Local | A web interface screen, a script, a test | No | The unit tests pass |
| 3. Pipeline step | `s0*_*.py`, parameter computation, export | Yes | The full suite, the validators against the reference, and a comparison of match rates |
| 4. Engine | The demand maths in `pier_db/`, RUMI formulas | Yes, everywhere | Ring 3, plus an explanation of how it stays identical to RUMI, and maintainer review |
| 5. Data or schema | Seed data, the DuckDB schema, golden manifests | Yes, without anyone noticing | Ring 4, plus regenerated fixtures, and the PI told |

To measure the effect of a change, run the validators before and after it. The **match rate**
against the reference is the number that matters:

```bash
cd General/Residential_Sector_Data/Residential_Workflow_FromInput
python validate_parameters_keyed.py && python validate_demand_keyed.py && python compare_outputs.py
```

!!! danger "Passing tests are not enough for ring 3 and above"
    The golden manifests record what our code produces today, known errors included. They can
    stay green while you drift further away from the reference. Record the match rates before
    and after your change.

## Keeping the engine identical to RUMI

Work from the RUMI source code rather than from intuition. The formulas below are only a guide
to find your way around; the source is the authority.

| Quantity | Formula |
|---|---|
| Demand | `NC × NI × ES_Demand × UP × TSR × ELS × SEC`, summed over efficiency levels and STC combinations |
| GT profile | `demand × GT / sum(GT × days_in_season)` |
| Season energy demand | `EnergyDemand × DayTypeWeight × NumDaysInSeason` |
| `seasons_size` | Uses 2019 as the reference year (not a leap year, 365 days) |

!!! warning "A tidier formula is a different formula"
    The order in which floating-point operations happen is part of what has to match.
    `ST_SEC` already has 126K value differences caused by floating-point drift in the
    stock-flow calculation. Reordering operations changes behaviour, even if the maths looks
    the same on paper.

## Read-only reference folders

Never write to `General/Residential_Sector_Data/Parameters/` or `PIER/Scenarios/`.

It follows that every static file in the pipeline should be generated (in `s06_export.py`),
never copied from the professor's folder. A copied file will always agree with the reference,
so it proves nothing.

## Tests

```bash
bash scripts/test.sh all | unit | integration
```

| Suite | What it covers |
|---|---|
| `tests/unit/` | In-memory DuckDB: `edit`, `snapshot`, `_quote`, and the static maps in `pier_db` |
| `test_db_lifecycle.py` | The database wrapper, persistence, and read-only mode |
| `test_real_db_smoke.py` | Skips itself if `residential.duckdb` is missing or locked |
| `test_pipeline_on_fixture.py`, `test_transport_...` | The full pipeline end to end, on small fixtures |
| `test_manifest_check.py` | Regression against the golden outputs, at a relative tolerance of `1e-6` |

The fixtures are small Parquet extracts of the real database (region NR, sub-geography DL),
1.5 MB in total, kept in `tests/fixtures/{residential,transport}_min/`.

### Regenerating fixtures and golden manifests

```bash
python scripts/build_residential_fixture.py
python scripts/build_transport_fixture.py
python scripts/build_manifest.py residential   # only after an intentional change to the maths
```

!!! danger "Regenerating a manifest changes what counts as correct"
    `build_manifest.py` overwrites the golden outputs with whatever your code produces now,
    so the test will pass by definition afterwards. Only run it when:

    - [ ] you meant to change the numbers
    - [ ] you can explain every value that changed
    - [ ] the validators against the reference got better, not just different
    - [ ] the before-and-after match rates are in the pull request description
    - [ ] a maintainer has agreed, because this is not a decision for the developer alone

## Lint

Lint is required to pass for `tests/`. For `nzi_pipeline/` and `pier_db/` it is advisory for
now: there are about 140 existing warnings, which are logged but do not block a merge.

```bash
ruff check tests                    # must pass
ruff check nzi_pipeline pier_db     # advisory for now
```

Fix the warnings in code you touch, but keep repository-wide lint clean-ups in their own pull
request, separate from any change in behaviour.

## Branches

`main`, `op_dev` and `storyline` all exist, and CI runs on pull requests into each of them.
Most current work happens on `main_dev`, so check with your maintainer which branch to target.

## Things to avoid

- Writing into the professor's reference folders.
- Copying a static file instead of generating it.
- Regenerating golden manifests to make a failing test pass.
- "Simplifying" a RUMI formula.
- Committing a `.duckdb` file.
- Holding a database connection across jobs in the web interface (see `_release_db` in
  `frontend/app.py`).

## PR checklist

Before asking for review, check that:

- [ ] the pull request says which ring the change is in
- [ ] `bash scripts/test.sh all` passes
- [ ] `ruff check tests` passes
- [ ] for ring 3 and above, the before-and-after match rates are in the description
- [ ] golden manifests were only regenerated with a justification and a maintainer's agreement
- [ ] fixtures were regenerated if the schema changed
- [ ] there is a test that would fail if the change were reverted
- [ ] the known-discrepancies list is updated if your change moved one of them
- [ ] this handbook is updated if a documented workflow changed

The [Maintainer](maintainer.md) page describes what happens next.
