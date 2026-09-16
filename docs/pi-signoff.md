# PI sign-off

Checklist for **anything that leaves the team**: paper, slide deck, public dataset, press
figure, partner deliverable, public repo, release. One checklist, both sides.

---

## The question

> **If someone outside this project asked me to justify this number, could I — and could
> they reproduce it without our help?**

If no, it does not go out yet.

---

## What the requester brings

If these are missing, the review has not begun.

- [ ] One paragraph, plain language: **what is being claimed**
- [ ] **Run IDs** and **base-case / seed version** behind every number
- [ ] **Commit SHA or tag** of the code that produced them
- [ ] The committed **experiment YAML** or **`run.yaml`**
- [ ] Evidence the reference case reproduced in the same environment

!!! danger "A number without a run ID is not a result"
    Send it back. Reconstructing provenance after the fact is how corrections happen.

---

## 1. Reproducibility

- [ ] Every number traces along the [provenance chain](principles/provenance.md#the-chain)
- [ ] Code is at a **tagged release**, not a commit on someone's branch
- [ ] An outsider could rerun from the public repo and published data
- [ ] Nothing depends on a file that exists only on one machine

## 2. Correctness

- [ ] Base case is the current approved version, and its version is stated
- [ ] Validators passed; match rates recorded
- [ ] **Known discrepancies affecting this claim are disclosed** — for demand work that
      means cooling, cook fuel mix, emissions
      ([list](demand-side/index.md#known-open-discrepancies)). A cooling result published
      without that caveat is a misstatement.
- [ ] Sensitivity ranges plausible; direction of effect explained
- [ ] Someone other than the author has checked the numbers

## 3. Scope of the claim

- [ ] Claim matches what was modelled — a single-region result is not a national result
- [ ] Uncertainty stated, not implied
- [ ] Assumptions driving the headline are in the text, not an appendix
- [ ] Not presented as a prediction of what *will* happen

## 4. FAIR and licensing

- [ ] Every published dataset carries an **explicit licence** ([R1.1](principles/fair.md))
- [ ] Every published code artefact carries an explicit licence
- [ ] Input-data licences permit this publication (check partner and survey data)
- [ ] Unpublishable data still has **published metadata and access rules**
- [ ] A **DOI** exists for anything citable — a GitHub URL is not a persistent identifier
- [ ] Metadata is rich enough to judge relevance without downloading

## 5. Attribution

- [ ] Every contributor credited, including data suppliers
- [ ] Upstream software cited: MACRO / MacroEnergy.jl, RUMI, other dependencies
- [ ] Partner institutions named as agreed
- [ ] Funder acknowledgements correct
- [ ] Nobody credited who did not consent to the claim

## 6. Communication

- [ ] Plain-language summary accurate, not just favourable
- [ ] Charts have units, axes, stated base case and scenario
- [ ] A figures-only reader would not draw a conclusion the text does not support
- [ ] Caveats sit with the headline, not only in methods

---

## The decision

| Outcome | Meaning |
|---|---|
| **Approved** | Goes out as is |
| **Approved with caveats** | Goes out once the named caveats are in the text — named **in writing** |
| **Hold** | Something in the chain is missing. Say which item. |
| **Rejected** | The claim is not supported by what was run |

- [ ] Decision recorded somewhere durable — issue, signed doc, dated note — with run IDs
      and release tag

!!! note "Record the approval, not just the result"
    Six months later the useful artefact is: "on this date, these run IDs at this tag were
    approved for this claim, with these caveats."

---

## When something goes out wrong

1. Establish what was actually published, from the recorded run IDs and tag
2. Determine whether the error is in the data, the model, or the claim
3. Correct publicly, with the same reach as the original
4. **Add a check** — a validator, a test, or a line on this page — so it cannot recur

Step 4 is the one that gets skipped, and the only one that compounds.
