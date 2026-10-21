# Model flow (v0.1.1)

**Source:** `@uhero/feasim` v0.1.1 (https://github.com/UHERO/feasim, tag `v0.1.1`). **Last verified: October 21, 2026.**

These steps are traced through `calcBill7Single` in `src/calculations/calc-bill7.ts`. The other pathways follow the same order, with their own envelope, unit and fee rules. Shared helpers are in `src/calculations/calc-shared.ts`.

## 0. Dispatch (`calcSingle`, `src/calculations/index.ts`)

- Picks the calculator from `program`.
- If an open-space bonus is on offer (not Bill 7), it runs with and without the bonus. It keeps the higher RLV, or the bigger building when neither is positive.

## 1. Envelope and units

| Step | Function | Rule |
| --- | --- | --- |
| Floor plates | `floorPlates` | `floor(height / 10)` floors; each plate is the smaller of coverage × lot and the setback footprint (setbacks widen above the step-back height) |
| Stack | `podiumEnvelope` | Floor area up to `max_far × lot_sqft` (parking counts toward height, not FAR). Ground-floor retail where allowed. Stalls go on a surface lot if they all fit beside a slimmer building; otherwise on the ground floor, then podium levels. Loading and bicycle parking take ground in the parking area |
| Units | `podiumEnvelope` | `floor(residential_sqft × 0.8 / avg unit size)`, capped by any legal unit cap |
| Affordable | per pathway | Bill 7 `ceil(0.8 × units)`; 201H `ceil(affordability × units)`; others 0 |

## 2. Costs

```
building_cost = (residential + lobby sqft) × hard_cost_per_sqft + retail sqft × (rate + 15)
hard_cost     = (building_cost + parking_cost) × 1.056^((permitting + const/2) / 12)
soft_cost     = calcSoftCosts(...).total
dev_cost      = hard_cost × 1.10 + soft_cost
```

`calcSoftCosts` items:

- **Waivable fees** (charged on the units set by `waivable_fee_basis`): building permit and plan review (scaled by hard cost per unit), wastewater, and the BWS water system facilities charge and meter (`calcWaterFees`).
- **Every pathway:** environmental ($1,800), utilities ($245/unit), grubbing / grading / stockpiling (based on excavation volume), temporary occupancy, ESCP, and construction management at 3% of hard cost.
- **Park dedication:** in-lieu = $200 + park land × `land_cost`.
- **TOD only:** zoning application and special district fees.

Parking cost: $190/sqft at 330 sqft/stall for structured parking; $30/sqft at 360 sqft/stall for surface.

## 3. Financing

```
loan (LTC)   = amount_to_finance × (1 − equity)
take-out     = NOI / (min_dscr × annual loan constant)   (holds only)
loan         = min(LTC loan, loan whose stabilized balance = take-out)
equity       = base + 0.75% loan fee − loan − (pre-construction grant)
```

- **Two loans.** `rlv_financing_cost` comes from a loan sized on construction cost only (RLV basis). `total_financing_cost` comes from the deal loan, sized with land in the base.
- **Construction interest** (`calcConstructionInterest`): 8%/yr, compounded monthly, on an S-curve draw (`3t² − 2t³`).
- **Lease-up carry** (`calcLeaseupCarry`): interest on the full balance for `leaseup_months`.
- **Pre-development** (`calcPredevEquity`): 10% of construction cost per year, during permitting.
- **Equity order** (`equityFirstDraws`): land first, then pre-development, then the rest at loan closing. Flips use this schedule. Bill 7 holds put all equity at month 0.

## 4. Revenue and NOI

| Item | Rule |
| --- | --- |
| Affordable rent escalation | `1.04^((const + carry) / 12)` |
| Market rent escalation | `1.04^((permitting + const + carry) / 12)` |
| PRI | rent × units × 12 |
| PGI | PRI + $100/stall/month parking + admin income ($50 application on 30% of units, $200 renewal on 70%) |
| EGI | PGI × (1 − 7.4%); retail × (1 − 12.6%) |
| NOI | EGI − `opex_ratio` × EGI − property tax |

Property tax is `0.35% × (land + hard cost)`, or land only while an improvement exemption runs. It's split between residential and retail by EGI.

## 5. Value and RLV (`calcDevRLV`)

```
gdv       = residential NOI / cap_rate + retail NOI / (cap_rate + 0.015)
residual  = gdv × (1 − 0.035) + post_grant − project_cost × 1.15
rlv       = residual / (1 + target_return)^(total_months/12) + pre_subsidy
```

Here `project_cost` = `dev_cost` + `rlv_financing_cost`, and `rlv_sqft` = `rlv / lot_sqft`.

## 6. Returns

**Flip** (`hold_years = 0`):

- Monthly cash flows: equity draws, then `gdv − 3.5% − loan payoff (+ post grant)` at `total_months`.
- `dev_levered_irr` is the monthly IRR, annualized (`solveIRR`).
- ROC = NOI ÷ (total cost incl. land and financing − grants).

**Hold:**

- **Permanent loan** (`calcPermanentLoan`): min(construction balance, DSCR-sized loan on year-1 NOI after tax).
- **Growth each year:** rent 3.5%, opex 3.5%, assessed value 2%.
- **Exit:** year-`hold_years` forward NOI ÷ (cap + 0.5 points), minus 3.5% and the remaining loan balance.
- **Cash multiple:** equity multiple = the sum of positive monthly flows ÷ equity.
