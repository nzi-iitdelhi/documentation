# NZI Modelling Handbook

This handbook explains how the Net Zero India modelling work is organised, how to run it,
and how to change it safely. It is written by and for the team at IIT Delhi and Manana
Labs.

The aim is simple: a new lab member should be able to get productive by reading these
pages, without a senior colleague having to sit with them for a day. If you find yourself
needing that conversation anyway, something is missing here, and we would like you to add
it. Every page has a pencil icon at the top right that opens it for editing on GitHub
(see [Edit this handbook](reference/contributing.md)).

## The two pipelines

The model is built in two halves that live in separate repositories.

The **[supply side](supply-side/index.md)** asks what the cheapest power system looks like
under different assumptions, for example if coal plants become 5% more expensive to build.
It keeps scenarios and their versions in a small database, turns each one into an input for
the MACRO energy model ([MacroEnergy.jl](https://github.com/macroenergy/MacroEnergy.jl)),
solves it, and stores the results. It is written in Python (FastAPI) with a Next.js web
interface, and calls Julia to solve.

The **[demand side](demand-side/index.md)** works out how much energy is needed in the first
place: how many households own an air conditioner, how far people travel, and so on. It
builds that up from surveys and trends in Python and DuckDB (with some R for the
residential sector), and produces demand files in the format the
[RUMI](https://github.com/prayas-energy/Rumi) framework uses.

Demand results feed into the supply model as part of its base case, and both end up as
MACRO outputs that we compare and chart.

```text
  demand-side ──┐
                ├──▶ MACRO ──▶ outputs + manifest ──▶ comparison / charts
  supply-side ──┘
```

## Where to start

If you are new, begin with [Start here](getting-started.md). It takes about half a day
and ends with you reproducing a known result on your own machine.

After that, go to the page that matches what you are doing:

- To run a scenario, read Run an experiment for the
  [supply side](supply-side/run-an-experiment.md) or the
  [demand side](demand-side/run-an-experiment.md).
- To change the code, read the Developer pages:
  [supply side](supply-side/developer/index.md), [demand side](demand-side/developer.md).
- To review, merge or release, read the Maintainer pages:
  [supply side](supply-side/maintainer.md), [demand side](demand-side/maintainer.md).
- To decide whether a result can be published, read [PI sign-off](pi-signoff.md).

## Who does what

We talk about three roles. They describe what you are doing at the moment, not your job
title, and most people take on more than one in a given week.

A **developer** makes a change and is responsible for showing it is safe. A **maintainer**
looks after a repository: reviewing and merging changes, cutting releases, and keeping
`main` in a state anyone can trust. The **PI** decides what leaves the team, whether that
is a figure, a dataset or a paper. [Roles at a glance](roles.md) goes into more detail.

!!! warning "This handbook is unfinished"
    It was put together from the repositories, meeting notes and the team's FAIR work, and
    it will drift out of date unless people keep fixing it. If something here is wrong,
    please correct the page or
    [open an issue](https://github.com/nzi-iitdelhi/documentation/issues).
