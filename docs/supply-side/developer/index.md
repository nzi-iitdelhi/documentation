# Developer (supply-side)

Changing the backend, the UI, how inputs are generated, or how runs execute.

Your job: make the change safely, and be able to say what it could have broken.

| Working on | Read |
|---|---|
| Models, services, API, migrations | [Backend and API](backend.md) |
| Pages, components, `web/lib/api.ts` | [Web UI](web-ui.md) |
| Generation, the worker, Julia, DuckDB | [Runs and results](runs-and-results.md) |
| What to test and how | [Tests](tests.md) |

---

## Blast radius

Work out your ring **before** you write the change. It determines what you must prove.

| Ring | Touching | Changes a published number? | Required |
|---|---|---|---|
| **1 Cosmetic** | Docs, comments, formatting, styling | No | `make lint` |
| **2 Local** | One page, one selector, one service rule, a test | No | `make test` |
| **3 Generation** | `macro_inputs/` (resolve, apply), `library/arithmetic.py`, `runs/organize_case.py` | **Yes** | Full suite + before/after input diff |
| **4 Contract** | Table schema, YAML format, API shape, manifest, the MACRO interface | **Yes, everywhere** | Ring 3 + a migration + maintainer review + written note in PR |
| **5 Data** | Base case content, seeds, the parameter library | **Yes, silently** | Ring 4 + changed-files list in PR + PI informed |

!!! danger "Ring 3+ changes history"
    A run is recreated by replaying its stored cell changes through `apply.py` on the
    stored base files, then checking the folder against its recorded hash. So:

    - Change **resolve** → the same version now gives different cell changes, a new
      fingerprint, a different input.
    - Change **apply** → every existing macro input fails its integrity check:
      `recreate` stops working for **every past run**, and new runs of those inputs fail
      at the materialize step.

    The PR must answer both: **does an unchanged version reuse its macro input, and do
    existing ones still rebuild?** Silence is read as "yes".

**How to measure it**

```bash
git checkout main && make init                    # wipes your local DBs
uv run python -m supply_side generate <n>         # note macro_input_id
git checkout your-branch
uv run python -m supply_side generate <n>         # "reused": True → resolve unchanged
uv run python -m supply_side rebuild <macro_input_id> /tmp/check   # error → apply changed
```

Do it for a version with no exploration params and one with several.

---

## Never OK

- Editing a locked version or sensitivity in the database to "fix" it
- A hard `DELETE` — every delete is soft (`is_deleted`)
- Editing files under `data/` or `runs/` to make a number work
- Committing `data/`, `runs/`, `db2.sqlite` or `results.duckdb`
- Editing a migration that has already been merged — write a new one
- Adding a scenario concept to a model input file ([principle 5](../../principles/index.md))
- A UI action the CLI or API cannot do ([principle 8](../../principles/index.md))

---

## PR checklist

- [ ] Blast-radius ring stated
- [ ] `make test`, `make lint` pass
- [ ] Input diff attached for ring 3+
- [ ] Migration included and tested for any schema change
- [ ] A test exists that fails if this change is reverted
- [ ] One sentence on what could break and how you checked it did not
- [ ] Handbook updated if a documented workflow changed
- [ ] No generated or local database files committed

→ [Maintainer](../maintainer.md)
