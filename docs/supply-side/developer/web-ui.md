# Web UI

Next.js (App Router) + React + shadcn components, in `web/`. It holds no state of its own:
every page fetches from the API, and `/api/*` is proxied to `SCENARIO_API`
(default `http://127.0.0.1:8002`).

!!! warning "Not the Next.js you know"
    `web/` runs Next.js 16, with breaking changes from older versions. Read the guide in
    `web/node_modules/next/dist/docs/` before writing code, and heed deprecation notices.
    `web/AGENTS.md` says the same for AI assistants.

---

## Layout

| Path | What |
|---|---|
| `app/scenarios/`, `versions/`, `sensitivities/` | One route per screen; `[id]` pages by UUID |
| `app/parameters/`, `runs/`, `trash/` | Library, global runs, deleted items |
| `components/` | Screen pieces: forms, tables, dialogs, `brand.tsx` (header, footer, logos) |
| `components/ui/` | shadcn primitives — prefer these over new ones |
| `lib/api.ts` | **The only place that calls the API.** Typed functions, one per endpoint |
| `lib/hooks.ts` | Data-loading hooks over `api.ts` |

---

## Rules

- **Follow the wireframe.** One wireframe screen is one page; do not merge layouts.
  Wireframes: `docs/wireframes/` in the repo (open in excalidraw.com).
- **Active is green.** Status colours come from the palette in `app/globals.css`, through
  `components/status-badge.tsx` — use those, do not pick new colours.
- **No business rules in the UI.** If a button should be disabled because a version is
  locked, the API already refuses it — show its `{"error"}` message, do not reimplement
  the rule.
- **Use the team's names.** Show `base_case` as *Base Case*; humanise, never reword.
- **Everything the UI does, the API does** ([principle 8](../../principles/index.md)).

---

## Runs switch

`RUNS_ENABLED` at the top of `lib/api.ts` gates every run button (run a version, run a
sensitivity, run again). While `false`, they open the *Runs are disabled* dialog instead
of queueing. Flipping it is a maintainer decision.

---

## Checking a change

```bash
make run-local                 # API + UI
cd web && npm run lint
./check-routes.sh              # headless Chrome renders every route and checks its content
```

`check-routes.sh` needs the API and UI both up and Chrome or Chromium installed. Pages
fetch in the browser, so `curl` only sees a skeleton — this is why the script exists.

- [ ] Every route still renders (`check-routes.sh`)
- [ ] Error states shown: API down, 404, 409 on a locked row
- [ ] Checked at phone width if you touched layout
