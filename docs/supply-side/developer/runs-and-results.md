# Runs and results

From a version or sensitivity to rows in DuckDB. This is the ring 3–4 code — read the
[blast radius](index.md#blast-radius) first.

---

## Generation

```text
  version / sensitivity
      ▼  resolve        exploration or sensitivity params → cell changes (path, row id, field, old, new)
      ▼  check          every combination valid before anything is stored
      ▼  fingerprint    sha256(base content hash + final cell values)
  macro_input           reused if the fingerprint exists; else stored with its cell_change rows
                        and the hash of the folder it builds
```

| Code | Does |
|---|---|
| `macro_inputs/resolve.py` | Params → cell changes; sensitivity combinations; fingerprint |
| `macro_inputs/apply.py` | Writes cell changes into the base files' text |
| `macro_inputs/services.py` | `generate_version`, `generate_sensitivity`, `rebuild` |
| `library/arithmetic.py` | `scale`, `add`, `set` |

Identical inputs are stored once: two versions that end up changing the same cells to the
same values share one macro input (`macro_input_source` records who asked for it).

`rebuild` recreates the folder from the stored base files + cell changes, and **refuses**
if the result does not match the recorded folder hash (it writes an `integrity_failed`
event). That check is what makes a past run trustworthy — do not weaken it.

---

## The worker

One worker per deployment, started inside the API process (`make api`) or alone
(`make worker`). It claims queued runs, oldest batch first.

| Limit | Set by |
|---|---|
| Runs at once, overall | `SUPPLY_MAX_PARALLEL` (default 4) |
| Runs at once, per batch | The concurrency chosen when pressing Run; *auto* uses free memory, half the CPUs and the largest run's peak memory so far |

Each run goes through named steps; a failure records which one:

| Step | Does |
|---|---|
| `materialize` | `rebuild` the macro input into `runs/<run_id>/case/` |
| `manifest` | Record everything needed to recreate the run (below) |
| `solve` | Shrink to `periods`/`subperiods` if asked, then Julia + MacroEnergy.jl `run_case`; log to `runs/<run_id>/solver.log` |
| `load_results` | Every result CSV into DuckDB; catalogue rows in `run_result` |
| `finish` | Objective, termination status, wall time, peak memory; log stored gzipped in `run_log` |

A run left `running` by a dead worker becomes `failed` with error `interrupted` when the
worker next starts.

---

## The manifest

Stored on the run, written to `manifest.json` by `recreate`:

| Field | Why |
|---|---|
| `base_content_hash` | Which base case |
| `macro_input_fingerprint`, `folder_hash` | Exactly which input, and its integrity check |
| `code_commit`, `code_dirty` | Which code — a dirty tree is recorded, not refused |
| `julia_version`, `macroenergy_version` | Which model |
| `solver` | `HiGHS` |
| `options` | `periods`, `subperiods`, and the like |
| `alembic_head`, `app_version` | Which database schema and app |

A run with `code_dirty: true` cannot be reproduced from git alone. Fine for a try-out; not
for anything going to [PI sign-off](../../pi-signoff.md).

---

## Results

`results/store.py` defines `ResultStore`; `results/duckdb_store.py` is the only
implementation. One table per result name (`r_capacity`, `r_costs` …), keyed by `run_id`
and `period`, plus a `run_context` table to join on.

- DuckDB allows **one process per file**: the API owns `results.duckdb` while it runs.
  Anything else (`run --wait`, a notebook) needs the API stopped, or goes through
  `GET /api/results/<name>`.
- Add a new result query to the store or an API endpoint, not as raw DuckDB access in a
  router.

---

## Julia and MacroEnergy.jl

The solver call is `$SUPPLY_JULIA -e 'using MacroEnergy; run_case(ARGS[1])' <case>`, with
`JULIA_PROJECT` set to `SUPPLY_JULIA_ENV`.
Install once per machine — `docs/how_to_run/run-macro-with-highs.md` in the repo — and
point `SUPPLY_JULIA` / `SUPPLY_JULIA_ENV` at it if it is not in `../.deps/`.

Tests inject a fake solver (`runs/solver.py` takes a `Solver`), so `make test` needs no
Julia.
