# Provenance and versioning

Provenance answers one question: where did this number come from? You should be able to
answer it from the repositories and the run records alone, without having to ask the person
who made it. If you cannot, the result is not reproducible.

## The chain

Every published number sits at the top of a chain of steps, and each step has to be
recorded.

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

If any link is missing, the number at the top is closer to an opinion than a result.

## What each link has to record

The **base case** needs its content hash, the version it replaced, and where the raw data
came from: who supplied it, when, and under what licence.

The **definition** needs to state its intent in a way someone can read without running
anything. On the supply side that is the version's message together with its review against
the parent; on the demand side it is the diff of `run.yaml`.

The **model input** is always generated, never edited by hand, and contains absolute values
only. It must be possible to rebuild it, and the rebuild is checked against a recorded hash.

The **manifest** records the run ID, the base-case hash, the input's fingerprint, the code
commit (including whether the working tree had uncommitted changes), the model versions,
and the run options.

!!! danger "Two rules that matter most"
    Never edit a generated input by hand, such as a folder under supply-side `runs/`. Fix
    the generator or the definition instead.

    Never delete a manifest to free disk space. Delete the bulky outputs if you must, but
    keep the manifest ([FAIR A2](fair.md)).

### How this works on the supply side

Each question in the chain has a concrete place to look:

- **Which base case was used?** The `base` row records its content hash and the base it
  replaced, and its `imported` event lists the files that changed.
- **What was changed?** The locked version or sensitivity, and its `cell_change` rows.
- **Which input exactly ran?** `python -m supply_side recreate <run_id> <dest>` rebuilds the
  input folder and its `manifest.json`, and refuses if the folder hash does not match.
- **Who did what, and when?** The `event` table is an append-only history, and every row
  also has `created_by` and `updated_by`.

Importing a new base case is a maintainer decision; see
[Maintainer](../supply-side/maintainer.md#base-case-management).

## Versioning the pipeline

We use semantic versioning, and the version number says something about results.

A **patch** release fixes a bug without changing any published number. A **minor** release
adds something new while every existing experiment still reproduces exactly. A **major**
release changes outputs, so it needs a written justification and
[PI sign-off](../pi-signoff.md).

To decide which one applies, ask whether the change alters the demand side's golden-output
manifests, or makes an unchanged supply-side version produce a different model input. If it
does either, it is a major change, however small the diff looks.

## Identifiers

A code release is identified by its git tag and commit SHA, which last as long as the
repository does. A base case is identified by its content hash and a run by its run ID (a
UUID) together with its manifest; both are permanent. A published dataset gets a DOI from
Zenodo or an institutional repository, which stays valid even if GitHub does not.

!!! tip
    A commit SHA is not something an outside reviewer can cite. Mint a DOI from the GitHub
    release; it is on the [PI checklist](../pi-signoff.md).
