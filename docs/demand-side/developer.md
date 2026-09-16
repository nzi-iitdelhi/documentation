# Developer (demand-side)

Changing the pipeline, the engine, the frontend, or the tests.

Defining constraint: **`pier_db/` must stay numerically identical to
[RUMI](https://github.com/prayas-energy/Rumi).** Most rules follow from that.

---

## Blast radius

| Ring | Touching | Changes a published number? | Required |
|---|---|---|---|
| **1 Cosmetic** | Docs, comments, lint fixes | No | `ruff check tests` |
| **2 Local** | Frontend screen, a script, a test | No | Unit tests |
| **3 Pipeline step** | `s0*_*.py`, parameter computation, export | **Yes** | Full suite + validators vs professor + match-rate comparison |
| **4 Engine** | `pier_db/` demand maths, RUMI formulas | **Yes, everywhere** | Ring 3 + how it stays RUMI-identical + maintainer review |
| **5 Data / schema** | Seed data, DuckDB schema, golden manifests | **Yes, silently** | Ring 4 + regenerated fixtures + PI informed |

**How to measure it** — run the validators before and after. The **match rate** is the
number that matters:

```bash
cd General/Residential_Sector_Data/Residential_Workflow_FromInput
python validate_parameters_keyed.py && python validate_demand_keyed.py && python compare_outputs.py
```

!!! danger "'Tests pass' is not evidence for ring 3+"
    Golden manifests encode *our current* output, known errors included. They can stay
    green while you drift further from the reference. Record before/after match rates.

---

## Keeping RUMI identical

Work from the RUMI source, not intuition. For orientation only — the source is authoritative:

| Quantity | Formula |
|---|---|
| Demand | `NC × NI × ES_Demand × UP × TSR × ELS × SEC`, summed over efficiency levels and STC combinations |
| GT profile | `demand × GT / sum(GT × days_in_season)` |
| Season energy demand | `EnergyDemand × DayTypeWeight × NumDaysInSeason` |
| `seasons_size` | Reference year 2019 (non-leap, 365 days) |

!!! warning "A cleaner formula is a different formula"
    Floating-point association order is part of the contract — `ST_SEC` already carries
    126K value diffs from stock-flow FP drift. Reordering operations is a behaviour change.

---

## Read-only reference folders

Never write to `General/Residential_Sector_Data/Parameters/` or `PIER/Scenarios/`.

**Corollary:** generate every static file in the pipeline (`s06_export.py`). Never copy one
from the professor's folder — a copied file looks like agreement and proves nothing.

---

## Tests

```bash
bash scripts/test.sh all | unit | integration
```

| Suite | Covers |
|---|---|
| `tests/unit/` | In-memory DuckDB: `edit`, `snapshot`, `_quote`, `pier_db` static maps |
| `test_db_lifecycle.py` | DB wrapper, persistence, read-only mode |
| `test_real_db_smoke.py` | Auto-skips if `residential.duckdb` missing or locked |
| `test_pipeline_on_fixture.py` / `test_transport_...` | Full pipeline e2e on tiny fixtures |
| `test_manifest_check.py` | Golden-output regression, `1e-6` rel tolerance |

Fixtures: Parquet slices of the real DB (region NR, sub-geo DL), 1.5 MB, in
`tests/fixtures/{residential,transport}_min/`.

### Regenerating fixtures and golden manifests

```bash
python scripts/build_residential_fixture.py
python scripts/build_transport_fixture.py
python scripts/build_manifest.py residential   # ONLY after an intentional maths change
```

!!! danger "Regenerating a manifest is a claim, not a chore"
    It overwrites golden outputs with whatever your code now produces — the test then
    passes by definition. Before running it:

    - [ ] You **intended** to change the numbers
    - [ ] You can explain every changed value
    - [ ] Validators vs the professor got **better**, not just different
    - [ ] Before/after match rates are in the PR body
    - [ ] A maintainer agreed — this is not a developer decision

---

## Lint

`tests/` is **required**; `nzi_pipeline/` and `pier_db/` are **advisory** (~140 existing
warnings, logged but non-blocking).

```bash
ruff check tests                    # must pass
ruff check nzi_pipeline pier_db     # advisory today
```

Clean warnings in the code you touched. Do not mix a repo-wide lint PR with a behaviour change.

---

## Branches

`main`, `op_dev` and `storyline` all exist and CI runs on PRs into each. Most current work
is on `main_dev` — confirm your target with your maintainer.

---

## Never OK

- Writing into the professor's reference folders
- Copying a static file instead of generating it
- Regenerating golden manifests to make a failing test pass
- "Simplifying" a RUMI formula
- Committing a `.duckdb` file
- Holding a DB connection across jobs in the frontend (`_release_db` in `frontend/app.py`)

---

## PR checklist

- [ ] Blast-radius ring stated
- [ ] `bash scripts/test.sh all` passes
- [ ] `ruff check tests` passes
- [ ] Ring 3+: before/after match rates in the PR body
- [ ] Golden manifests regenerated only with justification + maintainer agreement
- [ ] Fixtures regenerated if the schema changed
- [ ] A test exists that fails if this change is reverted
- [ ] Known-diffs list updated if your change moved one
- [ ] Handbook updated if a documented workflow changed

→ [Maintainer](maintainer.md)
