# Design principles

The rules the codebase is meant to obey. A PR that violates one is a reviewable defect,
not a matter of taste.

Sources: the [FAIR principles](fair.md) applied to models, and the team's concept note on
scalable scenario analysis.

---

| # | Principle | You are violating it when |
|---|---|---|
| 1 | **Separate data from code.** Inputs change; the logic that runs them does not. | You edit a `.py` file to change a coal cost. |
| 2 | **Base cases are versioned and immutable.** Snapshot + lock per experiment. | You fix a number by editing the staged case in place. |
| 3 | **Changes are version-controlled definitions.** A committed YAML, not an edited folder. | Your experiment cannot be reproduced from the repo alone. |
| 4 | **Expand into exact run inputs with a manifest.** Base version, changes, tags, status. | A result exists with no manifest. |
| 5 | **The model does not need your vocabulary.** MACRO gets absolute values only. | You add a scenario concept to a model input file. |
| 6 | **Validate against a known reference** before trusting the workflow. | "It looks plausible" is your evidence. |
| 7 | **Every run is independent.** Local, AWS, or HPC, no coordination. | Two runs share a writable path. |
| 8 | **One backend, many front ends.** CLI, UI and charts call the same operations. | The UI can do something the CLI cannot. |
| 9 | **Reproducible by a stranger**, without support from the original developers. | — this is the acceptance test for the whole project. |

---

## Enforcement

| Principle | Enforced by |
|---|---|
| 2 | `make tracking-check` / `tracking-report` / `tracking-lock` |
| 4 | Run manifests; see [Provenance](provenance.md) |
| 6 | `make b-verify` (supply); golden-manifest tests at `1e-6` (demand) |

!!! note "TODO"
    Principles 1, 3, 5, 7, 8 have no mechanical check today. Worth deciding which deserve one.

---

## Sources

- Wilkinson, M. D. et al. *Sci. Data* 3:160018 (2016) — see [FAIR for models](fair.md)
- Concept note: scalable scenario and sensitivity analysis for MACRO
- Meeting decisions, 24 Aug / 26 Aug / 1 Sep 2026
