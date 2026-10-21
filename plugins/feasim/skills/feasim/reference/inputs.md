# Inputs (v0.1.1)

The full type is `SliderInputs` in `dist/index.d.ts` (source: `src/calculations/calc-shared.ts`). Only `program` is required.

There are two layers of defaults:

- **`DEFAULT_INPUTS[program]`** (`src/parcels.ts`). `calcParcel` and `parcelInputs` always start from this.
- **The calculator's internal fallbacks** (`sliders.x ?? ...` in each `calc-*.ts`). These apply only to fields left out of a direct `calcSingle` call, and some differ from `DEFAULT_INPUTS`. For example, Standard's internal `hold_years` is 10, but `DEFAULT_INPUTS.standard.hold_years` is 8. And without `permitting_months` / `const_months`, the calculators look the timing up by zone.

## Parcel (for `calcParcel`)

| Field | Meaning |
| --- | --- |
| `lot_sqft` | Lot area, sqft (required) |
| `zone_class` | District code, exact case: `"A-1"`, `"A-2"`, `"A-3"`, `"AMX-1"`, `"AMX-2"`, `"AMX-3"`, `"B-1"`, `"B-2"`, `"BMX-3"`, `"BMX-4"`, `"R-5"`, `"I-1"`, `"I-2"`, `"I-3"`, `"IMX-1"`, Waikiki `"Apart"` / `"ApartMix"` (required) |
| `height` | Height limit, ft. `0` = unknown, so the pathway default is used (required) |
| `tmk` | ID copied onto the result |
| `parking_exempt` | § 21-6.20(a) no-parking area. Standard, Bill 6 (current zoning) and 201H only |
| `tod_regime` | `"pdt"` (inside an adopted TOD special district) or `"ipdt"` (elsewhere). Leave it out if the lot can't get a TOD permit |
| `slu_urban` | Entirely in the State Land Use urban district (TOD) |
| `station_area`, `tod_precinct` | Rail station area; `"core"` (≤ ¼ mi) or `"tiz"` (¼–½ mi) (TOD) |
| `tod_map_bonus_height` | PD-T bonus height from the TOD zoning map, ft |

## Levers / `SliderInputs`

The default columns are `DEFAULT_INPUTS` values. "Current" covers `bill6`, `standard`, `tod` and `h201`; the exceptions are listed under the table.

| Input | Unit | Current | Bill 7 | Notes |
| --- | --- | --- | --- | --- |
| `program` | | | | `"bill6"`, `"standard"`, `"bill7"`, `"tod"`, `"h201"`. Anything else runs `"bill6"` |
| `lot_sqft`, `zone_class` | | from parcel | from parcel | |
| `height_limit` | ft | from envelope | from envelope, max 60 | |
| `max_far` | FAR | from envelope | 4.0 | Overrides the envelope |
| `floor_height` | ft | 10 | 10 (fixed) | |
| `unit_type` | mix key | `"one_bb_100"` | `"studio50_1br50"` | `studio_100`, `studio75_1br25`, `studio50_1br50`, `studio25_1br75`, `one_bb_100`, `one_br50_2br50`, `two_bb_100`, `third_each`. Sizes: studio 300, 1BR 500, 2BR 700 sqft |
| `affordable_ami` | `"ami_NN"` | `"ami_80"` | `"ami_80"` | 10–140 in steps of 10 |
| `market_ami` | `"ami_NN"` | `"ami_120"` | `"ami_100"` | Market rent is modeled as an AMI rent |
| `affordability` | share | 201H: 0.51 | | 201H only; must be ≥ 0.51 |
| `include_retail` | bool | false | false | Ground-floor retail, only in zones that permit it (AMX, BMX, ApartMix) |
| `retail_rent` | $/sqft/mo | 5 | 3 | |
| `retail_share` | share of plate | 0.1 | 0.1 | |
| `hard_cost_per_sqft` | $/sqft building | 450 | 450 | Retail adds $15 |
| `land_cost` | $/sqft lot | 130 | 130 | Also the RLV benchmark in the verdict |
| `opex_ratio` | share of EGI | 0.30 (201H 0.19) | 0.19 | Excludes property tax |
| `waivable_fee_basis` | | `"market_units"` | `"none"` | Units charged building permit, plan review, wastewater and water fees: `"all_units"`, `"market_units"`, `"none"` |
| `park` | | `"in_lieu"` | `"exempt"` | Park dedication. Bill 7 waives it unless `"in_lieu"` |
| `impact_fee` | $/unit | 0 | n/a | School impact fee, currently 0 |
| `setback` | ft | 10 | 10 | |
| `coverage` | share | by zone | 0.8 | Building lot coverage cap |
| `parking_ratio` | stalls/unit | Article 6 minimum (TOD 0) | 0 | Leave unset for the legal minimum |
| `building_area_per_unit` | sqft | | 800 | Bill 7 legal unit cap: `lot × FAR ÷ 800` |
| `grant_type` | | | `"post_construct"` | Bill 7 only: `"none"`, `"pre_construct"`, `"post_construct"` |
| `grant_amount` | $/affordable unit | | 12000 | Internal fallback: $50k pre, $12k post |
| `permitting_months` | months | 18 | 18 | Internal fallback: by zone (Standard, Bill 6, Bill 7); 18 (TOD, 201H) |
| `const_months` | months | 12 | 12 | Internal fallback: by zone (18 / 24 / 36) |
| `leaseup_months` | months | 9 | 9 | Without it: units at 90% occupancy ÷ 20 a month + 3 |
| `equity` | share | 0.30 | 0.30 | Equity floor. The DSCR can push realized equity higher (`dev_equity_pct_realized`) |
| `construction_rate` | | 0.08 | 0.08 | |
| `permanent_rate` | | 0.0675 | 0.07 | |
| `amort_years` | years | 30 | 30 | |
| `min_dscr` | | 1.2 | 1.15 | Sizes both the construction take-out and the permanent loan |
| `hold_years` | years | 8 | 10 | `0` = develop and sell at stabilization (flip) |
| `target_return` | | 0.115 | 0.115 | Land carry rate used to discount the RLV residual |
| `profit_margin` | | 0.15 | 0.15 | Developer profit on project cost in the RLV |
| `cap_rate` | | 0.051 | 0.051 | Retail +1.5 points |
| `target_spread` | | 0.015 | 0.015 | Target ROC = cap rate + spread |
| `rental_split`, `sale_split` | | 1 / 0 | | 100% rental. For-sale is not active in v0.1.1 |
| `open_space_bonus`, `far_without_bonus` | FAR | from envelope | | § 21-3.120-2 bonus; `calcSingle` keeps it only if RLV rises |
| `max_units`, `front_no_parking_ft` | | from envelope | | TOD: IMX-1 unit cap, PD-T 40 ft front strip |
| `stepback_trigger_ft`, `stepback_above_ft`, `stepback_ft_per_10ft` | ft | by zone | by zone | Height step-backs |

**Exceptions to the "Current" column:** `tod` also sets `parking_ratio: 0` and `floor_height: 10`. `h201` sets `affordability: 0.51` and `opex_ratio: 0.19`.

## AMI rents

Rent = 30% of the HHFDC income limit for the household size ÷ 12, floored. The household sizes are: studio 1 person, 1BR the average of 1 and 2, 2BR 3. Income limits are the 2026 Honolulu table (`INCOME_LIMITS` in `calc-shared.ts`). Brackets not in the table are scaled from the 50% row.
