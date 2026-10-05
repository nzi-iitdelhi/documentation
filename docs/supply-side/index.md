# Supply-side

**Repository:** [`nzi-iitdelhi/supply-side`](https://github.com/nzi-iitdelhi/supply-side)

The supply side answers questions like "if coal plants cost 5% more to build, how does the
least-cost power system change?" It does this with a small scenario database. You record a
scenario and the changes you want to try, the tool turns each one into an exact input for
the MACRO model, solves it, and stores the results so they can be compared.

Demand is treated as fixed here. It arrives as part of the approved base case, and nothing
on the supply side modifies it ([principle 5](../principles/index.md)).

## How it fits together

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

Everything starts from the **base case**, a complete MACRO input folder. When it is
imported, the tool stores every file and records a hash of the whole folder, so we always
know exactly which base a result came from.

On top of the base sit **scenarios**, such as the reference case or a net zero pathway.
Each scenario has numbered **versions**. A new version is called an *exploration* while it
is being tried out, and becomes the *active* version once the team agrees with it. A
**sensitivity** varies a few parameters around the active version, for example coal cost at
−5%, 0% and +5%, to show how robust the result is.

When you run a version or a sensitivity, the tool works out exactly which cells of the base
case change, builds the MACRO input, and hands it to a background worker that solves it with
Julia. Results are loaded into DuckDB, one table per kind of result. The
[Nomenclature](nomenclature.md) page defines all of these terms precisely, and
[Run an experiment](run-an-experiment.md) walks through the workflow.

## What is in the repository

| Path | What it holds |
|---|---|
| `supply_side/` | The backend (FastAPI and SQLModel) and the command line, `python -m supply_side` |
| `web/` | The Next.js web interface, which forwards `/api/*` requests to the backend |
| `examples/yaml/` | One example version file and one example sensitivity file |
| `tests/` | The `unittest` test suite |
| `deploy/` | The install script `make deploy` runs on the server |
| `docs/` | Nomenclature, specs, plans and wireframes. `docs/archived/` is history, not current design. |
| `data/` | The staged base case, created by `make data`. Not tracked by git. |
| `runs/` | One folder per run, with its MACRO input and solver log. Not tracked by git. |
| `supply_side/db2.sqlite`, `results.duckdb` | The local databases, created by `make init`. Not tracked by git. |

!!! note "The old two-approach layout is gone"
    Until 29 September 2026 the repository had two parallel implementations
    (`approach_a/` and `approach_b/`), an `experiments/` folder of YAML files, and a
    `make b-verify` check comparing them. All of that was replaced by the scenario
    database. The old code is kept on the `main-archive` branch, and
    `docs/archived/two-approaches.md` explains it. Please check with a maintainer before
    bringing anything back from there.

## Everyday commands

`make help` always has the current list. These are the ones you will use most:

| Command | What it does |
|---|---|
| `make data` | Copies the base case into `data/` |
| `make init` | Builds fresh local databases: applies migrations, imports the base case, and adds the parameter library and four demo scenarios. This **wipes** any local databases you already have. |
| `make run-local` | Starts the API on port 8002 and the web interface on port 3000; Ctrl-C stops both |
| `make api`, `make ui` | Starts just one of them |
| `make worker` | Runs queued jobs without starting the API |
| `make test`, `make lint`, `make format` | Runs the tests, checks formatting with `black`, or reformats the code |
| `make deploy` | Deploys to the server; maintainers only, see [Maintainer](maintainer.md#deploying) |

The command line can do everything the web interface does, plus a few admin tasks. Run
these as `uv run python -m supply_side <command>`:

| Command | What it does |
|---|---|
| `run <scenario-number> [--wait]` | Queues a run of the scenario's active version. With `--wait` it solves it in your terminal, which needs the API stopped. |
| `recreate <run_id> <dest>` | Rebuilds a past run's MACRO folder and `manifest.json` from the database alone |
| `yaml import <file>`, `yaml export …` | Reads or writes versions and sensitivities as YAML |
| `import [--case DIR]` | Imports a new base case, if its contents have changed |
| `migrate`, `makemigrations -m "…"` | Upgrades the database schema, or creates a new migration |

## Where to go next

If you want to run a scenario, read [Run an experiment](run-an-experiment.md). If you are
changing the backend, the web interface or how runs work, read the
[Developer](developer/index.md) pages. Reviewing, merging and deploying are covered on the
[Maintainer](maintainer.md) page, and approving results for publication on
[PI sign-off](../pi-signoff.md).
