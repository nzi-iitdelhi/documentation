# Maintainer (supply-side)

You own the repo and the deployed app. One measure of success: **anyone can clone `main`,
follow [Start here](../getting-started.md), and reproduce the reference result.**

---

## Reviewing a PR

Start from the [Developer checklist](developer/index.md#pr-checklist). If unsatisfied, send
it back rather than doing the work yourself.

**Always**

- [ ] Blast-radius ring stated, and matches the diff as *you* read it
- [ ] `make test`, `make lint` pass — **run them yourself**; there is no CI on this repo yet
- [ ] Nothing from `data/`, `runs/`, `db2.sqlite` or `results.duckdb` committed
- [ ] A test exists that fails if reverted

**Ring 3+ (generation)**

- [ ] PR answers explicitly: does an unchanged version reuse its macro input, and do
      existing ones still rebuild? ([how to check](developer/index.md#blast-radius))
- [ ] If not — justified, recorded, PI informed if needed

**Ring 4 (contract)**

- [ ] Migration included, read by you, tested on a database with data
- [ ] YAML or API change: old exported YAML still imports, or the break is stated

**Ring 5 (base case, seeds, library)**

- [ ] Changed-files list from `import` in the PR
- [ ] Provenance of the new data stated: who supplied it, when, under what licence
- [ ] PI informed — results spanning this change are no longer directly comparable

---

## Merging

- **Squash merge.** One logical change, one commit, one revert.
- Branch protection on `main`: review required, no direct pushes.
- `main-archive` holds the old two-approach code. It is history — do not merge it back.

!!! note "TODO"
    No CI and no `CODEOWNERS` on supply-side yet. A workflow running `make test` and
    `make lint` on every PR is the cheapest guard missing today.

---

## Base-case management

The base case is a native MACRO folder. `data/reference.txt` (committed) names it;
`make data` copies it into ignored `data/base-case/`.

```bash
uv run python -m supply_side import    # "unchanged", or "new base; changed files: …"
```

Import hashes the whole folder. Same hash → nothing happens. Different → a **new base**
row linked to the previous one, with the changed files recorded in its `imported` event.
Old bases and every run on them stay intact.

Before importing a new base:

- [ ] You read the changed files and can explain every change
- [ ] Provenance recorded
- [ ] Team told that results spanning this change are not directly comparable

`make init` **wipes** the local databases and re-seeds from scratch. Fine on a laptop;
never on the server.

---

## Deploying

```bash
make deploy            # to supply_4gb; HOST=other for another host
```

It rsyncs the tree (excluding `data/`, `runs/` and the live databases), builds the UI on
the server, **backs up both databases** (last 5 kept), installs Python deps, runs
migrations, restarts the containers, then `deploy-check` confirms UI and API both return
200. Live at <https://supply-side.mananalabs.ai/> and <https://supply-side-nzi.mananalabs.ai/>.

- [ ] `make test` passes on what you are deploying
- [ ] Deploy from a clean tree — the deploy prints `+dirty` if not
- [ ] Push to GitHub after a successful deploy, so the server never runs code that is not in
      the repo
- [ ] A failed `deploy-check` means the site is down: fix or roll back now

The server's first `python -m supply_side init` is done by hand, once. Never again on a
server with data.

---

## Releases

Rules: [Provenance](../principles/provenance.md#versioning-the-pipeline).

- [ ] `make test`, `make lint` pass on `main`
- [ ] Reference case reproduces end to end
- [ ] `make data && make init && make run-local` works **from a fresh clone** — this is
      the newcomer path and it breaks silently
- [ ] `__version__` in `supply_side/__init__.py` bumped (every manifest records it)
- [ ] `INF_FIX_CHANGELOG.md` reflects the tag
- [ ] Annotated semantic tag
- [ ] Outputs changed? Say so at the top of the release notes, with PI sign-off
- [ ] Citable? Mint a **DOI** from the release

!!! tip
    Clone the tag into a fresh directory to test. Your working copy has staged data, a warm
    Julia env, and databases a newcomer does not have.

---

## Access and onboarding

- [ ] Minimum access that lets them work. Write access is not the default.
- [ ] Pointed at [Start here](../getting-started.md) — **not** walked through it
- [ ] First PR is against this handbook

---

## Monthly health check

Also before any workshop, deadline, or new joiner.

- [ ] Fresh clone → [Start here](../getting-started.md) → reference result reproduces
- [ ] `make help` matches what the Makefile does
- [ ] Deployed site matches `main` on GitHub
- [ ] Server backups exist and one has been restored somewhere, once
- [ ] No open PR older than two weeks
- [ ] This handbook still describes reality
