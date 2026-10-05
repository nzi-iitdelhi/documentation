# Maintainer (demand-side)

The demand-side maintainer looks after the repository, CI, the central server, and above all
the trustworthiness of the numbers. A good way to judge whether things are healthy: can
someone clone the repository, follow [Start here](../getting-started.md), and reproduce the
reference match rates?

## Reviewing a pull request

Start from the developer's [PR checklist](developer.md#pr-checklist).

For every pull request, check that the ring it claims matches the diff as you read it, that
`bash scripts/test.sh all` and `ruff check tests` pass, and that there is a test that would
fail if the change were reverted. Nothing should be committed from the reference folders, and
no `.duckdb` files or output CSVs.

For **pipeline or engine changes** (ring 3 and above), the pull request should show the match
rates against the professor's validators before and after. They should have improved; if they
got worse, the reason should be explained and you should agree to accept it. Changes to
`pier_db/` should explain how they stay identical to RUMI, with a reference to the RUMI
source.

!!! danger "Green tests do not prove the numbers are right"
    The golden manifests record what our code produces today, known errors included. The test
    suite only catches changes nobody intended. The validator match rate is what you are
    really reviewing.

**Regenerating golden manifests** (ring 5) is the riskiest change in the repository, because it
makes the regression test agree with the new behaviour by construction. Only accept it when the
change to the maths was intended and is described in plain words, every changed value can be
explained, the validators improved, the known-discrepancies list and `CLAUDE.md` have been
updated, and the PI has been told if any published number moves.

## Merging

We squash-merge, so each logical change is one commit. `main` and `op_dev` are protected:
CI has to pass, a review is required, and nobody can push directly. Note that `CODEOWNERS` on
its own does not block a merge; the branch protection rule does.

`CODEOWNERS` still contains placeholder handles (`@maintainer`, `@iitd-residential-team`,
`@prayas-transport-team`, `@frontend-owner`). Until they are replaced with real usernames,
review requests do not reach anyone.

## CI

`.github/workflows/ci.yml` runs lint, unit tests and integration tests on pull requests into
`main`, `op_dev` and `storyline`, and a Docker build on every push to `main`. A run in progress
is cancelled when a new commit is pushed to the same pull request.

Keep an eye on three things. CI should be genuinely green on `main`, including the advisory
step. The advisory warning count from `ruff check nzi_pipeline pier_db` should be going down;
it is allowed to fail today, but it should not grow, and someone should own cleaning it up.
And the integration tests should keep running in about a minute. Once they take several
minutes, people stop running them locally and CI becomes the only check.

## Releases

The versioning rules are on the [Provenance](../principles/provenance.md#versioning-the-pipeline)
page. Before tagging a release:

- [ ] `bash scripts/test.sh all` passes on a fresh clone
- [ ] the validator match rates are in the release notes
- [ ] the known-discrepancies list is current
- [ ] the Docker build succeeds
- [ ] the tag is an annotated semantic version
- [ ] if outputs changed, the release notes say so at the top, with PI sign-off
- [ ] if the release will be cited, a DOI has been minted from it

!!! tip
    Your own machine has R, a warmed-up virtual environment, a seed database and a working
    Python 3.12. A new student has none of these, and the Python 3.13 problem catches almost
    everyone. Test releases on a clean machine.

## The central server

The central server is an EC2 machine; its address is in the repository README. It sleeps when
nobody is using it, and the command line wakes it up. Check from time to time that it can be
reached and woken from cold, that pushed runs can be listed and pulled by others, and that the
URL in the README and on [Start here](../getting-started.md) is correct. If the URL changes,
update both in the same pull request.

## Access and onboarding

Give new people the least access that lets them do their work; write access is not the
default. Add them to the right `CODEOWNERS` team, and point them at
[Start here](../getting-started.md) rather than walking them through it. Warn them about Python
3.13 and the R dependency before they lose an afternoon to either, and ask them to make their
first pull request a fix to this handbook.

## Monthly health check

Once a month, check that:

- [ ] a fresh clone, following [Start here](../getting-started.md), reproduces the match rates
- [ ] the validator match rates have not quietly got worse
- [ ] the advisory lint count has not grown
- [ ] the central server is awake and reachable
- [ ] `CODEOWNERS` has real handles
- [ ] no pull request has been open for more than two weeks
- [ ] `CLAUDE.md`, the known-discrepancies list and this handbook agree with each other and
      with reality
