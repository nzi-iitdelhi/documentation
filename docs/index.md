# NZI Modelling Handbook

Living handbook for the NZI energy-system modelling pipelines (IIT Delhi + Manana Labs).

It exists so a new lab member can get productive **without a senior member giving a KT**.

!!! tip "Edit me"
    Every page has a :material-pencil: icon, top right. Edit in the browser, open a PR.
    See [Edit this handbook](reference/contributing.md).

---

## The two pipelines

| | [Supply-side](supply-side/index.md) | [Demand-side](demand-side/index.md) |
|---|---|---|
| Repo | [`supply-side`](https://github.com/nzi-iitdelhi/supply-side) | [`demand-side`](https://github.com/nzi-iitdelhi/demand-side) |
| Answers | "What if coal capex moves ±5%?" | "How much energy does the residential sector need?" |
| Core artefact | Experiment YAML → one MACRO case per sensitivity | `run.yaml` → sector pipeline → PIER/RUMI CSVs |
| Stack | Python + Julia (MacroEnergy.jl) | Python + DuckDB (+ R for residential) |

```text
  demand-side ──┐
                ├──▶ MACRO ──▶ outputs + manifest ──▶ comparison / charts
  supply-side ──┘
```

---

## Pick your path

| You are | Go to |
|---|---|
| New here | [Start here](getting-started.md) |
| Running a scenario | [Supply](supply-side/run-an-experiment.md) · [Demand](demand-side/run-an-experiment.md) |
| Changing code | [Supply dev](supply-side/developer.md) · [Demand dev](demand-side/developer.md) |
| Merging / releasing | [Supply maint](supply-side/maintainer.md) · [Demand maint](demand-side/maintainer.md) |
| Approving what goes public | [PI sign-off](pi-signoff.md) |

---

## The three roles

Hats, not job titles. One person often wears two in a week.

| Role | Owns |
|---|---|
| **Owner / Maintainer** | The repo: merges, releases, integrity of `main` |
| **Developer** | A change: blast radius, tests, config, the PR |
| **PI** | What the outside world sees: numbers, claims, licences |

[Roles at a glance →](roles.md)

---

!!! warning "This handbook is incomplete on purpose"
    Seeded from the repos, meeting notes and the FAIR work. It drifts unless the team
    keeps it honest. Found something wrong? Fix the page, or
    [open an issue](https://github.com/nzi-iitdelhi/documentation/issues).
