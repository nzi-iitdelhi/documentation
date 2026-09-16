# Roles at a glance

Three roles. Hats, not job titles.

| Role | Answers | Fails by |
|---|---|---|
| **Owner / Maintainer** | "Is `main` still trustworthy?" | Merging something that breaks reproducibility |
| **Developer** | "Is this change safe, tested, explainable?" | Shipping a change whose blast radius was never measured |
| **PI** | "Will we stand behind this in public?" | Letting out a number the team cannot reproduce |

---

## Who decides what

| Decision | Developer | Maintainer | PI |
|---|---|---|---|
| Approach taken in the code | **Decides** | Reviews | — |
| Whether a PR merges | Proposes | **Decides** | — |
| Regenerating golden outputs | Proposes + justifies | **Decides** | Informed |
| Changing a base-case version | Proposes | **Decides** | Informed |
| Cutting a release | — | **Decides** | Informed |
| Anything that changes a published number | Proposes | Reviews | **Decides** |
| Publishing a dataset, figure, paper | — | Prepares | **Decides** |
| Licence and attribution | — | Proposes | **Decides** |
| Repository write access | — | **Decides** | Consulted |

---

## Role pages

Specifics differ per pipeline, so role pages are per-side.

| | Supply-side | Demand-side |
|---|---|---|
| Overview | [→](supply-side/index.md) | [→](demand-side/index.md) |
| Run an experiment | [→](supply-side/run-an-experiment.md) | [→](demand-side/run-an-experiment.md) |
| Developer | [→](supply-side/developer.md) | [→](demand-side/developer.md) |
| Maintainer | [→](supply-side/maintainer.md) | [→](demand-side/maintainer.md) |
| PI | [PI sign-off](pi-signoff.md) — one page, both sides | |

---

## "I only run scenarios, I don't change code"

You are a Developer wearing a narrow hat. Go straight to **Run an experiment** for your
side. Come back to the Developer page the first time you need something a YAML cannot
express.

---

## Onboarding someone

- [ ] Grant repo access, add to the right `CODEOWNERS` team
- [ ] Point at [Start here](getting-started.md) — do not walk them through it
- [ ] Ask them to reproduce the reference case and report the numbers
- [ ] Their first PR is against **this handbook**, fixing whatever confused them

> If a KT session was needed, the handbook has a gap.
