# Web UI

The web interface lives in `web/`. It is a Next.js app using the App Router, React, and shadcn
components. It keeps no state of its own: every page fetches what it needs from the API, and
requests to `/api/*` are forwarded to `SCENARIO_API`, which defaults to
`http://127.0.0.1:8002`.

!!! warning "This is a newer Next.js than you may be used to"
    `web/` runs Next.js 16, which has breaking changes compared with older versions. Before
    writing code, read the relevant guide in `web/node_modules/next/dist/docs/`, and pay
    attention to deprecation notices. `web/AGENTS.md` says the same thing for AI assistants.

## How the code is laid out

| Path | What it holds |
|---|---|
| `app/scenarios/`, `app/versions/`, `app/sensitivities/` | One route per screen; the `[id]` pages take a UUID |
| `app/parameters/`, `app/runs/`, `app/trash/` | The parameter library, the list of all runs, and deleted items |
| `components/` | The pieces screens are built from: forms, tables, dialogs, and `brand.tsx` for the header, footer and logos |
| `components/ui/` | The shadcn building blocks. Use these before adding new ones. |
| `lib/api.ts` | The only place that talks to the API, with one typed function per endpoint |
| `lib/hooks.ts` | Hooks that load data through `api.ts` |

## Guidelines

**Follow the wireframes.** Each screen in the wireframes is one page, and layouts should not
be merged. The wireframes are in the repository under `docs/wireframes/`; open them at
excalidraw.com.

**Use the existing colours.** Status colours come from the palette in `app/globals.css`,
applied through `components/status-badge.tsx`. Active is always green. Please use these
rather than picking new colours.

**Leave business rules to the API.** For example, if a version is locked, the API already
refuses edits to it. The interface should show the `{"error"}` message the API returns, not
reimplement the rule.

**Keep the team's names.** Show `base_case` as *Base Case*: make names readable, but do not
reword them.

**Whatever the interface can do, the API can do too** ([principle 8](../../principles/index.md)).

## The runs switch

`RUNS_ENABLED`, at the top of `lib/api.ts`, controls every run button: running a version,
running a sensitivity, and running again. While it is `false`, those buttons open a "Runs
are disabled" dialog instead of queueing anything. Turning runs back on is a decision for a
maintainer.

## Checking a change

```bash
make run-local                 # starts the API and the web interface
cd web && npm run lint
./check-routes.sh              # opens every route in headless Chrome and checks what it shows
```

`check-routes.sh` needs both the API and the interface running, and Chrome or Chromium
installed. It exists because the pages load their data in the browser, so a plain `curl` only
ever sees an empty skeleton.

Before you open a pull request, make sure every route still renders, that error states are
handled (the API being down, a 404, and a 409 on a locked row), and, if you changed the
layout, that the page still works at phone width.
