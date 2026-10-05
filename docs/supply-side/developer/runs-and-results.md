# Runs and results

This page follows a version or sensitivity from the moment it is generated to the moment its
results are rows in DuckDB. Most of this code is ring 3 or 4, so read about
[blast radius](index.md#blast-radius) before changing it.

## Generating a model input

Generation happens in four steps:

```text
  version / sensitivity
      ▼  resolve        exploration or sensitivity params → cell changes (path, row id, field, old, new)
      ▼  check          every combination valid before anything is stored
      ▼  fingerprint    sha256(base content hash + final cell values)
  macro_input           reused if the fingerprint exists; else stored with its cell_change rows
                        and the hash of the folder it builds
```

First, the params are **resolved** into a list of cell changes. Each change names a file, a
row, a field, and the old and new values. For a sensitivity this happens once per combination
of amounts, and every combination is **checked** before anything is stored. The final cell
values, together with the base case's content hash, are then hashed into a **fingerprint**. If
a macro input with that fingerprint already exists, it is reused. Otherwise a new one is
stored with its cell changes and the hash of the folder it produces.

Because of this, identical inputs are only stored once. If two versions end up changing the
same cells to the same values, they share a macro input, and `macro_input_source` records
which versions asked for it.

The code is spread over a few files:

| File | What it does |
|---|---|
| `macro_inputs/resolve.py` | Turns params into cell changes, builds sensitivity combinations, and computes the fingerprint |
| `macro_inputs/apply.py` | Writes cell changes into the text of the base files |
| `macro_inputs/services.py` | `generate_version`, `generate_sensitivity` and `rebuild` |
| `library/arithmetic.py` | The `scale`, `add` and `set` operations |

`rebuild` recreates an input folder from the stored base files and cell changes, and refuses
if the result does not match the recorded folder hash. When that happens it writes an
`integrity_failed` event. This check is what lets us trust that a past run can be reproduced,
so please do not weaken it.

## The worker

There is one worker per deployment. It runs inside the API process when you start `make api`,
or on its own with `make worker`. It picks up queued runs, oldest batch first.

Two limits control how many runs happen at once. `SUPPLY_MAX_PARALLEL` (default 4) caps the
total. Each batch also has its own limit, which is the concurrency chosen when someone pressed
Run. If that was left on *auto*, the worker picks a number based on free memory, half the
CPUs, and the largest peak memory any run has used so far.

Each run goes through five named steps, and if a run fails, the run records which step it
failed in:

1. **materialize** rebuilds the macro input into `runs/<run_id>/case/`.
2. **manifest** records everything needed to recreate the run (see below).
3. **solve** shrinks the case to the requested `periods` and `subperiods` if any were given,
   then runs Julia and MacroEnergy.jl's `run_case`. The log goes to `runs/<run_id>/solver.log`.
4. **load_results** loads every result CSV into DuckDB and lists them in `run_result`.
5. **finish** records the objective, termination status, wall time and peak memory, and stores
   a gzipped copy of the log in `run_log`.

If the worker dies while a run is in progress, that run is marked `failed` with the error
`interrupted` the next time the worker starts.

## The manifest

The manifest is stored on the run, and `recreate` writes it out as `manifest.json`. It
records:

- `base_content_hash`, the base case that was used;
- `macro_input_fingerprint` and `folder_hash`, which identify the exact input and let it be
  checked;
- `code_commit` and `code_dirty`, the code that ran and whether it had uncommitted changes;
- `julia_version` and `macroenergy_version`, the model versions;
- `solver`, which is `HiGHS`;
- `options`, such as `periods` and `subperiods`;
- `alembic_head` and `app_version`, the database schema and app version.

A dirty working tree is recorded rather than refused. That is fine while you are trying
things out, but a run with `code_dirty: true` cannot be reproduced from git alone, so it should
not go to [PI sign-off](../../pi-signoff.md).

## Results

`results/store.py` defines the `ResultStore` interface, and `results/duckdb_store.py` is its
only implementation. Each kind of result gets its own table (`r_capacity`, `r_costs` and so
on), with `run_id` and `period` columns added, and a separate `run_context` table describes
each run so you can join on it.

DuckDB only allows one process to open a file at a time, and the API holds `results.duckdb`
while it is running. Anything else that needs the results, such as `run --wait` or a
notebook, either has to wait until the API is stopped or go through
`GET /api/results/<name>`. If you need a new kind of query, add it to the store or as an API
endpoint, rather than opening DuckDB directly from a router.

## Julia and MacroEnergy.jl

The worker solves a case by running `$SUPPLY_JULIA -e 'using MacroEnergy; run_case(ARGS[1])' <case>`,
with `JULIA_PROJECT` set to `SUPPLY_JULIA_ENV`. You install these once per machine by
following `docs/how_to_run/run-macro-with-highs.md` in the repository. If they end up
somewhere other than `../.deps/`, set `SUPPLY_JULIA` and `SUPPLY_JULIA_ENV` to point at them.

The tests swap in a fake solver (`runs/solver.py` accepts any `Solver`), so `make test` does
not need Julia.
