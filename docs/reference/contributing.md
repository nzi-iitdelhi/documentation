# Edit this handbook

This is a **living document**. If it was wrong when you needed it, fixing it is part of
the work.

---

## The fast way

1. Click the :material-pencil: pencil icon at the top right of any page
2. Edit the markdown in GitHub's browser editor
3. "Commit changes" → propose a pull request

No clone, no toolchain. This is the intended path for most edits.

---

## Locally

```bash
git clone git@github.com:nzi-iitdelhi/documentation.git
cd documentation
pip install mkdocs-material
mkdocs serve          # http://127.0.0.1:8000, live reload
```

Pages are plain markdown under `docs/`. Navigation is the `nav:` block in `mkdocs.yml` —
a **new page must be added there** or it will not appear.

Merging to `main` deploys automatically via GitHub Actions.

---

## What belongs here

| Belongs here | Belongs in the code repo |
|---|---|
| Roles, responsibilities, checklists | How a specific module works |
| Cross-repo workflows | API/function docs |
| Design principles and their rationale | Design notes for one component (`docs/` there) |
| Onboarding | Changelogs |

The test: *how the team works* → here. *How this module works* → next to the module.

---

## House style

- **Checklists over prose.** Someone is reading this mid-task.
- **Say what breaks** when a rule is ignored. A rule without a consequence gets skipped.
- **Link, do not duplicate.** Duplicated facts drift apart; one of them becomes a lie.
- **Commands must be copy-pasteable** and correct today. Verify before committing.
- Mark uncertainty with a `TODO` admonition rather than guessing.

---

## Keeping it honest

- Changing a documented workflow? Update the page **in the same PR** as the code.
- Onboarding someone? Their first PR is against this handbook.
- Maintainers: reviewing this handbook is on the monthly health check.

> If a KT session was needed, the handbook has a gap. Fix the handbook rather than
> repeating the session.

---

## Reporting without fixing

Not sure what the right answer is? [Open an issue](https://github.com/nzi-iitdelhi/documentation/issues).
A reported gap is better than a silent one.
