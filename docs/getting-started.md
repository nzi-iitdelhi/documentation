# Start here

Your goal for the first week is modest: reproduce one known result on your own machine.
You do not need to understand the whole system yet. Setting up and running the reference
case usually takes about half a day.

## 1. Get access

Ask your maintainer to add you to the [`nzi-iitdelhi`](https://github.com/nzi-iitdelhi)
organisation on GitHub with read access to `supply-side` and `demand-side`. Make sure your
SSH key works (`ssh -T git@github.com` should greet you by name). You will also need access
to the meeting notes and to the shared folder with the reference data, because the base
case is not stored in git.

## 2. Work out which side you are on

If your work is about power plants, capacity, costs or the least-cost generation mix, you
are on the [supply side](supply-side/index.md). If it is about appliances, vehicles,
households or how much energy people need, you are on the
[demand side](demand-side/index.md).

Before going further, spend ten minutes on the [design principles](principles/index.md).
They explain why the code refuses to do some things that look like obvious shortcuts.

## 3. Set up

=== "Supply-side"

    You need Python 3.10 or newer with [`uv`](https://docs.astral.sh/uv/), and Node 20 or
    newer for the web interface. Julia and MacroEnergy.jl are only needed if you will solve
    cases yourself; the repo's `docs/how_to_run/run-macro-with-highs.md` explains how to
    install them.

    The base case is copied from the reference data folder. The file `data/reference.txt`
    says which folder `make data` copies from.

    ```bash
    git clone git@github.com:nzi-iitdelhi/supply-side.git
    cd supply-side
    make help           # lists every target; worth reading once
    make data           # copies the base case into data/ (not tracked by git)
    make test
    make init           # builds the local databases with the base case and 4 demo scenarios
    ```

=== "Demand-side"

    Use Python 3.10, 3.11 or 3.12. Python 3.13 does not work: `rumi` pins
    `numpy==1.26.4` and `pandas==2.2.1`, which have no 3.13 wheels, so the install tries to
    compile numpy from source and fails.

    ```bash
    git clone git@github.com:nzi-iitdelhi/demand-side.git nzi
    cd nzi && git checkout main_dev
    python3.12 -m venv .venv && source .venv/bin/activate
    pip install -e ".[dev]"
    bash scripts/test.sh unit
    ```

    The residential sector also needs R with the `survey` package, because the pipeline
    calls `Rscript` for three survey-weighted logistic regressions. Transport does not need
    R.

## 4. Reproduce a known result

This is the step that matters.

=== "Supply-side"

    Start the API and the web interface together:

    ```bash
    make run-local       # UI at http://127.0.0.1:3000, API on port 8002
    ```

    Open a scenario and its active version, then click **Review** to compare it with the
    base. You should see exactly which cells of the model input it changes.

    To solve a small version of the base case (this needs Julia), stop the API first and
    run:

    ```bash
    uv run python -m supply_side run 0 --periods 2 --subperiods 2 --wait   # 0 is the Base Case scenario
    ```

    Write down the objective value it reports. That is your baseline.

=== "Demand-side"

    ```bash
    python -m nzi_pipeline seed --sector residential
    python -m nzi_pipeline run  --sector residential
    python -m pier_db run --sector D_RES
    bash scripts/test.sh integration   # checks outputs against the golden files at 1e-6
    ```

## 5. Read about the known problems

Some numbers are known to be off, and the reasons are written down. Before you report a
bug, check the supply side's `INF_FIX_CHANGELOG.md` and `docs/nomenclature.md`, and the
demand side's [known discrepancies](demand-side/index.md#known-open-discrepancies).

## 6. Make your first contribution

Your first pull request should not be code. Make it a fix to this handbook, covering
everything that was wrong or unclear while you worked through this page. The pencil icon at
the top of each page takes you straight to the editor. That pull request is the end of your
onboarding.

## What next

When you are ready to run your own scenario, read Run an experiment for the
[supply side](supply-side/run-an-experiment.md) or the
[demand side](demand-side/run-an-experiment.md). When you need to change code, read the
Developer pages for the [supply side](supply-side/developer/index.md) or the
[demand side](demand-side/developer.md). Unfamiliar words are in the
[glossary](reference/glossary.md).
