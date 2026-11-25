# Add an input file

This guide tells you how to make the pipeline accept and read a new input file. You add the file in code and send it as a pull request. Each input file has its own module, and each kind of MACRO asset has its own translator. A new file therefore edits no shared lines, so the pull request can stay open until review without conflicts. The full reference is the [Developer](developer.md) page.

## 1. Bronze: accept and check the file

1. In the supply-side repository, copy a module of the same kind in `supply_side/prescenario/bronze/sources/pier/` (PIER files) or `bronze/sources/projections/` (NZI projection files).
2. Name the copy after the file in snake case, for example `ECT_RampRates.csv` becomes `ect_ramp_rates.py`.
3. Change `PATH`, `COLUMNS` and `Schema`:

```python
"""PIER ECT_RampRates.csv: how fast a generator can change its output."""

from supply_side.prescenario.bronze.schemas.pier import AYear, Name, Place, Row, _value
from supply_side.prescenario.common.value_types import Fraction

PATH = "Supply/Parameters/Technologies/ECT_RampRates.csv"
COLUMNS = {"technology": "EnergyConvTech", "year": "Year"}


class Schema(Row):  # one field per CSV column, in the file's own name
    EnergyConvTech: Name
    ModelGeography: Place
    SubGeography1: Place
    Year: AYear
    RampUp: _value(Fraction)
```

4. The input page now lists the file as allowed, checks each upload against `Schema`, and Bronze stores it.
5. After you add the module, each build needs the file. Upload it in a new input set before you build.

## 2. Silver: give the values a meaning

1. If a column has a Silver name under a different spelling, add `SAME_AS = {"RampUp": "RampUpFraction"}` to the module.
2. If the file does not state a unit, add `UNITS = {"RampUp": ("fraction", "share of capacity per hour", "by definition")}` to the module.
3. If you must calculate a value from other values, add a rule in `silver/derive.py`.

## 3. Gold: use the values in MACRO

1. Each kind of asset has its own translator in `gold/macro/assets/`, for example `storage.py`. Map the Silver parameter to a MACRO field in that file. Thermal fields are in `FIELDS` in `gold/macro/common.py`.
2. A new kind of asset is a new file in `gold/macro/assets/` with a `write(ctx)` function. Copy `links.py` as a start.

## 4. Test and send

1. Add one test in `tests/prescenario/` of the supply-side repository that reads the file and checks one value in the MACRO case.
2. Run `make test`.
3. Open a pull request.
