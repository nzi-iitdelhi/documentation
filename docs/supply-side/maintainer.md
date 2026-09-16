# Maintainer (supply-side)

You own the repo. One measure of success: **anyone can clone `main`, follow
[Start here](../getting-started.md), and reproduce the reference result.**

---

## Reviewing a PR

Start from the [Developer checklist](developer.md#pr-checklist). If unsatisfied, send it
back rather than doing the work yourself.

**Always**

- [ ] Blast-radius ring stated, and matches the diff as *you* read it
- [ ] `make test`, `make lint` pass
- [ ] `make b-verify` passes, or divergence explained and time-boxed
- [ ] Nothing from `data/` or `runs/` committed
- [ ] A test exists that fails if reverted

**Ring 3+ (generation)**

- [ ] Before/after config diff attached
- [ ] PR answers explicitly: do previously generated configs still regenerate byte for byte?
- [ ] If not — justified, recorded, PI informed if needed

**Ring 5 (base case)**

- [ ] `make tracking-report` output in the PR
- [ ] Version bumped, not silently overwritten
- [ ] Provenance of the new data stated: who supplied it, when, under what licence
- [ ] PI informed — results spanning this change are no longer directly comparable

---

## Merging

- **Squash merge.** One logical change, one commit, one revert.
- Branch protection on `main`: CI green, review required, no direct pushes.
- Keep `CODEOWNERS` current — a stale one is worse than none, it fakes review.

---

## Base-case management

```bash
make tracking-check    # matches the lock?
make tracking-report   # what changed
make tracking-lock     # re-approve — MAINTAINER ONLY
```

Before `tracking-lock`:

- [ ] You read the diff and can explain every change in it
- [ ] Provenance recorded
- [ ] Version identifier bumped
- [ ] Team told that results spanning this change are not directly comparable

---

## Releases

Rules: [Provenance](../principles/provenance.md#versioning-the-pipeline).

- [ ] `make test`, `lint`, `b-verify`, `tracking-check` pass on `main`
- [ ] Reference case reproduces end to end
- [ ] `make smoke-test` works **from a fresh clone** — this is the newcomer path and it
      breaks silently
- [ ] `INF_FIX_CHANGELOG.md` reflects the tag
- [ ] Annotated semantic tag
- [ ] Outputs changed? Say so at the top of the release notes, with PI sign-off
- [ ] Citable? Mint a **DOI** from the release

!!! tip
    Clone the tag into a fresh directory to test. Your working copy has staged data, a warm
    Julia env, and state a newcomer does not have.

---

## Access and onboarding

- [ ] Minimum access that lets them work. Write access is not the default.
- [ ] Added to the right `CODEOWNERS` team
- [ ] Pointed at [Start here](../getting-started.md) — **not** walked through it
- [ ] First PR is against this handbook

---

## Monthly health check

Also before any workshop, deadline, or new joiner.

- [ ] Fresh clone → [Start here](../getting-started.md) → reference result reproduces
- [ ] `make help` matches what the Makefile does
- [ ] CI green on `main`, not quietly failing
- [ ] No open PR older than two weeks
- [ ] `CODEOWNERS` matches the people actually here
- [ ] This handbook still describes reality
