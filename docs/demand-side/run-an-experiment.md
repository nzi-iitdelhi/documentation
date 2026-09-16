# Run an experiment (demand-side)

Pull a seed DB, change documented constants, run a sector, push. No code changes.

If what you want to vary is not in `overrides:`, go to [Developer](developer.md).

---

## Before you start

- [ ] Python 3.10–3.12 venv, `pip install -e ".[dev]"` done
- [ ] R + `survey` installed **if** running residential
- [ ] `bash scripts/test.sh unit` passes
- [ ] On `main_dev` unless told otherwise

---

## 1. Get a seed database

```bash
nzi-pipeline pull-seed
# or: python -m nzi_pipeline seed --sector residential
```

- [ ] **Record which seed you pulled.** Two people on different seeds get different
      numbers and lose a day finding out why.

---

## 2. Define the experiment in `run.yaml`

Variation goes in `overrides:` — a flat set of curated tunable constants that mutate the
sector's defaults at startup. The YAML is the record ([principle 3](../principles/index.md)).

```yaml
sector: residential
overrides:
  FRIDGE_DC_CAGR: 0.04
  LED_CAGR: 0.12
  INIT_AC3_SHARE: 0.35
```

- [ ] Every key exists in the sector's `TUNABLE` block (a typo should fail loudly at
      startup — if it silently does nothing, report it)
- [ ] Values physically sensible — a share is in `[0,1]`; a CAGR of `1.5` is a slipped decimal
- [ ] Committed **before** the run
- [ ] You can say in one sentence what this experiment tests

!!! note "The tunable set is small on purpose"
    If your constant is not exposed, do not reach into `defaults.py` at run time — get it
    added to `TUNABLE` properly. See [Developer](developer.md).

---

## 3. Run

```bash
python -m nzi_pipeline run --sector residential
python -m pier_db run --sector D_RES
```

Transport: `--sector transport`, `D_TRA`.

The web UI (`python -m frontend`, :8000) is useful for editing inputs, stepping, snapshots
and undo — same backend operations as the CLI ([principle 8](../principles/index.md)).
The CLI is the reproducible path and the one CI exercises.

!!! warning "DB locking"
    A crashed frontend leaves the DuckDB locked. Kill the process, then restart.

---

## 4. Validate before you believe it

```bash
bash scripts/test.sh integration    # golden-output manifest check, 1e-6 rel tol
```

Residential reference validators:

```bash
cd General/Residential_Sector_Data/Residential_Workflow_FromInput
python validate_parameters_keyed.py   # PIER params vs professor
python validate_demand_keyed.py       # demand output vs professor
python compare_outputs.py             # pier_db output vs professor
```

- [ ] Match rate at or above the recorded baseline
- [ ] Any new mismatch is caused by **your override**, not a regression — run the
      no-override baseline first if unsure
- [ ] Differences are already in the [known list](index.md#known-open-discrepancies); if
      not, that is a finding — report it

!!! danger
    The known-diffs list is not a licence to ignore diffs. Cooling, cook fuel mix and
    emissions are open. Everything *else* matching is what makes those three interpretable.

---

## 5. Push and record

```bash
nzi-pipeline push
```

- [ ] Run pushed with its identifier and metadata
- [ ] The `run.yaml` that produced it is committed and pushed
- [ ] Seed DB version recorded with the results
- [ ] You can name the run, its seed, and its overrides without opening anything

Going public? → [PI sign-off](../pi-signoff.md).

---

## Common mistakes

| Symptom | Likely cause |
|---|---|
| `pip install` compiles numpy and fails | Python 3.13. Rebuild the venv with 3.12. |
| Residential run fails calling `Rscript` | R or `survey` missing. Transport does not need R. |
| "database is locked" | Crashed frontend holds the connection. Kill and restart. |
| Numbers differ from a colleague's | Different seed DB, or uncommitted overrides. Compare seeds first. |
| Manifest check fails, no maths changed | Your overrides are *supposed* to change outputs. Run the no-override baseline to separate them. |
| A "bug" in cooling or cook fuel numbers | Already tracked — [known discrepancies](index.md#known-open-discrepancies) |
