# Backend and API

FastAPI + SQLModel + Alembic, in `supply_side/`. The full design is in the repo:
`docs/specs/2026-09-28-approach-b-rebuild.md` (layout, lock rule, API conventions) and
`docs/specs/2026-09-28-db2-entity-model.md` (every table).

---

## Layout

One package per area, each with the same five files:

| File | Holds | Rule |
|---|---|---|
| `models.py` | Tables | Inherit the base model |
| `schemas.py` | Request / response shapes | — |
| `selectors.py` | Reads | No writes |
| `services.py` | Writes | **Every business rule lives here** |
| `router.py` | HTTP endpoints | Calls one selector or one service, nothing else |

| Package | Owns |
|---|---|
| `core/` | Settings, base model, enums, DB engine, errors, events, middleware, migrations runner |
| `base/` | The imported base case: `base`, `base_file`, `base_cell` |
| `library/` | `parameter`, `sensitivity_param`, the scale/add/set arithmetic |
| `scenarios/` | Scenarios, versions, exploration params, sensitivities, trash |
| `macro_inputs/` | Resolve changes, fingerprint, apply, rebuild a folder |
| `runs/` | Batches, runs, logs, manifest, worker, solver call |
| `results/` | `ResultStore` and its DuckDB implementation |
| `yaml_io/` | YAML import and export |

---

## Rules the code keeps

- **Base model on every table:** UUID `id`, `created_at/by`, `updated_at/by`,
  `is_deleted`, `deleted_at/by`. Detail rows (`base_cell`, `cell_change` …) carry only
  `id` + `created_at`.
- **Delete is soft.** Set `is_deleted`; query live rows with `live()` / `get_live()`.
- **All enums in `core/enums.py`**, as `class X(str, Enum)`.
- **No SQL triggers or views.** Foreign keys and (partial) unique constraints only.
- **Portable SQL.** The relational store is reached only through a SQLAlchemy URL, so it
  can move off SQLite. The columnar store only through `ResultStore`.
- **Errors:** services raise `DomainError` (400), `NotFound` (404), `Conflict` (409).
  They leave the API as `{"error": "<message>"}`.
- **Every action writes an event** (`core/events.py`). The event table is append-only
  history — the audit trail.

---

## The lock rule, in code

Services enforce it; the UI only reflects it.

1. Draft versions and sensitivities are editable; each edit records an `updated` event.
2. They lock when their first run is queued; a version also on first **Mark active**.
3. A write to a locked row returns **409** with the reason and what to do instead.
4. A new version starts from a locked version or the base, never a draft.
5. **Mark active** needs a successful run. (Seeds may activate without one; the event
   records that.)

---

## API conventions

- Everything under `/api`; resources by UUID: `/api/scenarios/{id}`, `/api/versions/{id}`,
  `/api/sensitivities/{id}`, `/api/macro-inputs/{id}`, `/api/runs/{id}`.
- `DELETE` is soft and returns `{"ok": true}`.
- Every response carries `X-Request-Id`.
- OpenAPI page at <http://127.0.0.1:8002/docs> — the quickest way to see every endpoint.

---

## Migrations

Any change to a model is a migration, in the same PR.

```bash
uv run python -m supply_side makemigrations -m "what changed"   # autogenerate
# read the generated file in supply_side/migrations/versions/ — autogenerate misses things
uv run python -m supply_side migrate
```

- [ ] Renames are written by hand as renames, not drop + add (see `a7c3e91b5d20_rename_params.py`)
- [ ] Upgrade tested on a database that already has data, not only a fresh one
- [ ] A merged migration is never edited — add a new one

The deploy runs `migrate` on the server after a backup; a broken migration breaks the live
site.

---

## Settings

Everything is overridable from the environment (`core/settings.py`):

| Variable | Default |
|---|---|
| `SUPPLY_RELATIONAL_URL` | `sqlite:///supply_side/db2.sqlite` |
| `SUPPLY_COLUMNAR_URL` | `duckdb:///supply_side/results.duckdb` |
| `SUPPLY_CASE_DIR` | `data/base-case` |
| `SUPPLY_RUNS_DIR` | `runs/` |
| `SUPPLY_JULIA` / `SUPPLY_JULIA_ENV` | `../.deps/julia/bin/julia` / `../.deps/env` |
| `SUPPLY_ACTOR` | Your login name — written to `created_by` / `updated_by` |
| `SUPPLY_MAX_PARALLEL` | `4` |
