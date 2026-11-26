# Period years

This page shows, for each MACRO period, which year each kind of input value comes from. A
MACRO period stands for a span of financial years, but the case reads each value at one
year. That year is not the same for all values. Use this page to find which year a value
comes from, and to check the effect of a change to the period rows.

The period rows are in the database: `PeriodLength`, `SilverYear`, `DemandYear` and
`PeriodOn` (table `bronze_nzi_base`, technology `MACRO`). When you change one of these rows or
a rule below, update this page in the same change.

## The agreed rule

1. PIER and MACRO both use Indian financial years (FY). A MACRO period reads its own PIER year:
   period 2031 reads PIER 2031.
2. Exception: legacy capacity. Retirements occur after the year they are listed in. So the
   legacy capacity of period 2031 is the legacy capacity of PIER 2030 minus the retirements
   of PIER 2030.
3. Period 2023 is the legacy build year. PIER has no 2023. The period exists only to give the
   legacy fleet a cost, and it uses the 2024 demand.
4. PIER ends in 2041. After 2041, a cost comes from the NZI projection in the period's own year.
   A value that no source gives reads 2041.

## Current mapping

The code does not follow the agreed rule yet. `SilverYear` still has the base case's years for
2024, 2031 and 2041 (columns 3 and 6 below). This is an open item.

| Period | FY covered (length) | SilverYear | Thermal new-build bounds | Availability | Demand | Legacy capacity: retirements removed up to |
|---|---|---|---|---|---|---|
| 2023 | 2023 (1) | 2024 | none: no new build | 2023 | 2024 | 2023 |
| 2024 | 2024 (1) | 2025 | 2024 | 2024 | 2024 | 2024 |
| 2031 | 2025–2031 (7) | 2030 | 2031 | 2031 | 2031 | 2029 |
| 2041 | 2032–2041 (10) | 2040 | 2041 | 2041 | 2041 | 2039 |
| 2051 | 2042–2051 (10) | 2051 | 2041 | 2041 | 2051 | 2040 |
| 2061 | 2052–2061 (10) | 2061 | 2041 | 2041 | 2061 | 2040 |
| 2071 | 2062–2071 (10) | 2071 | 2041 | 2041 | 2071 | 2040 |
| 2076 | 2072–2076 (5) | 2076 | 2041 | 2041 | 2076 | 2040 |

Notes on the columns:

1. **SilverYear** is the year for most values:
    1. Thermal: costs, heat rate, efficiency, lifetime and the other asset fields. A new build
       has SilverYear as its build year (vintage).
    2. Solar and wind: costs, and the minimum new build (cumulative minimum capacity).
    3. Hydro, storage and transmission: costs, lifetime and new-build bounds.
    4. Coal transport: cost and loss.
    5. Fuel nodes: fuel prices and supply limits.
    6. Legacy capacity (last column): PIER legacy capacity minus the retirements of the years
       before SilverYear.
2. **Thermal new-build bounds**: PIER `ECT_CapAddBounds` additions, summed from 2024 up to the
   period year (to 2041 after 2041).
3. **Availability**: the hourly file of the build year. A legacy fleet reads 2023. A new build
   reads its period year, up to 2041.
4. **Demand**: the file `ELECTRICITY_<DemandYear>.csv`.
5. After 2041, a value with no source reads 2041 (PIER's last year), whatever the column says.
   Costs from the NZI projection read SilverYear.

## Periods that are off by default

These periods are in the database but are off by default (`PeriodOn` = false). The Gold page
lists them unchecked. You cannot turn them on yet: they have no demand file, and no rule
splits the solar, wind and hydro potential for them.

| Period | Length when on | Takes its years from | SilverYear | Demand |
|---|---|---|---|---|
| 2025 | 1 | 2031 | 2025 | 2025 (no file) |
| 2036 | 5 | 2041 | 2035 | 2036 (no file) |
| 2046 | 5 | 2051 | 2046 | 2046 (no file) |
| 2056 | 5 | 2061 | 2056 | 2056 (no file) |
| 2066 | 5 | 2071 | 2066 | 2066 (no file) |
| 2074 | 3 | 2076 | 2074 | 2074 (no file) |

## A period that is off

1. A period that is off gives its years to the next period that is on. For example, with 2031
   off, period 2041 covers 2025–2041 (17 years).
2. Thermal bounds are cumulative, so the next period includes the additions of the period that
   is off.
3. The solar and wind share of potential of a period that is off goes to the next period that
   is on.
4. Periods 2023 and 2024 are always on.
5. Known gap: with 2071 off, period 2076 has no solar, wind or biomass build that can expand.

## Open items

1. Set `SilverYear` to the period's own year for 2024 (2024), 2031 (2031) and 2041 (2041),
   and for the new periods 2025 (2025) and 2036 (2036).
2. After that change, the legacy capacity of period 2031 removes the retirements up to 2030, as
   the agreed rule says.
3. Thermal bounds and availability use the period year, but the other values use SilverYear.
   When SilverYear is each period's own year, all values for 2024–2041 use the same year.
