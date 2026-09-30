# NEM Price Volatility and Solar Penetration: Clean Datasets

Cleaned and transformed datasets supporting a Master of Business Analytics capstone project (DATA6000, Kaplan Business School). The project analyses wholesale electricity price volatility across the five regions of Australia's National Electricity Market (NEM), from July 2021 to June 2026. It relates price behaviour to demand, generation mix and rooftop solar penetration, and extends the series with forecasts to December 2030.

These files were exported from the final Power BI data model (via DAX Studio). All cleaning and transformation steps are already applied.


## Contents


| File | Description | Grain | Coverage | Rows | Size |
|---|---|---|---|---:|---:|
| `PriceDemand.csv` | Interval-level regional price and demand | 5-min (30-min before Oct 2021) × region | 1 Jul 2021 – 30 Jun 2026 | 2,519,040 | 393 MB |
| `Daily_PriceDemand.csv` | Daily average price and demand (actuals) | Day × region | 1 Jul 2021 – 30 Jun 2026 | 9,135 | 0.7 MB |
| `LSTM_Forecast.csv` | Forecast daily price and demand (LSTM model) | Day × region | 2 Jul 2026 – 31 Dec 2030 | 8,220 | 0.5 MB |
| `PriceDemand_Combined.csv` | Actuals and forecasts stacked in one table | Day × region | 1 Jul 2021 – 31 Dec 2030 | 17,355 | 1.2 MB |
| `Solar.csv` | Rooftop solar PV output | 30-min × region | 1 Jul 2021 – 30 Jun 2026 | 810,190 | 68 MB |
| `GenerationMix.csv` | Daily generation by fuel type, with mean temperature | Day × region × fuel type | 26 Jul 2025 – 26 Jul 2026 | 24,888 | 1.7 MB |
| `Population Per Region.csv` | Estimated resident population | Quarter × region | Sep 2021 – Dec 2025 | 90 | < 0.1 MB |

**Regions:** `NSW`, `QLD`, `SA`, `TAS`, `VIC` (the NEM region codes NSW1, QLD1, SA1, TAS1 and VIC1, with the trailing "1" removed).


## Data sources

| Dataset | Source |
|---|---|
| Price and demand (`PriceDemand`, `Daily_PriceDemand`) | AEMO – NEMWeb / MMSDM archive (DISPATCH / TRADING price and demand) |
| Rooftop solar (`Solar`) | AEMO – NEMWeb / MMSDM archive (rooftop PV actual estimates) |
| Generation mix (`GenerationMix`) | Open Electricity - Daily generation by fuel technology and regional mean temperature |
| Population (`Population Per Region`) | ABS - Quarterly state population estimates |
| Forecasts (`LSTM_Forecast`) | Model output produced for this project |

Raw AEMO data was ingested with Python into a MySQL database, then loaded and transformed in Power BI (Power Query) before export.

---

## Data dictionary

### `PriceDemand.csv`

| Column | Type | Unit | Description |
|---|---|---|---|
| `Region` | text | – | NEM region |
| `SETTLEMENTDATE` | datetime | AEST (UTC+10) | End of the dispatch/trading interval |
| `TOTALDEMAND` | decimal | MW | Regional operational demand for the interval |
| `RRP` | decimal | $/MWh | Regional Reference Price (spot price) |
| `PERIODTYPE` | text | – | AEMO period type (always `TRADE`) |
| `IntervalHours` | decimal | hours | Interval length: `0.5` (30-min) until 30 Sep 2021, `0.0833` (5-min) from 1 Oct 2021 |
| `EnergyGWh` | decimal | GWh | Energy for the interval = `TOTALDEMAND × IntervalHours / 1000` |
| `Date` | date | – | Calendar date of `SETTLEMENTDATE` |
| `Year` | integer | – | Calendar year |
| `Month_Number` | integer | – | Month (1–12) |
| `Month_Name` | text | – | Month name |
| `TimeOfDay` | time | – | Time part of `SETTLEMENTDATE` |

### `Daily_PriceDemand.csv`

