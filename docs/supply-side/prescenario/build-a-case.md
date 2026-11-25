# Build a MACRO case

This page walks through the pre-scenario pages of the supply-side app: upload inputs, build
Bronze and Silver, then build and check a MACRO case. Reviewers and experimenters use it to
see what a case holds and to try a change to the inputs or the case settings.

The pages are at <https://supply-side.mananalabs.ai/prescenario>. To run them on your own
machine, start the app with `make run-local` and open <http://127.0.0.1:3000/prescenario>.

!!! warning "No login yet"
    Anyone with the URL can upload inputs and start jobs. Do not upload data that is not
    cleared for the team.

The three steps are tabs at the top of the page: **1. Inputs**, **2. Bronze + Silver** and
**3. Gold**. A tick on a tab means that step has a finished result.

## 1. Inputs

An **input set** is one complete copy of the inputs: the PIER folder, the NZI projection files
and the demand files. Set 1 is the repository's own `inputs/` folder.

The run file (`run_HiGHS.jl`, the HiGHS solver options) is not an input: every case uses the
repository's copy, and an upload that includes it is refused. To change the solver options,
change that file in the repository.

To change an input, upload it. You can choose a whole folder (a PIER folder works), single
files, or a `.zip`.

- The upload is laid over the newest input set, and the result becomes a new set. Nothing is
  replaced; every earlier set stays.
- Before sending anything, the page sends only the file names. A file the pipeline does not
  know stops the whole upload, and the page says which file. **Allowed files** lists every file
  the pipeline reads.
- PIER files that the pipeline does not read are not stored. The page lists them after the
  upload.
- Each file is checked against its schema: its columns, and the type and range of each value.
  A file that fails stops the upload, with the line and column at fault.
- A file identical to the one already in the newest set is skipped. An upload where every file
  is identical is refused, because it would make an empty set.

In the **Input sets** table, **View files** shows every file of a set and which set it came
from.

## 2. Bronze + Silver

A build reads an input set into a new Bronze build and a new Silver build. One job runs at a
time; the page shows when another is running.

You have two ways to build:

- **Full build.** Choose **Nothing: full build** and an input set. Every input of that set is
  read.
- **Apply one input.** Choose an earlier build under **Build on top of**, then the input to
  apply (**PIER** or **NZI projections**) and the set to take it from. Every other value stays
  as it was in the earlier build. Use this to see the effect of one change.

Each build stores only what changed (see [Builds keep history](index.md#builds-keep-history)).
The table lists every build with its log; a failed build shows its error there.

## 3. Gold: build a MACRO case

Pick a Silver build, review the choices, and build. The form has six numbered steps:

| Step | What you choose |
|---|---|
| 1. Silver build | The build the case is made from |
| 2. Regions | 5 regions or 25 regions. The case gets one node per region. |
| 3. Periods | Which periods are in the case. The first period is always in. A period left out gives its years to the next period kept. Each period is 12 representative days × 24 hours. |
| 4. Case settings | How MACRO solves: solution algorithm, expansion horizon, discount rate, scaling |
| 5. Policy | The unserved electricity demand price, and the CO2 cap (see below) |
| 6. Review | What the build holds: commodities, and the assets at a chosen place. **Show network** draws the network. |

**CO2 cap.** With **CO2 cap on**, each period's total CO2 emissions are limited to that
period's cap (tonnes). Emissions above the cap are allowed, but each tonne over costs the CO2
price over the cap (Rs per tonne). For example, with a cap of 100,000 tonnes, emitting
100,500 tonnes is 500 tonnes over the cap, so the case pays 500 × the price. The cap and price
are set per period in a table that appears when the cap is on. A changed value is marked.

The Review step is read only. To change an asset or a link, change the inputs (step 1) and
build again.

The bar at the end says **Default case**, or lists each change from the defaults. **Build
case** starts the job; **Reset to defaults** clears your changes.

## Cases and checks

Each case in the **Cases** table has:

| Link | What it gives |
|---|---|
| **zip** | The MACRO case folder, ready to run |
| **settings** | The settings the case was built with |
| **network** | The case's network as MACRO reads it |
| **Run check** | MACRO Diagnostics on the case |
| **log** | The job's log |

The default case is checked against the 5-region reference case: the check counts errors, and
assets and nodes missing compared with the reference. Any other case is checked for errors
only, because its differences from the reference are expected. The check's HTML reports
(**assets**, **nodes**, **topology**) open from the table.

## From the command line

The same pipeline runs without the app:

| Command | What it does |
|---|---|
| `make prescenario` | Builds Bronze, Silver and Gold from `inputs/` into `runs/coal-example/` |
| `make prescenario_test` | Runs the pre-scenario tests |
| `make macro_check` | Runs MACRO Diagnostics on the case |
| `make macro_recreate MANIFEST=path/manifest.json OUT=dir` | Makes a case again from its manifest, in a temporary git worktree, and checks every file |
| `make prescenario_diff OLD=<silver> NEW=<silver>` | Lists what changed between two Silver builds: code, inputs and Silver values |

A case built with uncommitted code (`dirty: true` in its manifest) cannot be made again.
Commit first, then build the case you keep.

## Where the files are

Everything the app makes is under `runs/prescenario-ui/` (not in git; `PRESCENARIO_UI_ROOT`
moves it):

| Path | What |
|---|---|
| `inputs/<n>/` | Input set n; `set.json` says what changed |
| `prescenario.sqlite` | Every Bronze and Silver build |
| `cases/<job>/macro/`, `macro.zip` | A MACRO case and its download |
| `cases/<job>/check.json`, `macro_check/` | The last check result and its reports |
| `jobs/<job>.json`, `jobs/<job>.log` | Each job's status and log |

## When something looks wrong

**An upload is refused.** Read the message: it names the file and, for a schema error, the
line and column. If the file is new to the pipeline, it has to be added in code first
([Add an input file](add-an-input-file.md)).

**A build fails.** Open its log from the build table. The first line of the error is also in
the table.

**A value in the case looks wrong.** Use the Review step and **Show network** to see what the
build holds. Then compare two builds with `make prescenario_diff` to see which input changed
it.

**25 regions.** Hydro potential is held per 5-region region only, so after 2041 each state's
hydro limit is a sample value (its 2041 build). The 25-region case has no reference case to be
compared with.
