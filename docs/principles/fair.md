# FAIR for models

FAIR stands for Findable, Accessible, Interoperable and Reusable. The principles were
written for research data, but their authors said clearly that they meant them to go
further:

> "Importantly, it is our intent that the principles apply not only to 'data' in the
> conventional sense, but also to the algorithms, tools, and workflows that led to that
> data."
>
> Wilkinson, M. D. et al. *Sci. Data* 3:160018 (2016)

That is why we treat the modelling pipeline itself as a research output. It should be
versioned, described, licensed and published, just like the data it produces.

## What each principle means here

### Findable

Everything we produce should have an identifier that does not change. A run has a run ID,
a base case has its version, and code has a git tag. For anything we release publicly, we
also mint a DOI, because a GitHub link is not guaranteed to last (F1).

Each run carries a manifest describing it: which base case, which changes, what state the
code was in, and its status (F2). The manifest names the run ID and the base-case version
directly, so the two can always be matched up (F3). Demand-side runs are pushed to the
central server, where anyone on the team can list and search them from the command line
(F4).

### Accessible

Results can be fetched by their identifier over ordinary protocols: HTTP for the central
server, and git for code (A1). Those protocols, and the file formats we use (CSV, YAML,
JSON, Parquet), are open and free to implement (A1.1). Where access needs to be restricted,
the repositories and the server have their own access control (A1.2).

The metadata must outlive the data (A2). In practice this means you may delete bulky model
outputs to free disk space, but you should never delete a manifest.

### Interoperable

Definitions are written in YAML, data is CSV or Parquet, and run configuration is JSON, all
formats other people can read without our tools (I1). We use the vocabularies that already
exist in our field instead of inventing new ones: RUMI and PIER on the demand side, and the
MACRO case format on the supply side (I2). Every output points back to the run that made
it, and every run points back to its base case (I3).

### Reusable

A result is only reusable if someone can tell exactly how it was made. The manifest and the
recorded experiment definition together provide that (R1). Every dataset we publish needs a
clear licence, which the [PI checklist](../pi-signoff.md) asks for (R1.1). Provenance is
covered in detail on the [Provenance](provenance.md) page (R1.2), and we follow the
community formats mentioned above (R1.3).

## Data we cannot publish

Some of our inputs come from partners or surveys and cannot be released. FAIR allows for
this:

> "…publication of rich metadata to facilitate discovery, including clear rules regarding
> the process for accessing the data, provides a high degree of 'FAIRness' even in the
> absence of FAIR publication of the data itself."

So if you cannot publish a dataset, publish a description of it instead: what it is, where
it came from, and how someone could request access. Calling data confidential is never a
reason to leave out that description.

## FAIR is not about particular tools

> "These high-level FAIR Guiding Principles precede implementation choices."

We use DuckDB, Julia, YAML and GitHub Pages today, and any of those could change. The
principles should still hold after a rewrite.

## Further reading

- Wilkinson et al. (2016), [doi:10.1038/sdata.2016.18](https://doi.org/10.1038/sdata.2016.18)
- Jacobsson et al., Perovskite Database, *Nat Energy* 7 (2022),
  [doi:10.1038/s41560-021-00941-3](https://doi.org/10.1038/s41560-021-00941-3). Our
  pipeline diagram is adapted from its Figure 1.
- Buster et al., *Nat Energy* 9 (2024),
  [doi:10.1038/s41560-024-01507-9](https://doi.org/10.1038/s41560-024-01507-9)
- [Dataverse](https://dataverse.org/), a reference FAIR repository