| Column | Type | Unit | Description |
|---|---|---|---|
| `Region` | text | – | NEM region |
| `Date` | date | – | Calendar date |
| `RRP` | decimal | $/MWh | Daily average spot price |
| `TotalDemand` | decimal | MW | Daily average operational demand |
| `Type` | text | – | Always `Actual` |

### `LSTM_Forecast.csv`

| Column | Type | Unit | Description |
|---|---|---|---|
| `Date` | date | – | Forecast date |
| `Region` | text | – | NEM region |
| `RRP` | decimal | $/MWh | Forecast daily average spot price |
| `TotalDemand` | decimal | MW | Forecast daily average operational demand |
| `Type` | text | – | Always `Forecast` |

### `PriceDemand_Combined.csv`

Union of `Daily_PriceDemand` and `LSTM_Forecast`, with the same columns (`Region`, `Date`, `RRP`, `TotalDemand`, `Type`). Filter on `Type` (`Actual` / `Forecast`) to separate the two.

### `Solar.csv`

| Column | Type | Unit | Description |
|---|---|---|---|
| `IntervalDateTime` | datetime | AEST (UTC+10) | Half-hour interval |
| `Region` | text | – | NEM region |
| `RooftopMW` | decimal | MW | Estimated rooftop PV output |
| `Date` | date | – | Calendar date of the interval |
| `Time` | time | – | Time part of the interval |

### `GenerationMix.csv`

| Column | Type | Unit | Description |
|---|---|---|---|
| `date` | date | – | Calendar date |
| `Temperature Mean - C` | decimal | °C | Daily mean temperature for the region |
| `Region` | text | – | NEM region |
| `Fuel Type` | text | – | Generation or flow technology (20 types, see below) |
| `GWh` | decimal | GWh | Daily energy. Negative values represent loads or outflows (battery charging, pumps, exports, curtailment) |
| `Index` | integer | – | Helper column used in the Power BI model (always `1`) |
| `Category` | text | – | `Renewable`, `Fossil` or `Other` |

Fuel types by category:

- **Renewable:** Bioenergy (Biomass), Hydro, Solar (Rooftop), Solar (Utility), Wind
- **Fossil:** Coal (Black), Coal (Brown), Distillate, Gas (CCGT), Gas (OCGT), Gas (Reciprocating), Gas (Steam), Gas (Waste Coal Mine)
- **Other:** Battery (Charging), Battery (Discharging), Exports, Imports, Pumps, Solar (Utility) (Curtailment), Wind (Curtailment)

### `Population Per Region.csv`

| Column | Type | Unit | Description |
|---|---|---|---|
| `Date` | date | – | Quarter reference date (1st of Mar / Jun / Sep / Dec) |
| `Region` | text | – | State matching the NEM region |
| `Population` | integer | persons | Estimated resident population |

---

## Notes

- **Interval-ending timestamps.** `SETTLEMENTDATE` marks the end of each interval. As a result, the final interval (`2026-07-01 00:00`) appears as a separate day, 1 July 2026, in `PriceDemand` and `Daily_PriceDemand`. That day is based on a single interval and should be excluded from daily analysis.
- **5-minute settlement.** The NEM moved from 30-minute to 5-minute settlement on 1 October 2021, which is why `IntervalHours` has two values. Use `EnergyGWh` (not row counts) when aggregating energy across that change.
- **Duplicate rows in `Solar.csv`.** 372,000 rows are exact duplicates of other rows. Deduplicate on `IntervalDateTime` + `Region` before summing `RooftopMW`.
- **Generation mix coverage.** `GenerationMix.csv` covers one year only (26 Jul 2025 – 26 Jul 2026) and is not aligned to the five-year price window.

---

## Licence and attribution

AEMO data is used under AEMO's [copyright permissions](https://www.aemo.com.au/privacy-and-legal-notices/copyright-permissions), which allow use with attribution. Source: © Australian Energy Market Operator (AEMO). Third-party datasets remain subject to their providers' terms. These files were prepared for academic purposes.
