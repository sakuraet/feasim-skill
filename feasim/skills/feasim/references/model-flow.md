# Model flow (v0.5.0)

**Source:** `@uhero/feasim` v0.5.0 (https://github.com/UHERO/feasim, tag `v0.5.0`). **Last verified: November 17, 2026.**

These steps are traced through `calcBill7Single` in `src/calculations/calc-bill7.ts`. The other pathways follow the same order, with their own envelope, unit and fee rules; since v0.5.0 `bill6` and `standard` run one shared pipeline (`calc-far-pathway.ts`), and all four non-Bill 7 pathways share its hold/flip engine (`runDevFlipStrategy`, `runHoldStrategy`). Shared helpers are in `src/calculations/calc-shared.ts`. The narrative version, with legal citations, is `docs/simulator-logic.md` in the repo.

## 0. Dispatch (`calcSingle`, `src/calculations/index.ts`)

- Picks the calculator from `program`.
- If an open-space bonus is on offer (not Bill 7), it runs with and without the bonus. It keeps the higher RLV, or the bigger building when neither is positive. `open_space_bonus_mode` overrides this: `"auto"` (default) compares, `"on"` always takes the bonus, `"off"` never does.

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
- **Park dedication:** in-lieu = $200 + park land × `land_cost`, where park land = min(110 sqft × chargeable units, 10% of floor area built, bonus included). Chargeable units are those not exempt under § 22-7.3(j): market units in 201H and Bill 7, every unit elsewhere.
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
selling    = sellingCosts(gdv) = 0.035 × gdv + conveyanceTax(gdv)
residual   = gdv − selling − project_cost × 1.15
land_carry = (1 + target_return)^(total_months/12)
grants_pv  = Σ grant_flows[m] / (1 + target_return)^(m/12)
rlv        = residual / land_carry + grants_pv
```

Here `project_cost` = `dev_cost` + `rlv_financing_cost`, and `rlv_sqft` = `rlv / lot_sqft`.

**Conveyance tax** (`conveyanceTax`, HRS § 247-2, new in v0.5.0): the seller pays the bracket's rate on the whole price, not marginally. A whole rental building always takes the general schedule: 0.10% under $600k, 0.20% to $1M, 0.30% to $2M, 0.50% to $4M, 0.70% to $6M, 0.90% to $10M, 1.00% at $10M or more. So a $9.9M sale pays $89,100 and a $10M sale $100,000. The `condo_sfr` schedule (0.15–1.25%) exists but no pathway uses it.

## 7. Returns

**Flip** (`hold_years = 0`):

- Monthly cash flows: equity draws, grants as paid, then `gdv − sellingCosts(gdv) − loan payoff` at `total_months`.
- `dev_levered_irr` is the monthly IRR, annualized (`solveIRR`).
- Equity multiple = (exit proceeds + grants) ÷ equity.
- ROC = NOI ÷ (total cost incl. land and financing − grants at face value).

**Hold:**

- **Permanent loan** (`calcPermanentLoan`): min(construction balance, DSCR-sized loan on year-1 NOI after tax).
- **Growth each year** (`holdEgi`): affordable rent income 4%; market rent, parking and retail 3% (4% less the 1% aging drag, `HOLD.age_rent_drag`); opex 3.5%; assessed value 2%.
- **Exit cap** (`exitCapRate`): entry cap + 7.5 bps per year held, capped at 75 bps (10+ years). At 5.1%: 1 yr 5.175%, 5 yr 5.475%, 8 yr 5.70%, 10+ yr 5.85%.
- **Exit:** forward NOI (grown at the next year's `holdEgi` growth) ÷ exit cap, minus `sellingCosts` on that price and the remaining loan balance.
- **IRR** (`solveIRR`): bisection; below 0% it tests the sign on future value, so long holds and very large parcels no longer return `NaN` from overflow.
- **Cash multiple:** equity multiple = the sum of positive monthly flows ÷ equity.
