# Provenance and versioning

One question: **"where did this number come from?"** If you cannot answer from the repo
and run artefacts alone — without asking a person — it is not reproducible.

---

## The chain

```text
  published figure
      ▲  chart script (committed)
  model outputs ── run manifest (run ID, status, tags)
      ▲  solver
  exact run config (generated) ── applied transformations
      ▲  expansion
  experiment YAML (committed) ── sweeps, validators
      ▲  selection
  base case snapshot ── version + lock hash
      ▲
  raw source data (surveys, statistics, partner files)
```

A break anywhere means the number at the top is an opinion.

---

## What each link records

| Link | Must record |
|---|---|
| **Base case** | Version identifier, lock hash, provenance of the raw data (who, when, licence) |
| **Experiment YAML** | Intent — readable as a diff, without running it |
| **Run config** | Generated, never hand-edited, absolute values only |
| **Manifest** | Run ID, base version, exact transformations, tags, status |

!!! danger "Two rules that carry most of the weight"
    - **Never hand-edit a file under `runs/`.** Fix the generator or the YAML.
    - **Never delete a manifest** to reclaim disk. Delete outputs instead ([FAIR A2](fair.md)).

### Base-case tracking (supply-side)

```bash
make tracking-check    # staged case still matches its lock?
make tracking-report   # what changed — JSON + Markdown
make tracking-lock     # deliberately re-approve (maintainer only)
```

`tracking-lock` is a decision, not a fix. If `tracking-check` fails, run `tracking-report`
first.

---

## Versioning the pipeline

| Bump | Means |
|---|---|
| **Patch** | Bug fix, no published number changes |
| **Minor** | New capability, existing experiments still reproduce byte for byte |
| **Major** | Outputs change — needs written justification + [PI sign-off](../pi-signoff.md) |

**The deciding test:** if a change alters the golden-output manifests or the equivalence
check, it is **major**, however small the diff looks.

---

## Identifiers

| Object | Identifier | Persistent? |
|---|---|---|
| Code release | git tag + SHA | While the repo exists |
| Base case | version + lock hash | Yes |
| Run | run ID in manifest | Yes |
| Published dataset | **DOI** (Zenodo / institutional) | Yes, independent of GitHub |

!!! tip
    A commit SHA is not citable by an outside reviewer. Mint a DOI from the GitHub
    release — on the [PI checklist](../pi-signoff.md).
