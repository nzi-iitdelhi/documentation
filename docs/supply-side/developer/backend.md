# Backend and API

The backend lives in `supply_side/` and is built with FastAPI, SQLModel and Alembic. The full
design is written up in the repository. `docs/specs/2026-09-28-approach-b-rebuild.md` covers
the layout, the lock rule and the API conventions, and
`docs/specs/2026-09-28-db2-entity-model.md` describes every table.

## How the code is laid out

The backend is split into packages by area. Each package has the same five files, and each
file has one job:

- `models.py` defines the tables, which all inherit the shared base model.
- `schemas.py` defines the shapes of requests and responses.
- `selectors.py` holds functions that read, and never write.
- `services.py` holds functions that write. Every business rule lives here.
- `router.py` defines the HTTP endpoints. Each endpoint calls one selector or one service and
  does nothing else.

The packages are:

| Package | What it owns |
|---|---|
| `core/` | Settings, the base model, enums, the database engine, errors, events, middleware, and the migration runner |
| `base/` | The imported base case: `base`, `base_file` and `base_cell` |
| `library/` | Parameters, sensitivity params, and the `scale`, `add` and `set` arithmetic |
| `scenarios/` | Scenarios, versions, exploration params, sensitivities, and the trash |
| `macro_inputs/` | Resolving changes, fingerprinting, applying them, and rebuilding a folder |
| `runs/` | Run batches, runs, logs, the manifest, the worker, and the call to the solver |
| `results/` | `ResultStore` and its DuckDB implementation |
| `yaml_io/` | YAML import and export |

## Conventions

Every table inherits the base model, which gives it a UUID `id`, `created_at` and
`created_by`, `updated_at` and `updated_by`, and `is_deleted`, `deleted_at` and `deleted_by`.
Detail tables such as `base_cell` and `cell_change` only carry `id` and `created_at`, and
follow their parent row.

Deleting is always a soft delete: it sets `is_deleted` and keeps the row. Use the `live()`
and `get_live()` helpers to query rows that have not been deleted.

All enums live in `core/enums.py` and are written as `class X(str, Enum)`.

We do not use SQL triggers or views, only foreign keys and unique constraints (some of them
partial). The relational store is reached only through a SQLAlchemy URL with portable SQL, so
it can move off SQLite later. The results store is reached only through `ResultStore`.

Services report problems by raising `DomainError`, `NotFound` or `Conflict`. These become HTTP
400, 404 and 409 responses with a body of `{"error": "<message>"}`.

Every action writes an event through `core/events.py`. The event table is never edited or
cleared, so it is the full audit trail.

## The lock rule in code

The services enforce the lock rule; the web interface only reflects it. Drafts of versions and
sensitivities can be edited, and each edit records an `updated` event. They lock when their
first run is queued, and a version also locks the first time it is marked active. Any write to
a locked row returns 409, with a message explaining why and what to do instead. A new version
must start from a locked version or from the base, never from a draft. Marking a version
active requires a successful run, with one exception: the seed data may activate versions
without a run, and the event records that it did.

## API conventions

All endpoints sit under `/api`, and resources are addressed by UUID, for example
`/api/scenarios/{id}`, `/api/versions/{id}`, `/api/sensitivities/{id}`,
`/api/macro-inputs/{id}` and `/api/runs/{id}`. `DELETE` is a soft delete and returns
`{"ok": true}`. Every response carries an `X-Request-Id` header.

The quickest way to see every endpoint is the OpenAPI page at
<http://127.0.0.1:8002/docs> while the API is running.

## Migrations

Any change to a model needs a migration in the same pull request:

```bash
uv run python -m supply_side makemigrations -m "what changed"   # generates a migration
uv run python -m supply_side migrate                            # applies it
```

Always read the generated file in `supply_side/migrations/versions/` before committing it,
because autogenerate misses things. A rename in particular has to be written by hand as a
rename; otherwise Alembic drops the old column and adds a new one, losing the data.
`a7c3e91b5d20_rename_params.py` is a good example. Test the upgrade on a database that already
has data in it, not only on a fresh one. Once a migration is merged, never edit it; add a new
one instead.

This matters beyond your laptop. Each deploy backs up the server's database and then runs
`migrate`, so a broken migration takes the live site down.

## Settings

Every setting can be overridden with an environment variable (see `core/settings.py`):

| Variable | Default |
|---|---|
| `SUPPLY_RELATIONAL_URL` | `sqlite:///supply_side/db2.sqlite` |
| `SUPPLY_COLUMNAR_URL` | `duckdb:///supply_side/results.duckdb` |
| `SUPPLY_CASE_DIR` | `data/base-case` |
| `SUPPLY_RUNS_DIR` | `runs/` |
| `SUPPLY_JULIA`, `SUPPLY_JULIA_ENV` | `../.deps/julia/bin/julia` and `../.deps/env` |
| `SUPPLY_ACTOR` | Your login name, which is written to `created_by` and `updated_by` |
| `SUPPLY_MAX_PARALLEL` | `4` |
