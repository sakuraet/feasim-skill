# Inputs (v0.3.0)

**Source:** `@uhero/feasim` v0.3.0 (https://github.com/UHERO/feasim, tag `v0.3.0`). **Last verified: November 9, 2026.**

The full type is `SliderInputs` in `dist/index.d.ts` (source: `src/calculations/calc-shared.ts`). Only `program` is required. Every lever, with its source, is in `docs/variables.md` in the repo.

There are two layers of defaults:

- **`DEFAULT_INPUTS[program]`** (`src/parcels.ts`). `calcParcel` and `parcelInputs` always start from this.
- **The calculator's internal fallbacks** (`sliders.x ?? ...` in each `calc-*.ts`). These apply only to fields left out of a direct `calcSingle` call, and some differ from `DEFAULT_INPUTS`:
  - `hold_years` falls back to 10 in every pathway; `DEFAULT_INPUTS` sets 8 (Bill 7: 10).
  - An unset `parking_ratio` gives the Article 6 legal minimum (average unit size ÷ 1,000, or 0 on an exempt parcel) in `bill6`, `standard` and `h201`; 0.85 in `tod`; 0.5 in `bill7`. `calcParcel` can't do this, since it always fills the lever from `DEFAULT_INPUTS`.
  - An unset `leaseup_months` is computed from absorption (see below) instead of 9.

## Parcel (for `calcParcel`)

| Field | Meaning |
| --- | --- |
| `lot_sqft` | Lot area, sqft (required) |
| `zone_class` | District code, exact case: `"A-1"`, `"A-2"`, `"A-3"`, `"AMX-1"`, `"AMX-2"`, `"AMX-3"`, `"B-1"`, `"B-2"`, `"BMX-3"`, `"BMX-4"`, `"R-5"`, `"I-1"`, `"I-2"`, `"I-3"`, `"IMX-1"`, Waikiki `"Apart"` / `"ApartMix"` (required) |
| `height` | Height limit, ft. `0` = unknown, so the pathway default is used (required) |
| `tmk` | ID copied onto the result |
| `parking_exempt` | § 21-6.20(a) no-parking area. Standard, Bill 6 (current zoning) and 201H only. Removes retail stalls; housing stalls still follow the `parking_ratio` lever, so at the defaults with retail off it changes nothing |
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
| `height_limit` | ft | from envelope | from envelope (max 60) | A lever overrides the envelope, including Bill 7's 60 ft cap |
| `max_far` | FAR | from envelope | 4.0 | Overrides the envelope |
| `floor_height` | ft | 10 | 10 (fixed) | |
| `unit_type` | mix key | `"studio50_1br50"` | `"studio50_1br50"` | `studio_100`, `studio75_1br25`, `studio50_1br50`, `studio25_1br75`, `one_bb_100`, `one_br50_2br50`, `two_bb_100`, `third_each`. Sizes: studio 300, 1BR 500, 2BR 700 sqft. Anything else is treated as all one-bedroom |
| `affordable_ami` | `"ami_NN"` | `"ami_80"` | `"ami_80"` | 10–140 in steps of 10 |
| `market_ami` | `"ami_NN"` | `"ami_120"` | `"ami_100"` | Market rent is modeled as an AMI rent |
| `affordability` | share | 201H: 0.51 | | 201H only; below 50% + 1 unit, `calcSingle` throws |
| `include_retail` | bool | false | false | Ground-floor retail, only in zones that permit it (AMX, BMX, ApartMix) |
| `retail_rent` | $/sqft/mo | 5 | 3 | |
| `retail_share` | share of plate | 0.1 | 0.1 | |
| `hard_cost_per_sqft` | $/sqft building | 450 | 450 | Retail adds $15 |
| `land_cost` | $/sqft lot | 130 | 130 | Also the RLV benchmark in the verdict |
| `opex_ratio` | share of EGI | 0.30 (201H 0.19) | 0.19 | Excludes property tax |
| `waivable_fee_basis` | | `"market_units"` | `"none"` | Units charged building permit, plan review, wastewater and BWS water fees: `"all_units"`, `"market_units"`, `"none"` |
| `park` | | `"in_lieu"` | `"exempt"` | Park dedication. Bill 7 waives it unless `"in_lieu"` |
| `impact_fee` | $/unit | 0 | n/a | School impact fee, currently 0 |
| `setback` | ft | 10 | 10 | |
| `coverage` | share | by zone | 0.8 | Building lot coverage cap |
| `parking_ratio` | stalls/unit | 0.95 (TOD 0.85) | 0.5 | Flat defaults, set on exempt parcels too. Not legal minimums: Bill 7 and TOD require no car parking. Values below the Article 6 minimum are allowed |
| `building_area_per_unit` | sqft | | 800 | Bill 7 legal unit cap: `floor(lot × FAR ÷ 800)`. The building stops at the floor area those units need |
| `grant_type` | | | `"post_construct"` | Bill 7 only: `"none"`, `"pre_construct"`, `"post_construct"`. Anything else pays no grant |
| `grant_amount` | $/unit | | not set | Not in `DEFAULT_INPUTS`. Left out: $50,000 per unit pre-construction; post-construction $12,000 per affordable unit, or $40/sqft for units under 300 sqft (`mixPostGrant`). A value you pass applies to every unit and overrides the small-unit rule. The pre-construction subsidy is still capped |
| `grant_units` | | | `"all"` (fallback) | Bill 7 pre-construction only: units the subsidy pays on, `"all"` or `"affordable"`. Post-construction always pays on affordable units |
| `prevailing_wage_premium` | share | | 0.45 (fallback) | Bill 7 pre-construction only: extra cost of on-site labor at prevailing wages |
| `onsite_labor_share` | share | | 0.25 (fallback) | Bill 7: on-site labor as a share of hard cost. Sets the premium and the subsidy cap, so setting it low cuts the subsidy |
| `permitting_months` | months | 18 | 18 | Internal fallback: 18 in every pathway |
| `const_months` | months | 12 | 12 | Internal fallback: 12 in every pathway. `0` is treated as unset |
| `leaseup_months` | months | 9 | 9 | Without it: `ceil(rental units × 90% ÷ 20)` + 3 |
| `equity` | share | 0.30 | 0.30 | Equity floor. The DSCR can push realized equity higher (`dev_equity_pct_realized`) |
| `construction_rate` | | 0.08 | 0.08 | |
| `permanent_rate` | | 0.0675 | 0.07 | Must be above 0, or IRR, equity multiple, profit and ROI become `NaN` |
| `amort_years` | years | 30 | 30 | Must be above 0 (same as above) |
| `min_dscr` | | 1.2 | 1.15 | Sizes both the construction take-out and the permanent loan |
| `hold_years` | years | 8 | 10 | `0` = develop and sell at stabilization (flip) |
| `target_return` | | 0.115 | 0.115 | Land carry rate used to discount the RLV residual and the grants |
| `profit_margin` | | 0.15 | 0.15 | Developer profit on project cost in the RLV |
| `cap_rate` | | 0.051 | 0.051 | Retail +1.5 points |
| `target_spread` | | 0.015 | 0.015 | Target ROC = cap rate + spread |
| `rental_split`, `sale_split` | | 1 / 0 | | 100% rental. For-sale is not active, and the code doesn't read these |
| `open_space_bonus`, `far_without_bonus` | FAR | from envelope | | § 21-3.120-2 bonus; `calcSingle` keeps it only if RLV rises |
| `max_units`, `front_no_parking_ft` | | from envelope | | TOD: IMX-1 unit cap, PD-T 40 ft front strip |
| `stepback_trigger_ft`, `stepback_above_ft`, `stepback_ft_per_10ft` | ft | by zone | by zone | Height step-backs |

**Exceptions to the "Current" column:** `tod` sets `parking_ratio: 0.85` and `floor_height: 10`. `h201` sets `affordability: 0.51` and `opex_ratio: 0.19`.

## AMI rents

Rent = 30% of the HHFDC income limit for the household size ÷ 12, floored. The household sizes are: studio 1 person, 1BR the average of 1 and 2, 2BR 3. Income limits are HHFDC's **2025** Honolulu County schedule (median $129,300; `INCOME_LIMITS` in `calc-shared.ts`). The 2026 schedule is published but not yet in the model. Brackets not in the table (e.g. `"ami_85"`) are scaled from the 50% row and won't match published limits.
