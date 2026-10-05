# Supply-side

**Repo:** [`nzi-iitdelhi/supply-side`](https://github.com/nzi-iitdelhi/supply-side)

A scenario database for the supply side. It records scenarios, their versions and
sensitivities, turns each into an exact MACRO input, runs it, and stores the results.
Answers: "if coal capex moves ±5%, what happens to the least-cost system?"

Demand is opaque here — part of the approved native MACRO case, never modified
([principle 5](../principles/index.md)).

---

## The flow

```text
  data/base-case/   native MACRO input folder (staged, git-ignored)
      │  import → base, with a content hash; files + cells stored in SQLite
      ▼
  scenario ── version v1, v2 …   exploration params (one change each)
      │   └── sensitivity        sensitivity params (lists of amounts)
      ▼  generate
  macro input   fingerprinted; one per version, one per sensitivity combination
      ▼  run    worker → Julia + MacroEnergy.jl (HiGHS)
  run + manifest + solver log ──▶ results in DuckDB (r_* tables + run_context)
```

Explorations and sensitivities are covered in [Run an experiment](run-an-experiment.md).
Vocabulary: [Glossary](../reference/glossary.md), and `docs/nomenclature.md` in the repo.

---

## Repository map

| Path | What |
|---|---|
| `supply_side/` | FastAPI + SQLModel backend and CLI (`python -m supply_side`) |
| `web/` | Next.js UI; proxies `/api/*` to the backend |
| `examples/yaml/` | A version file and a sensitivity file to import |
| `tests/` | `unittest` suite |
| `deploy/` | Server-side install script used by `make deploy` |
| `docs/` | Nomenclature, specs, plans, wireframes; `archived/` is history, not current design |
| `data/` | Staged base case. **Git-ignored**, created by `make data` |
| `runs/` | One folder per run: MACRO input + solver log. **Git-ignored** |
| `supply_side/db2.sqlite`, `results.duckdb` | The local databases. **Git-ignored**, created by `make init` |

!!! note "The two-approach layout is gone"
    `approach_a/`, `approach_b/`, `experiments/` and `make b-verify` were removed on
    2026-09-29. They live on the `main-archive` branch; `docs/archived/two-approaches.md`
    explains them. Do not port anything back without a maintainer.

---

## Commands

`make help` is authoritative.

| Command | Does |
|---|---|
| `make data` | Stage the base case into ignored `data/` |
| `make init` | Fresh databases: migrate, import the base case, seed the library and 4 demo scenarios. **Wipes local DBs.** |
| `make run-local` | API on `:8002` and UI on `:3000` together; Ctrl-C stops both |
| `make api` / `make ui` | Just one of them |
| `make worker` | Execute queued runs without the API |
| `make test` / `make lint` / `make format` | Unit tests / `black --check` / `black` |
| `make deploy` | Maintainer only — see [Maintainer](maintainer.md#deploying) |

The CLI covers what the UI does, plus admin tasks:

| `uv run python -m supply_side …` | Does |
|---|---|
| `run <scenario-number> [--wait]` | Queue the active version's run (`--wait` solves it here; API must be stopped) |
| `recreate <run_id> <dest>` | Rebuild a run's MACRO folder + `manifest.json` from the database alone |
| `yaml import <file>` / `yaml export …` | Versions and sensitivities as YAML |
| `import [--case DIR]` | Import a new base case if its content changed |
| `migrate` / `makemigrations -m "…"` | Alembic upgrade / new migration |

---

## Next

| You are | Go to |
|---|---|
| Running a scenario | [Run an experiment](run-an-experiment.md) |
| Changing backend, UI or run code | [Developer](developer/index.md) |
| Merging, deploying, guarding `main` | [Maintainer](maintainer.md) |
| Signing off results | [PI sign-off](../pi-signoff.md) |
