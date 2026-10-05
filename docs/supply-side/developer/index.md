# Developer (supply-side)

These pages are for anyone changing the supply-side code: the backend, the web interface,
how model inputs are generated, or how runs are executed. Your job as a developer is to make
the change safely, and to be able to explain what it might have broken.

The details are split across four pages:

- [Backend and API](backend.md) covers models, services, the API and migrations.
- [Web UI](web-ui.md) covers the pages, the components and `web/lib/api.ts`.
- [Runs and results](runs-and-results.md) covers generating inputs, the worker, Julia and
  DuckDB.
- [Tests](tests.md) covers what to test and how.

## Blast radius

Before you start, work out how far your change could reach. That decides how much you need to
prove before it merges. We sort changes into five rings.

| Ring | What you are touching | Can it change a published number? | What you need to show |
|---|---|---|---|
| 1. Cosmetic | Docs, comments, formatting, styling | No | `make lint` passes |
| 2. Local | One page, one selector, one service rule, a test | No | `make test` passes |
| 3. Generation | `macro_inputs/` (resolve and apply), `library/arithmetic.py`, `runs/organize_case.py` | Yes | The full suite, plus a before-and-after comparison of generated inputs |
| 4. Contract | Table schema, YAML format, API shape, the manifest, the MACRO interface | Yes, everywhere | Ring 3, plus a migration, maintainer review, and a written note in the pull request |
| 5. Data | Base-case content, seeds, the parameter library | Yes, without anyone noticing | Ring 4, plus the list of changed files in the pull request, and the PI told |

Changes in ring 3 or above deserve extra care because they can affect results that already
exist. When a past run is recreated, the tool takes the stored base files, replays the stored
cell changes through `apply.py`, and checks the resulting folder against the hash recorded
at the time. That leads to two different risks:

- If you change how changes are **resolved**, an unchanged version will now produce different
  cell changes, a new fingerprint, and a different model input.
- If you change how changes are **applied**, every existing input will fail its integrity
  check. `recreate` stops working for every past run, and new runs of those inputs fail at
  the materialize step.

So a ring 3 pull request should answer two questions explicitly. Does an unchanged version
still reuse its existing macro input? And do existing inputs still rebuild? Reviewers will
assume the answer is yes unless you say otherwise.

Here is a quick way to check both:

```bash
git checkout main && make init                    # careful: this wipes your local databases
uv run python -m supply_side generate <n>         # note the macro_input_id it prints
git checkout your-branch
uv run python -m supply_side generate <n>         # "reused": True means resolve is unchanged
uv run python -m supply_side rebuild <macro_input_id> /tmp/check   # an error means apply changed
```

Try it with one version that has no exploration params and one that has several.

## Things to avoid

Some shortcuts look harmless but break the guarantees the rest of the system relies on:

- Editing a locked version or sensitivity directly in the database. Create a new version
  instead.
- Deleting rows for real. Every delete is a soft delete that sets `is_deleted`.
- Editing files under `data/` or `runs/` to get a number to come out right.
- Committing `data/`, `runs/`, `db2.sqlite` or `results.duckdb`.
- Editing a migration that has already been merged. Write a new one.
- Putting scenario concepts into model input files ([principle 5](../../principles/index.md)).
- Adding something the web interface can do that the API or command line cannot
  ([principle 8](../../principles/index.md)).

## PR checklist

Before asking for review, check that:

- [ ] the pull request says which ring the change is in
- [ ] `make test` and `make lint` pass
- [ ] for ring 3 and above, the before-and-after input comparison is attached
- [ ] any schema change comes with a tested migration
- [ ] there is a test that would fail if the change were reverted
- [ ] the description says, in a sentence, what could break and how you checked it did not
- [ ] this handbook is updated if a documented workflow changed
- [ ] no generated files or local databases are committed

The [Maintainer](../maintainer.md) page describes what happens next.
