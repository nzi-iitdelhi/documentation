# PI sign-off

Anything that leaves the team goes through this page first: a paper, a slide deck, a public
dataset, a figure for the press, a deliverable for a partner, a public repository or a
release. The same checklist applies to both the supply and demand sides.

It all comes down to one question:

> If someone outside the project asked me to justify this number, could I? And could they
> reproduce it without our help?

If the answer is no, it is not ready to go out yet.

## What the person asking for sign-off should bring

The review cannot really start without these:

- [ ] a plain-language paragraph saying what is being claimed
- [ ] the run IDs, and the base-case or seed version, behind every number
- [ ] the commit SHA or release tag of the code that produced them
- [ ] the recorded experiment definition: the exported YAML on the supply side, or `run.yaml`
      on the demand side
- [ ] evidence that the reference case was reproduced in the same environment

!!! danger "A number without a run ID is not a result"
    Send it back. Reconstructing where a number came from after the fact is how
    corrections end up being needed.

## 1. Can it be reproduced?

- [ ] every number can be traced along the [provenance chain](principles/provenance.md#the-chain)
- [ ] the code is a tagged release, not a commit on someone's branch
- [ ] someone outside the team could rerun it from the public repository and published data
- [ ] nothing depends on a file that only exists on one person's machine

## 2. Is it correct?

- [ ] the base case is the current approved version, and the version is stated
- [ ] the validators passed, and the match rates are recorded
- [ ] known discrepancies that affect this claim are disclosed. For demand-side work that means
      cooling, the cooking fuel mix and emissions
      (see the [list](demand-side/index.md#known-open-discrepancies)). Publishing a cooling
      result without that caveat would be misleading.
- [ ] the sensitivity ranges are plausible, and the direction of each effect is explained
- [ ] someone other than the author has checked the numbers

## 3. Does the claim match what was modelled?

- [ ] the claim does not go beyond the model; a single-region result is not a national one
- [ ] the uncertainty is stated, not just implied
- [ ] the assumptions that drive the headline are in the main text, not hidden in an appendix
- [ ] it is not presented as a prediction of what will happen

## 4. FAIR and licensing

- [ ] every published dataset has an explicit licence ([R1.1](principles/fair.md))
- [ ] every published piece of code has an explicit licence
- [ ] the licences on our input data allow this publication; check partner and survey data in
      particular
- [ ] data we cannot publish still has published metadata and rules for requesting access
- [ ] anything people will cite has a DOI, since a GitHub link is not a persistent identifier
- [ ] the metadata is detailed enough for someone to judge whether the data is relevant
      without downloading it

## 5. Attribution

- [ ] every contributor is credited, including the people who supplied data
- [ ] the software we built on is cited: MACRO (MacroEnergy.jl), RUMI and other dependencies
- [ ] partner institutions are named as agreed
- [ ] funders are acknowledged correctly
- [ ] nobody is credited who has not agreed to the claim

## 6. Communication

- [ ] the plain-language summary is accurate, not just favourable
- [ ] charts have units and axis labels, and state the base case and the scenario
- [ ] someone who only looks at the figures would not reach a conclusion the text does not
      support
- [ ] caveats appear next to the headline, not only in the methods section

## Recording the decision

There are four possible outcomes. **Approved** means it goes out as it is. **Approved with
caveats** means it can go out once specific caveats are added to the text, and those caveats
are written down. **Hold** means something in the chain is missing; say which item. **Rejected**
means the claim is not supported by what was run.

Record the decision somewhere it will last, such as an issue, a signed document or a dated
note, together with the run IDs and the release tag.

!!! note "Record what was approved, not just the result"
    Six months later, the useful record is something like: "On this date, these run IDs at
    this tag were approved for this claim, with these caveats."

## If something goes out wrong

1. Use the recorded run IDs and tag to establish exactly what was published.
2. Work out whether the error is in the data, the model or the claim.
3. Correct it publicly, reaching the same audience as the original.
4. Add a check, whether a validator, a test or a line on this page, so the same mistake
   cannot happen again.

The last step is the one most often skipped, and it is the only one that keeps paying off.
