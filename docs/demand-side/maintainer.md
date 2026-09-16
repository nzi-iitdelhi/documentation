# Maintainer (demand-side)

You own the repo, CI, the central-server relationship, and the integrity of the numbers.

Success: **anyone can clone, follow [Start here](../getting-started.md), and reproduce the
reference match rates.**

---

## Reviewing a PR

Start from the [Developer checklist](developer.md#pr-checklist).

**Always**

- [ ] Blast-radius ring stated, matching the diff as *you* read it
- [ ] `bash scripts/test.sh all` passes
- [ ] `ruff check tests` passes
- [ ] No `.duckdb`, no output CSVs, nothing from the reference folders committed
- [ ] A test exists that fails if reverted

**Ring 3+ (pipeline step or engine)**

- [ ] Before/after match rates vs the professor's validators in the PR body
- [ ] Rates got **better**, or the regression is explained and accepted
- [ ] `pier_db/` changes state how they stay RUMI-identical, citing the RUMI source

!!! danger "Green tests are not evidence of correctness here"
    Golden manifests encode our current output, known errors included. The **validator
    match rate** is what you review; the test suite only catches *unintended* change.

**Ring 5 (regenerating golden manifests)** — highest-risk change in the repo, because it
makes the regression test agree with the new behaviour by construction.

- [ ] The maths change was intended and is described in plain words
- [ ] Every changed value can be explained
- [ ] Validators improved
- [ ] Known-diffs list and `CLAUDE.md` updated
- [ ] PI informed if any published number moves

---

## Merging

- **Squash merge.** One logical change, one commit.
- Branch protection on `main` and `op_dev`: CI green, review required, no direct pushes.
  `CODEOWNERS` alone does **not** block merges — the protection rule does.
- `CODEOWNERS` currently has placeholder handles (`@maintainer`,
  `@iitd-residential-team`, `@prayas-transport-team`, `@frontend-owner`). Replace with real
  usernames or review routing is decorative.

---

## CI

`.github/workflows/ci.yml`: lint + unit + integration on PRs into `main`, `op_dev`,
`storyline`; docker-build on push to `main`. In-flight runs cancel on fix-up commits.

- [ ] Green on `main`, not "green except the advisory step nobody reads"
- [ ] Advisory `ruff check nzi_pipeline pier_db` count is trending **down**. Allowed to
      fail today; not allowed to grow. Make the cleanup someone's task.
- [ ] Integration tests still run in ~1 minute. Past a few minutes, people stop running
      them locally and CI becomes the only check.

---

## Releases

Rules: [Provenance](../principles/provenance.md#versioning-the-pipeline).

- [ ] `bash scripts/test.sh all` passes on a **fresh clone**
- [ ] Validator match rates in the release notes
- [ ] Known-diffs list current
- [ ] Docker build succeeds
- [ ] Annotated semantic tag
- [ ] Outputs changed? Stated at the top of the notes, with PI sign-off
- [ ] Citable? Mint a **DOI**

!!! tip
    Your machine has R, a warm venv, a seed DB and a working Python 3.12. A new student has
    none of those, and the 3.13 trap catches almost everyone. Verify on something clean.

---

## The central server

EC2, address in the repo README, sleeps when idle, woken by the CLI.

- [ ] Reachable, and the wake path works from cold
- [ ] Pushed runs are listable and pullable by others
- [ ] The URL in the README and [Start here](../getting-started.md) is current — both
      change in the same PR

---

## Access and onboarding

- [ ] Minimum viable access; write access is not the default
- [ ] Added to the right `CODEOWNERS` team
- [ ] Pointed at [Start here](../getting-started.md), not walked through it
- [ ] Warned about Python 3.13 and the R dependency before they lose an afternoon
- [ ] First PR is against this handbook

---

## Monthly health check

- [ ] Fresh clone → [Start here](../getting-started.md) → match rates reproduce
- [ ] Validator match rates have not silently regressed
- [ ] Advisory lint count has not grown
- [ ] Central server awake and reachable
- [ ] `CODEOWNERS` has real handles
- [ ] No open PR older than two weeks
- [ ] `CLAUDE.md`, known-diffs list and this handbook agree with each other and reality
