# Tests

```bash
make test     # unittest, ~180 tests, ~15 s — no Julia, no network
make lint     # black --check --diff, CI-safe
make format   # black, rewrites files
```

UI: `cd web && npm run lint`, then `./check-routes.sh` with the app up — see
[Web UI](web-ui.md#checking-a-change).

---

## How the suite is built

- **`unittest`**, not pytest. Files are `tests/test_*.py`.
- **`tests/helpers.py`** gives each test an in-memory SQLite database, a `TestClient`
  wired to it, a tiny fake MACRO case (`make_case`), and a `seeded_world` with scenarios
  and runs already in place. Use these instead of building fixtures by hand.
- **A fake solver** is injected into the worker, so runs complete in milliseconds and
  their outputs are whatever the test says.
- **Migrations** have their own tests (`test_migrations.py`): a migration that renames or
  moves data gets a test that upgrades a database holding old rows.

---

## What a good test asserts

| Change | The test must show |
|---|---|
| Resolve / scope | **Only** the intended cells change, by the intended amount |
| Apply | The rebuilt folder matches its recorded hash; an existing input still rebuilds |
| A service rule | The API refuses the broken case with the right status (400 / 404 / 409) and message |
| Lock rule | A write to a locked version or sensitivity gets **409** |
| Soft delete | The row is hidden, restorable from trash, never removed |
| YAML | Import → export → import is a no-op (`test_yaml_round_trip.py`) |
| A migration | Upgrade keeps existing rows correct |

"It ran without error" is not a test — the failure mode here is a plausible wrong input.
Every PR needs a test that fails if the change is reverted.
