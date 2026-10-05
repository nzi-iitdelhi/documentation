# Roles at a glance

We use three roles. They describe the work you are doing at the moment rather than your
position, so the same person might be a developer in the morning and a maintainer in the
afternoon.

The **maintainer** owns a repository. Their question is whether `main` is still
trustworthy, and the way they fail is by merging something that quietly breaks
reproducibility.

The **developer** owns a change. Their question is whether this change is safe, tested and
explainable. The usual failure is shipping something without working out what else it
could affect.

The **PI** owns what the outside world sees. Their question is whether we will stand behind
a result in public. The failure here is letting out a number the team cannot reproduce.

## Who decides what

Most decisions have one person who makes the call, and others who review it or need to be
told.

| Decision | Developer | Maintainer | PI |
|---|---|---|---|
| How the code solves a problem | Decides | Reviews | |
| Whether a pull request merges | Proposes | Decides | |
| Regenerating golden outputs | Proposes and justifies | Decides | Informed |
| Changing the base-case version | Proposes | Decides | Informed |
| Cutting a release | | Decides | Informed |
| Anything that changes a published number | Proposes | Reviews | Decides |
| Publishing a dataset, figure or paper | | Prepares | Decides |
| Licence and attribution | | Proposes | Decides |
| Who gets write access | | Decides | Consulted |

## The role pages

The details differ between the two pipelines, so each side has its own pages.

| | Supply side | Demand side |
|---|---|---|
| Overview | [Supply-side](supply-side/index.md) | [Demand-side](demand-side/index.md) |
| Running experiments | [Run an experiment](supply-side/run-an-experiment.md) | [Run an experiment](demand-side/run-an-experiment.md) |
| Changing code | [Developer](supply-side/developer/index.md) | [Developer](demand-side/developer.md) |
| Looking after the repo | [Maintainer](supply-side/maintainer.md) | [Maintainer](demand-side/maintainer.md) |

The PI has a single page for both sides: [PI sign-off](pi-signoff.md).

## If you only run scenarios

Running scenarios without touching code is still developer work, just a narrow part of it.
Go straight to Run an experiment for your side. You only need the Developer page once you
want something the experiment format cannot express, whether that is a YAML file or a
parameter change in the web interface.

## Onboarding someone new

Give them access to the repositories and point them at [Start here](getting-started.md).
Try not to walk them through it in person; the point is to find out whether the handbook
works on its own. Ask them to reproduce the reference case and send you the numbers, and to
make their first pull request a fix to this handbook covering whatever confused them. If
they needed a long explanation from you along the way, that is a gap in the handbook worth
closing.
