# Design principles

These are the rules the code is meant to follow. They come from the
[FAIR principles](fair.md), applied to models rather than only to data, and from the
team's concept note on scalable scenario and sensitivity analysis. A pull request that
breaks one of them should be treated as a defect in review, not as a difference of taste.

**1. Keep data separate from code.** Inputs change all the time; the logic that runs them
should not. If you find yourself editing a `.py` file to change a coal cost, the cost is in
the wrong place.

**2. Base cases are versioned and never edited in place.** Each base case is a snapshot with
a content hash, and every experiment says which one it starts from. Fixing a number by
editing the staged case directly breaks every comparison made against it.

**3. Changes are written down as definitions.** An experiment is a recorded, locked version
or a YAML file, not a folder someone edited by hand. If your experiment cannot be
reproduced from what is recorded, it is not finished.

**4. Every definition expands into an exact model input, with a manifest.** The manifest
records the base version, the changes applied and the state of the code. A result with no
manifest cannot be traced.

**5. The model does not need our vocabulary.** MACRO only ever sees absolute values in its
own input format. Scenario names, sensitivities and other ideas of ours stay in our tools
and never leak into model input files.

**6. Check the workflow against a known reference before trusting it.** "The numbers look
plausible" is not evidence. The base case should reproduce its known objective value, and
the demand pipeline should match the reference outputs.

**7. Every run is independent.** A run should work the same locally, on AWS or on the HPC,
without coordinating with other runs. Two runs writing to the same path is a bug.

**8. One backend, many front ends.** The command line, the web interface and the charts all
call the same operations. If the UI can do something the CLI cannot, the logic has ended up
in the wrong layer.

**9. A stranger can reproduce the work.** Someone outside the project should be able to
repeat our results from what we publish, without asking the original developers for help.
This is the real test for the whole project.

## How they are enforced

Some of these principles are checked by the code, and some rely on review.

On the supply side, principle 2 is enforced by the content hash recorded when a base case is
imported, and principle 3 by the lock rule: a version is fixed once it has been run or made
active. Principle 4 is covered by run manifests and by the folder-hash check when a past
input is rebuilt (see [Provenance](provenance.md)). Principle 6 is a manual check on the
supply side, where someone confirms the base case reproduces its reference objective. On the
demand side it is automated, through golden-output tests at a relative tolerance of `1e-6`.

!!! note "TODO"
    Principles 1, 5, 7 and 8 have no automatic check today, and supply-side principle 6 is
    manual. It is worth deciding which of these deserve one.

## Sources

- Wilkinson, M. D. et al. *Sci. Data* 3:160018 (2016); see [FAIR for models](fair.md)
- The concept note on scalable scenario and sensitivity analysis for MACRO
- Team meetings on 24 August, 26 August and 1 September 2026
