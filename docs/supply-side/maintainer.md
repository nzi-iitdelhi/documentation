# Maintainer (supply-side)

The maintainer looks after the repository and the deployed app. A good way to judge whether
things are healthy: can someone clone `main`, follow [Start here](../getting-started.md), and
reproduce the reference result without help?

## Reviewing a pull request

Start from the developer's [PR checklist](developer/index.md#pr-checklist). If something on
it is missing, send the pull request back and ask for it, rather than doing the work yourself.

For every pull request, check that the ring it claims matches the diff as you read it. Run
`make test` and `make lint` yourself, because there is no CI on this repository yet. Make sure
nothing from `data/` or `runs/`, and neither database file, has been committed, and that
there is a test that would fail if the change were reverted.

For **generation changes** (ring 3 and above), the pull request should say explicitly whether
an unchanged version still reuses its macro input and whether existing inputs still rebuild.
The developer page explains [how to check](developer/index.md#blast-radius). If the answer is
no, the reason should be written down, and the PI told if published results are affected.

For **contract changes** (ring 4), read the migration yourself and make sure it was tested on
a database with real data in it. If the YAML format or the API changed, check that previously
exported YAML still imports, or that the pull request says clearly that it no longer does.

For **data changes** (ring 5: the base case, the seeds or the parameter library), the pull
request should include the list of changed files that `import` reports, and say where the new
data came from: who supplied it, when, and under what licence. Tell the PI, because results
from before and after the change can no longer be compared directly.

## Merging

We squash-merge, so each logical change is one commit that can be reverted on its own. `main`
should be protected so that changes need a review and nobody can push to it directly. The
`main-archive` branch holds the old two-approach code. It is there for reference and should
not be merged back.

!!! note "TODO"
    The supply-side repository has no CI and no `CODEOWNERS` file yet. A workflow that runs
    `make test` and `make lint` on every pull request would be the cheapest protection to add.

## Base-case management

The base case is a native MACRO input folder. The committed file `data/reference.txt` says
which folder it is, and `make data` copies it into `data/base-case/`, which git ignores. To
bring a new version of the base case into the database, run:

```bash
uv run python -m supply_side import    # prints "unchanged", or "new base; changed files: …"
```

Import hashes the whole folder. If the hash matches the latest base, nothing happens. If it is
different, the tool creates a new base linked to the previous one and records the changed
files in its `imported` event. Older bases, and every run made on them, are left untouched.

Before importing a new base, read through the changed files and make sure you can explain each
change. Record where the data came from, and let the team know that results from before and
after are not directly comparable.

Note that `make init` wipes the local databases and builds them again from scratch. That is
fine on a laptop, but never run it on the server.

## Deploying

```bash
make deploy            # deploys to supply_4gb; use HOST=other for a different host
```

The deploy copies the code to the server with rsync, leaving out `data/`, `runs/` and the
live databases. It then builds the web interface on the server, backs up both databases
(keeping the last five copies), installs the Python dependencies, runs the migrations, and
restarts the containers. Finally, `deploy-check` confirms that both the interface and the API
respond with 200. The app is live at <https://supply-side.mananalabs.ai/> and
<https://supply-side-nzi.mananalabs.ai/>.

Make sure `make test` passes on whatever you deploy, and deploy from a clean working tree; the
deploy prints `+dirty` if it is not. After a successful deploy, push to GitHub, so the server
never runs code that is not in the repository. If `deploy-check` fails, the site is down, so
fix it or roll back straight away.

The server's database was set up once by hand with `python -m supply_side init`. Never run
that again on a server that has data.

## Releases

The versioning rules are on the [Provenance](../principles/provenance.md#versioning-the-pipeline)
page. Before tagging a release:

- [ ] `make test` and `make lint` pass on `main`
- [ ] the reference case reproduces from start to finish
- [ ] `make data && make init && make run-local` works from a fresh clone
- [ ] `__version__` in `supply_side/__init__.py` has been bumped, since every manifest records it
- [ ] `INF_FIX_CHANGELOG.md` is up to date
- [ ] the tag is an annotated semantic version
- [ ] if outputs changed, the release notes say so at the top, with PI sign-off
- [ ] if the release will be cited, a DOI has been minted from it

The fresh-clone check is the path every newcomer takes, and it tends to break without anyone
noticing. Test it by cloning the tag into a new directory: your usual working copy has staged
data, a warmed-up Julia environment and databases that a newcomer will not have.

## Access and onboarding

Give new people the least access that lets them do their work; write access is not the
default. Point them at [Start here](../getting-started.md) rather than walking them through
it, and ask them to make their first pull request a fix to this handbook.

## Monthly health check

Once a month, and before any workshop, deadline or new person joining, check that:

- [ ] a fresh clone, following [Start here](../getting-started.md), reproduces the reference result
- [ ] `make help` matches what the Makefile actually does
- [ ] the deployed site matches `main` on GitHub
- [ ] the server backups exist, and one has been restored somewhere at least once
- [ ] no pull request has been open for more than two weeks
- [ ] this handbook still describes how things really work
