# Run an experiment (supply-side)

This page is for people who want to try a change to a scenario and see what it does to the
results. You do not need to know how the backend works.

There are two kinds of experiment, and the tool keeps them apart on purpose.

An **exploration** is a new version of a scenario that you are still trying out, usually to
convince the PIs that the scenario is right. It is made of *exploration params*, each of
which changes one parameter by one amount, such as "scale coal investment cost by 1.05". An
exploration produces one MACRO input.

A **sensitivity** varies some parameters around a scenario's active version to show how
robust the result is, which is what the paper needs. It is made of *sensitivity params*,
each of which gives a list of amounts, such as coal cost at 0.95, 1.00 and 1.05. A
sensitivity produces one MACRO input for every combination of those amounts.

Both kinds of param also have an *operation* (`scale`, `add` or `set`) and a *scope*, which
says which cells of the base case they apply to. If the change you have in mind cannot be
written that way, read the [Developer](developer/index.md) pages instead.

## Before you start

Make sure `make test` passes on a clean checkout and that `make run-local` is running. The
web interface is then at <http://127.0.0.1:3000> and the API on port 8002. It also helps to
know which scenario and version you are starting from.

!!! warning "Runs are switched off in the web interface for now"
    `RUNS_ENABLED` is set to `false` in `web/lib/api.ts`, so every Run button opens a
    "Runs are disabled" dialog. You can still create and edit everything. To actually
    solve something, use the command line (see step 4) or ask your maintainer.

## 1. Pick a scenario

The **Scenarios** page lists them, and **New scenario** creates one. A scenario is a
storyline, such as the reference case or a net zero pathway, and the project has four of
them. Most experiments are a new version or a sensitivity of an existing scenario rather
than a new scenario.

## 2a. Try a change as an exploration

On the scenario page, click **Add exploration**. This creates a draft version, starting
either from an existing locked version or from the base. You cannot start from another
draft.

Then click **Add exploration param** and fill in the four parts:

- **Parameter** is the knob you are turning, picked from the Parameters library, for
  example *Coal investment cost*.
- **Operation** is `scale`, `add` or `set`. Be careful here: `add 1.05` adds 1.05 to the
  value, which is rarely what you mean when you are thinking of "+5%".
- **Amount** is the number. The library shows the parameter's unit.
- **Scope** is the file pattern, ID pattern and periods that select which cells change. A
  scope that is too broad changes cells you did not intend, and the results will still look
  reasonable, so this is the part to double-check.

When you are done, click **Review →**. This compares the version with its parent cell by
cell. Check that only the cells you meant to change have changed, by the amount you
expected, and that the number of changed cells makes sense. A scope that is too broad or too
narrow usually shows up here. Finally, give the version a message that says what changed in
one sentence.

## 2b. Test robustness with a sensitivity

On the scenario page, click **New sensitivity on v*N***. The sensitivity is attached to the
active version. Add sensitivity params from the library, or click **Create sensitivity
param** if none of the existing ones fit. Give each one a name that says what it varies, such
as `standard_coal_cost`.

**Review & run →** lists every combination before anything runs. The number of combinations
grows quickly, since two params with three amounts each already give nine runs, so check
that it is what you expected.

## 3. Understand the lock rule

A draft version or sensitivity can be edited freely, and every edit is recorded. It
*locks* the first time a run is queued for it (a version also locks the first time it is
made active), and after that it never changes. If you need to change something that is
locked, create a new version from it, or click **Duplicate** on the sensitivity.

This rule is what makes results traceable: whatever ran is exactly what is on record.

## 4. Run it

When runs are enabled, click **Run** in the web interface and choose how many runs may go at
once. From the command line, with the API running, you can queue a run and let the API's
worker solve it:

```bash
uv run python -m supply_side run <scenario-number>
```

To solve directly in your terminal instead, stop the API first and add `--wait`. For a quick
check on a smaller case, also add `--periods 2 --subperiods 2`.

The `run` command always runs the scenario's **active** version. Each run is independent and
gets its own folder, `runs/<run_id>/`, with the MACRO input and the solver log. The **Runs**
page shows the newest 200 runs with their status and log, and lets you cancel or retry them.

Solving needs Julia and MacroEnergy.jl. The repository's
`docs/how_to_run/run-macro-with-highs.md` explains how to install them. If they are not in
`../.deps/`, point `SUPPLY_JULIA` and `SUPPLY_JULIA_ENV` at them.

Once a version has a successful run and the PIs agree with it, click **Mark active**. Each
scenario has at most one active version, so the previous one becomes *retired*.

## 5. Read the results

Each version page shows its runs and a chart of the results. For your own analysis, each kind
of result is a table in DuckDB that you join to `run_context`:

```sql
SELECT * FROM r_capacity JOIN run_context USING (run_id)
```

While the API is running it holds the only connection to `results.duckdb`, so use
`GET /api/results/<name>` instead. Open the DuckDB file directly only when the API is stopped.

Before you trust the numbers, check that every run you expected finished as `done`. A
`failed` run tells you something, so read its log. Objective values should be finite and
plausible; `INF_FIX_CHANGELOG.md` explains past problems with infinite values. Most
importantly, the base case should still reproduce its known reference value. If the base has
moved, every sensitivity is being measured from the wrong starting point. When you chart
results, join them to `run_context` rather than matching them up by file name.

## 6. Keep a record

The database is the record, so mostly you just need to avoid breaking the chain. Make sure
version messages and sensitivity names say what changed. When you report a result, export the
definitions behind it:

```bash
uv run python -m supply_side yaml export <scenario-number> --out defs/
```

Keep the run IDs with any figure you make, because `recreate <run_id>` can rebuild the exact
input from them later. If the result is going into a slide, a partner report or a paper, it
also needs [PI sign-off](../pi-signoff.md).

## Using YAML instead of the web interface

You can also write a version or a sensitivity as a YAML file and import it. Importing a file
that has not changed does nothing. This is a version file:

```yaml
# examples/yaml/nz_a_v1.yaml
scenario: nz_a
description: Net zero A
message: coal investment cost up 5%
exploration_params:
  - parameter: Coal investment cost
    scale: 1.05
    where: {id: "*_coal"}
```

And this is a sensitivity file. Its version (v1 here) has to be active or retired before you
import it:

```yaml
# examples/yaml/nz_a_robustness.yaml
scenario: nz_a
version: 1
name: robustness
sensitivity_params: [standard_coal_cost, standard_phwr_cost]
```

```bash
uv run python -m supply_side yaml import examples/yaml/nz_a_v1.yaml
```

## When something looks wrong

**The results are identical across a sensitivity.** Either the scope matched no cells, or the
parameter does not affect the objective. The review's cell count will tell you which.

**You cannot edit a version.** It is locked. Create a new version from it.

**You cannot start a version from a draft.** That is intended. Run the parent first so it
locks, or start from the base.

**Mark active is refused.** The version does not have a successful run yet.

**`run --wait` or DuckDB says the file is locked.** The API is running and holds
`results.duckdb`. Stop it, or run without `--wait` so the API's worker does the solve.

**The solver reports the case is infeasible.** The scope or amount produced a case that does
not make physical sense. The tool should have caught it before solving, so please report it
as a missing check.
