# Provenance and versioning

One question: **"where did this number come from?"** If you cannot answer from the repo
and run artefacts alone — without asking a person — it is not reproducible.

---

## The chain

```text
  published figure
      ▲  chart script (committed)
  model outputs ── run manifest (run ID, code commit, model versions, options)
      ▲  solver
  exact model input (generated) ── cell changes, fingerprint, folder hash
      ▲  generation
  locked definition ── supply: scenario version / sensitivity · demand: run.yaml
      ▲  selection
  base case snapshot ── content hash
      ▲
  raw source data (surveys, statistics, partner files)
```

A break anywhere means the number at the top is an opinion.

---

## What each link records

| Link | Must record |
|---|---|
| **Base case** | Content hash, previous version, provenance of the raw data (who, when, licence) |
| **Definition** | Intent — readable without running it (supply: version message + review; demand: `run.yaml` diff) |
| **Model input** | Generated, never hand-edited, absolute values only; rebuildable and hash-checked |
| **Manifest** | Run ID, base hash, input fingerprint, code commit (+ dirty flag), model versions, options |

!!! danger "Two rules that carry most of the weight"
    - **Never hand-edit a generated input** (supply `runs/`). Fix the generator or the definition.
    - **Never delete a manifest** to reclaim disk. Delete outputs instead ([FAIR A2](fair.md)).

### The supply-side chain, concretely

| Question | Answered by |
|---|---|
| Which base case? | `base` row: content hash, the previous base, changed files in its `imported` event |
| What was changed? | The locked version or sensitivity, and its `cell_change` rows |
| Exactly which input ran? | `recreate <run_id> <dest>` rebuilds it + `manifest.json`, refusing if the folder hash differs |
| Who did what, when? | The append-only `event` table; `created_by` / `updated_by` on every row |

A new base is a maintainer decision: see [Maintainer](../supply-side/maintainer.md#base-case-management).

---

## Versioning the pipeline

| Bump | Means |
|---|---|
| **Patch** | Bug fix, no published number changes |
| **Minor** | New capability, existing experiments still reproduce byte for byte |
| **Major** | Outputs change — needs written justification + [PI sign-off](../pi-signoff.md) |

**The deciding test:** if a change alters the golden-output manifests (demand), or makes an
unchanged supply-side version generate a different input, it is **major**, however small
the diff looks.

---

## Identifiers

| Object | Identifier | Persistent? |
|---|---|---|
| Code release | git tag + SHA | While the repo exists |
| Base case | content hash | Yes |
| Run | run ID (UUID) + manifest | Yes |
| Published dataset | **DOI** (Zenodo / institutional) | Yes, independent of GitHub |

!!! tip
    A commit SHA is not citable by an outside reviewer. Mint a DOI from the GitHub
    release — on the [PI checklist](../pi-signoff.md).
