# FAIR for models

FAIR was written for data. The authors were explicit that it goes further:

> "Importantly, it is our intent that the principles apply not only to 'data' in the
> conventional sense, but also to the algorithms, tools, and workflows that led to that
> data."
>
> — Wilkinson, M. D. et al. *Sci. Data* 3:160018 (2016)

That is why the pipeline is treated as a research object: versioned, described, licensed,
published.

---

## The principles, mapped to this project

### Findable

| | Principle | Here |
|---|---|---|
| F1 | Globally unique, persistent identifier | Run ID; base-case version; git tag. **DOI for public releases.** |
| F2 | Rich metadata | Run manifest: base version, transformations, tags, status |
| F3 | Metadata name the data's identifier | Manifest names run ID + base-case version |
| F4 | Indexed in a searchable resource | Runs pushed to the central server, listable from the CLI |

### Accessible

| | Principle | Here |
|---|---|---|
| A1 | Retrievable by identifier, standard protocol | HTTP (central server); git |
| A1.1 | Open, free, universally implementable | HTTP, git, CSV, YAML, JSON, Parquet |
| A1.2 | Supports authentication where needed | Repo access control; server auth |
| A2 | Metadata survive the data | **Never delete a manifest to reclaim disk.** |

### Interoperable

| | Principle | Here |
|---|---|---|
| I1 | Formal shared language | YAML definitions, CSV/Parquet data, JSON run configs |
| I2 | FAIR vocabularies | RUMI/PIER (demand), MACRO case schema (supply) — adopted, not reinvented |
| I3 | Qualified references to other (meta)data | Run config → base-case version; output → run |

### Reusable

| | Principle | Here |
|---|---|---|
| R1 | Rich, accurate attributes | Manifest + committed experiment YAML |
| R1.1 | Clear, accessible licence | **Every published dataset needs one** — [PI checklist](../pi-signoff.md) |
| R1.2 | Detailed provenance | [Provenance](provenance.md) |
| R1.3 | Domain community standards | RUMI/PIER; MACRO case format |

---

## Data you cannot publish

FAIR anticipates it:

> "…publication of rich metadata to facilitate discovery, including clear rules regarding
> the process for accessing the data, provides a high degree of 'FAIRness' even in the
> absence of FAIR publication of the data itself."

**Rule:** if you cannot publish the data, publish the metadata and the access rules.
"Confidential" is never a reason to omit what it is, where it came from, and how to get it.

---

## FAIR is not a technology choice

> "These high-level FAIR Guiding Principles precede implementation choices."

DuckDB, Julia, YAML, GitHub Pages are revisable. The principles should survive a rewrite.

---

## Reading

- Wilkinson et al. (2016) [doi:10.1038/sdata.2016.18](https://doi.org/10.1038/sdata.2016.18)
- Jacobsson et al., Perovskite Database, *Nat Energy* 7 (2022) [doi:10.1038/s41560-021-00941-3](https://doi.org/10.1038/s41560-021-00941-3) — our pipeline diagram is adapted from its Fig. 1
- Buster et al., *Nat Energy* 9 (2024) [doi:10.1038/s41560-024-01507-9](https://doi.org/10.1038/s41560-024-01507-9)
- [Dataverse](https://dataverse.org/) — reference FAIR repository
