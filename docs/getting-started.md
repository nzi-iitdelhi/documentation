# Start here

Goal for week one: **reproduce one known result on your own machine.** Not to understand
the whole system. Budget half a day.

---

## 1. Access

- [ ] GitHub account in the [`nzi-iitdelhi`](https://github.com/nzi-iitdelhi) org
- [ ] Read access to `supply-side` and `demand-side`
- [ ] SSH key working (`ssh -T git@github.com` greets you by name)
- [ ] Access to meeting notes + reference data folder

---

## 2. Know which side you are on

| Your work is about | You are on |
|---|---|
| Capacity, capex, least-cost mix, MACRO scenarios | [Supply-side](supply-side/index.md) |
| Appliances, vehicles, households, energy *needed* | [Demand-side](demand-side/index.md) |

Then read the [design principles](principles/index.md) — 10 minutes, explains why the code
refuses some obvious-seeming things.

---

## 3. Set up

=== "Supply-side"

    Python 3.10+ with [`uv`](https://docs.astral.sh/uv/), Node 20+. Julia +
    MacroEnergy.jl only if you will solve: `docs/how_to_run/run-macro-with-highs.md`.

    The base case comes from the reference data folder (see Access); `data/reference.txt`
    names the folder `make data` copies.

    ```bash
    git clone git@github.com:nzi-iitdelhi/supply-side.git
    cd supply-side
    make help           # read the target list first
    make data           # stage the base case into ignored data/
    make test
    make init           # databases: base case, library, 4 demo scenarios
    ```

=== "Demand-side"

    Python **3.10–3.12, not 3.13** — `rumi` pins `numpy==1.26.4` / `pandas==2.2.1`, which
    have no 3.13 wheels, so install tries to compile numpy from source and fails.

    ```bash
    git clone git@github.com:nzi-iitdelhi/demand-side.git nzi
    cd nzi && git checkout main_dev
    python3.12 -m venv .venv && source .venv/bin/activate
    pip install -e ".[dev]"
    bash scripts/test.sh unit
    ```

    **Residential only:** needs R + the `survey` package (`Rscript` is called for three
    survey-weighted logistic regressions). Transport has no R dependency.

---

## 4. Reproduce a known result

This is the part that matters.

=== "Supply-side"

    ```bash
    make run-local       # UI at http://127.0.0.1:3000, API at :8002
    ```

    Open a scenario, its active version, and **Review** it against the base — you should
    see exactly which cells it changes. Then solve a small case (needs Julia), with the
    API stopped:

    ```bash
    uv run python -m supply_side run 0 --periods 2 --subperiods 2 --wait   # 0 = Base Case
    ```

    Write down the objective value. That is your baseline.

=== "Demand-side"

    ```bash
    python -m nzi_pipeline seed --sector residential
    python -m nzi_pipeline run  --sector residential
    python -m pier_db run --sector D_RES
    bash scripts/test.sh integration   # golden-output check at 1e-6
    ```

---

## 5. Read the known gaps

Before you "discover" a tracked bug:

- Supply: `INF_FIX_CHANGELOG.md`, `docs/nomenclature.md`
- Demand: [known discrepancies](demand-side/index.md#known-open-discrepancies)

---

## 6. Your first contribution

Not code. A PR against [this handbook](https://github.com/nzi-iitdelhi/documentation)
fixing everything that was wrong or unclear while you worked through it. Use the
:material-pencil: icon on any page.

That PR is your onboarding deliverable.

---

## Then

| Next | Go to |
|---|---|
| Run your own scenario | [Supply](supply-side/run-an-experiment.md) · [Demand](demand-side/run-an-experiment.md) |
| Change code | [Supply dev](supply-side/developer/index.md) · [Demand dev](demand-side/developer.md) |
| Vocabulary | [Glossary](reference/glossary.md) |
