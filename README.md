# NZI Modelling Handbook

Living documentation for the NZI energy-system modelling pipelines (IIT Delhi + Manana Labs).

**→ https://nzi-iitdelhi.github.io/documentation/**

Design principles, role-based workflows and checklists for the
[supply-side](https://github.com/nzi-iitdelhi/supply-side) and
[demand-side](https://github.com/nzi-iitdelhi/demand-side) pipelines. Written so a new lab
member can onboard without a knowledge-transfer session.

## Editing

Every page has a pencil icon — click it, edit the markdown, open a PR. No clone needed.

Locally:

```bash
pip install -r requirements.txt
mkdocs serve      # http://127.0.0.1:8000
```

Pages are plain markdown in `docs/`. Add new pages to the `nav:` block in `mkdocs.yml`.
Merging to `main` deploys automatically.

## Licence

MIT — see [LICENSE](LICENSE).
