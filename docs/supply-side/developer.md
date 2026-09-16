# Developer (supply-side)

Changing the generator, validators, a sweep, the tracking machinery, or the UI.

Your job: make the change safely, and be able to say what it could have broken.

---

## Blast radius

Work out your ring **before** you write the change. It determines what you must prove.

| Ring | Touching | Changes a published number? | Required |
|---|---|---|---|
| **1 Cosmetic** | Docs, comments, formatting | No | `make lint` |
| **2 Local** | One validator, one UI screen, a test | No | Unit tests |
| **3 Generator** | Config generation, sweep expansion, transforms | **Yes** | Full suite + `make b-verify` + before/after config diff |
| **4 Contract** | Run-config schema, tracking, the MACRO interface | **Yes, everywhere** | Ring 3 + maintainer review + written note in PR |
| **5 Data** | Base case content, version, lock | **Yes, silently** | Ring 4 + `tracking-report` in PR + PI informed |

!!! danger "Ring 3+ changes history"
    A generator change can alter every future run *and* make old runs irreproducible. The
    PR must answer: **do previously generated configs still regenerate byte for byte?**
    Silence is read as "yes".

**How to measure it**

```bash
git checkout main && make configs CONFIG_PERIODS=2 CONFIG_SUBPERIODS=2
cp -r runs /tmp/configs-before
git checkout your-branch && make configs CONFIG_PERIODS=2 CONFIG_SUBPERIODS=2
diff -r /tmp/configs-before runs
```

---

## The two approaches

- **Neither may import from the other** — that independence is what makes `b-verify` mean
  something
- `tests/test_equivalence.py` is the only file allowed to read both
- Change generation behaviour in one → change it in both, same PR, or state in the PR why
  the divergence is intentional and time-boxed
- **Do not add a third approach.** The plan is to delete one.

```bash
make b-verify
```

---

## Adding a sweep parameter

The most common real task. A sweep is a **registered, named object** so a typo fails loudly.

- [ ] Named like the existing ones — region, technology, quantity
      (`nr_coal_max_capacity`, `nr_onshore_wind_investment_cost`)
- [ ] Registered in **both** approaches
- [ ] Validation added — what values are physically meaningful?
- [ ] Unit test asserting **only the intended cells change**
- [ ] Generated small configs and diffed them by hand once

!!! note "Broad selectors"
    "All solar assets in every region" was discussed and is **not fully specified**. Get
    the precise use case written down first — a selector that silently matches too much is
    this pipeline's worst failure mode, because the output still looks reasonable.

---

## Validators

A validator at generate time is worth ten hours of solver time.

- [ ] Fails at **generate** time, not run time
- [ ] Message names the file, the field, and the offending value
- [ ] Test with a deliberately invalid input
- [ ] Registered in the right set (`validate: supply_case`)

If a solver run fails in a way a validator could have caught, writing that validator is
part of the fix.

---

## Tests

```bash
make test        # unit
make lint        # black --check --diff, CI-safe
make b-verify    # cross-approach equivalence, 17 scenarios
make check-case  # every declared input path resolves
```

**A good test here** asserts that generation changes *only* the selected cells. "It ran
without error" is not a test — the failure mode is producing a plausible wrong case.

---

## Never OK

- Editing a file under `runs/`
- Editing the staged base case to make a number work
- Committing anything from `data/` or `runs/`
- `make tracking-lock` to silence a failing `tracking-check`
- Adding a scenario concept to a model input file ([principle 5](../principles/index.md))

---

## PR checklist

- [ ] Blast-radius ring stated
- [ ] `make test`, `make lint` pass
- [ ] `make b-verify` passes, or divergence explained
- [ ] Config diff attached for ring 3+
- [ ] A test exists that fails if this change is reverted
- [ ] One sentence on what could break and how you checked it did not
- [ ] Handbook updated if a documented workflow changed
- [ ] No generated or staged files committed

→ [Maintainer](maintainer.md)
