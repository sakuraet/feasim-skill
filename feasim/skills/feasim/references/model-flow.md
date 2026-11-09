# Model flow (v0.3.0)

**Source:** `@uhero/feasim` v0.3.0 (https://github.com/UHERO/feasim, tag `v0.3.0`). **Last verified: November 9, 2026.**

These steps are traced through `calcBill7Single` in `src/calculations/calc-bill7.ts`. The other pathways follow the same order, with their own envelope, unit and fee rules. Shared helpers are in `src/calculations/calc-shared.ts`. The narrative version, with legal citations, is `docs/simulator-logic.md` in the repo.

## 0. Dispatch (`calcSingle`, `src/calculations/index.ts`)

- Picks the calculator from `program`.
- If an open-space bonus is on offer (not Bill 7), it runs with and without the bonus. It keeps the higher RLV, or the bigger building when neither is positive.

## 1. Envelope and units

| Step | Function | Rule |
| --- | --- | --- |
| Floor plates | `floorPlates` | `floor(height / 10)` floors; each plate is the smaller of coverage × lot and the yard footprint on a 2:1 deep lot (yards widen above the step-back height) |
| Stack | `podiumEnvelope` | Floor area up to `max_far × lot_sqft` (parking, loading, bicycle parking and lobby count toward height, not FAR). Ground-floor retail where allowed. Stalls go on a surface lot if they all fit beside a slimmer building; otherwise on the ground floor, then whole podium levels, keeping the arrangement with the most units |
| Units | `podiumEnvelope` | `floor(residential_sqft × 0.8 / avg unit size)`. With a unit cap (`unit_cap`: Bill 7's legal cap, TOD's IMX-1 cap), the building stops at the floor area the capped units need |
| Built size | `podiumEnvelope` | `built_floors` = ground floor + podium levels + the storeys the housing fills; `built_height` = `built_floors × floor_height` |
| Affordable | per pathway | Bill 7 `ceil(0.8 × units)`; 201H `ceil(affordability × units)`; others 0 |

Stalls (Article 6 formula, used when `parking_ratio` is unset): `round(j × (r × units + retail_sqft / 500))`, with `r` = average unit size ÷ 1,000 and `j` = 0.9 when housing and retail both need stalls. A set `parking_ratio` replaces `r`.

## 2. Costs

```
building_cost  = (residential + lobby sqft) × hard_cost_per_sqft + retail sqft × (rate + 15)
open_shop_hard = (building_cost + parking_cost) × 1.056^((permitting + const/2) / 12)
pw_premium     = open_shop_hard × onsite_labor_share × prevailing_wage_premium   (Bill 7 pre-construction only)
hard_cost      = open_shop_hard + pw_premium
soft_cost      = calcSoftCosts(...).total
dev_cost       = hard_cost × 1.10 + soft_cost
```

`calcSoftCosts` items:

- **Waivable fees** (charged on the units set by `waivable_fee_basis`): building permit and plan review (scaled by hard cost per unit), wastewater, and the BWS water system facilities charge and meter (`calcWaterFees`).
- **Every pathway:** environmental ($1,800), utilities ($245/unit), grubbing / grading / stockpiling (based on excavation volume), temporary occupancy, ESCP, and construction management at 3% of hard cost.
- **Park dedication:** in-lieu = $200 + park land × `land_cost`.
- **TOD only:** zoning application and special district fees.

Parking cost: $190/sqft at 330 sqft/stall for structured parking; $30/sqft at 360 sqft/stall for surface. Bicycle racks and lockers cost $225 and $1,250 per space.

## 3. Grants (Bill 7)

```
post_grant  = affordable_units × grant_amount                      (post_construct)
subsidy     = min(subsidy_units × grant_amount,                     (pre_construct)
                  open_shop_hard × onsite_labor_share + pw_premium)
grant_flows = subsidy ÷ const_months in each month (permitting, permitting + const]
            + post_grant at month permitting + const
```

## 4. Financing

```
loan (LTC)   = amount_to_finance × (1 − equity)
take-out     = NOI / (min_dscr × annual loan constant)   (holds only)
loan         = min(LTC loan, loan whose stabilized balance = take-out)
equity       = base + 0.75% loan fee − loan
```

- **Grants don't touch the financing.** Since v0.2.0 neither grant is a source at closing.
- **Two loans.** `rlv_financing_cost` comes from a loan sized on construction cost only (RLV basis). `total_financing_cost` comes from the deal loan, sized with land in the base.
- **Construction interest** (`calcConstructionInterest`): 8%/yr, compounded monthly, on an S-curve draw (`3t² − 2t³`).
- **Lease-up carry** (`calcLeaseupCarry`): interest on the full balance for `leaseup_months`.
- **Pre-development** (`calcPredevEquity`): 10% of construction cost per year, during permitting.
- **Equity order** (`equityFirstDraws`): land at month 0, then pre-development through permitting, then the rest at loan closing. Flips and holds use this schedule in every pathway (holds used to put all equity at month 0).

## 5. Revenue and NOI

| Item | Rule |
| --- | --- |
| Rent escalation (affordable and market) | `1.04^((permitting + const + carry) / 12)`. Affordable caps follow each year's HUD limits, so they escalate through permitting too |
| PRI | rent × units × 12 |
| PGI | PRI + $100/stall/month parking + admin income ($50 application on 30% of units, $200 renewal on 70%) |
| EGI | PGI × (1 − 7.4%); retail × (1 − 12.6%) |
| NOI | EGI − `opex_ratio` × EGI − property tax |

Property tax is `0.35% × (land + hard cost)`, or land only while an improvement exemption runs. Bill 7 uses the market-wage hard cost (no premium). Tax is split between residential and retail by EGI.

## 6. Value and RLV (`calcDevRLV`)

```
gdv        = residential NOI / cap_rate + retail NOI / (cap_rate + 0.015)
residual   = gdv × (1 − 0.035) − project_cost × 1.15
land_carry = (1 + target_return)^(total_months/12)
grants_pv  = Σ grant_flows[m] / (1 + target_return)^(m/12)
rlv        = residual / land_carry + grants_pv
```

Here `project_cost` = `dev_cost` + `rlv_financing_cost`, and `rlv_sqft` = `rlv / lot_sqft`.

## 7. Returns

**Flip** (`hold_years = 0`):

- Monthly cash flows: equity draws, grants as paid, then `gdv − 3.5% − loan payoff` at `total_months`.
- `dev_levered_irr` is the monthly IRR, annualized (`solveIRR`).
- Equity multiple = (exit proceeds + grants) ÷ equity.
- ROC = NOI ÷ (total cost incl. land and financing − grants at face value).

**Hold:**

- **Permanent loan** (`calcPermanentLoan`): min(construction balance, DSCR-sized loan on year-1 NOI after tax).
- **Growth each year:** rent 4% (affordable and market), opex 3.5%, assessed value 2%.
- **Exit:** year-`hold_years` forward NOI ÷ (cap + 0.5 points), minus 3.5% and the remaining loan balance.
- **Cash multiple:** equity multiple = the sum of positive monthly flows ÷ equity.
