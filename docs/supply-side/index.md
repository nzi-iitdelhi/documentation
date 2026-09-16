# Supply-side

**Repo:** [`nzi-iitdelhi/supply-side`](https://github.com/nzi-iitdelhi/supply-side)

Turns a *scenario definition* into *exact MACRO run cases*. Answers: "if coal capex moves
±5%, what happens to the least-cost system?"

Demand is opaque here — an approved native MACRO case, never modified
([principle 5](../principles/index.md)).

---

## The flow

```text
  experiments/*.yaml
      ├── Experiments   which base case, which scenario label
      ├── SweepParams   named reusable sweeps → sensitivity points
      └── Validate      general + supply-case constraints
              ▼
      runs/<label>/<sensitivity>.json   one complete, absolute run config each
              ▼
      MACRO (MacroEnergy.jl) → outputs + manifest → comparison / tornado charts
```

---

## Repository map

| Path | What |
|---|---|
| `experiments/` | Committed scenario + sensitivity YAML. Source of truth for what was run. |
| `approach_a/` | Scenario as a registered Python object; `validate_*` at generate time |
| `approach_b/` | Same pipeline, scenario model in SQLite; constraints in `schema.sql`; Next.js UI |
| `data/` | Staged base case. **Git-ignored**, created by `make data` |
| `runs/` | Generated configs. **Git-ignored, never hand-edited** |
| `tests/` | Unit tests + `test_equivalence.py` (the only file reading both approaches) |
| `base_case_tracking_example/` | Worked example of base-case locking and diff reporting |
| `docs/` | Design notes, scenario schema diagrams, two-approaches comparison |

!!! note "Two approaches, one output"
    `approach_a/` and `approach_b/` are independent implementations that generate
    identical configs byte for byte (`make b-verify`, 17 scenarios). Neither imports from
    the other, so whichever loses is one `rm -rf`.

    A **live design decision**, not permanent architecture — see `docs/two-approaches.md`.
    Do not add a third.

---

## Commands

`make help` is authoritative. The ones that matter:

| Command | Does |
|---|---|
| `make data` | Stage the base case into ignored `data/` |
| `make test` / `make lint` / `make format` | Unit tests / `black --check` / `black` |
| `make configs` | Generate one case per sensitivity |
| `make run` | Generate, then solve with the native Julia solver |
| `make sample` | Shrink one case, solve with HiGHS (no Gurobi) |
| `make check-case` | Every declared input path in the staged case resolves |
| `make tracking-check` / `-report` / `-lock` | Base-case lock verify / diff / re-approve |
| `make b-verify` | Both approaches generate identical configs |
| `make run-scenarios PARALLEL=3` | Base case + 16 scenarios, 3 at a time |

From the **parent repo root**: `make smoke-test`, `make smoke-results`, `make install-julia`.

---

## Next

| You are | Go to |
|---|---|
| Running a scenario | [Run an experiment](run-an-experiment.md) |
| Changing generator / validators / approach code | [Developer](developer.md) |
| Merging, releasing, guarding `main` | [Maintainer](maintainer.md) |
| Signing off results | [PI sign-off](../pi-signoff.md) |
