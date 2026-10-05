# Tests

```bash
make test     # about 180 tests in about 15 seconds; needs no Julia and no network
make lint     # checks formatting with black, without changing anything
make format   # reformats the code with black
```

For the web interface, run `npm run lint` in `web/`, then `./check-routes.sh` with the app
running. The [Web UI](web-ui.md#checking-a-change) page explains both.

## How the suite works

The tests use `unittest`, not pytest, and live in `tests/test_*.py`.

`tests/helpers.py` does most of the setup for you. It gives each test an in-memory SQLite
database, a `TestClient` connected to it, a tiny fake MACRO case (`make_case`), and a
`seeded_world` that already contains scenarios and runs. Please use these rather than building
fixtures by hand.

The worker is given a fake solver, so runs finish in milliseconds and produce whatever outputs
the test asks for.

Migrations have their own tests in `test_migrations.py`. A migration that renames or moves
data should get a test that upgrades a database containing old rows and checks they survived.

## What a good test checks

The usual way things go wrong here is not a crash. It is a model input that looks reasonable
but is subtly wrong. So "it ran without errors" does not count as a test. What a test should
check depends on what you changed:

- **Resolve or scope:** only the intended cells change, and by the intended amount.
- **Apply:** the rebuilt folder matches its recorded hash, and existing inputs still rebuild.
- **A service rule:** the API refuses the bad case with the right status (400, 404 or 409)
  and a useful message.
- **The lock rule:** writing to a locked version or sensitivity returns 409.
- **Soft delete:** the row disappears from normal views, can be restored from the trash, and
  is never actually removed.
- **YAML:** importing, exporting and importing again changes nothing
  (see `test_yaml_round_trip.py`).
- **A migration:** upgrading keeps existing rows correct.

Every pull request should include at least one test that would fail if the change were
reverted.
