# Fixed assumptions (v0.3.0)

**Source:** `@uhero/feasim` v0.3.0 (https://github.com/UHERO/feasim, tag `v0.3.0`). **Last verified: November 9, 2026.**

These are constants in `src/calculations/calc-shared.ts` (and `DEFAULT_INPUTS` in `src/parcels.ts`). Users can't change the fixed ones without editing the code. `docs/variables.md` in the repo lists every constant with its source and review cycle.

| Constant | Value | Where | Basis (from code comments / docs) |
| --- | --- | --- | --- |
| Building efficiency | 80% | `EFFICIENCY` | Net rentable ÷ gross |
| Unit sizes | studio 300, 1BR 500, 2BR 700, 3BR 900, 4BR 1,200 sqft | `UNIT_SIZE` | ROH § 32-2.9 maximum affordable unit sizes, used for every pathway |
| Default unit mix | Half studio, half 1BR (all pathways) | `DEFAULT_INPUTS.unit_type` | Was all 1BR outside Bill 7 before v0.3.0 |
| Hard cost contingency | 10% | `FINANCING.contingency` | |
| Construction cost escalation | 5.6%/yr to the construction midpoint | `FINANCING.cost_inflation` | |
| Retail cost premium | +$15/sqft | `RETAIL_PREMIUM` | |
| Construction management | 3% of hard cost | `CONSTRUCTION_MGMT_RATE` | |
| Structured parking | $190/sqft, 330 sqft/stall | `PARKING` | RLB Q2 2026 Honolulu (low end); Ulupono 2020 |
| Surface parking | $30/sqft, 360 sqft/stall | `PARKING` | Ulupono 2020, adjusted to 2026 |
| Default parking ratio | 0.95 stalls/unit (Bill 7 0.5, TOD 0.85) | `DEFAULT_INPUTS.parking_ratio`, `PARKING.stalls_per_unit` | Flat assumption, not a legal minimum; applies on exempt parcels too |
| Parking income | $100/stall/month | `PARKING.monthly_rent_per_stall` | |
| Bicycle parking | $225 rack, $1,250 locker, 22 sqft/space | `BICYCLE_PARKING` | GoodLockers, Nov 2025 |
| Timing | 18 months permitting, 12 construction, 9 lease-up | `DEFAULT_PERMITTING_MONTHS`, `DEFAULT_CONST_MONTHS`, `DEFAULT_INPUTS` | Same for every pathway and zone (the zone lookup was removed in v0.2.0) |
| Construction loan rate | 8% | `FINANCING.construction_rate` | |
| Loan fee | 0.75% | `FINANCING.loan_fee` | HHFDC typical |
| Pre-development spend | 10% of construction cost per year of permitting | `FINANCING.cost_carried` | |
| AMI / rent escalation to lease-up | 4%/yr, affordable and market, from today through permitting, construction and lease-up | `FINANCING.ami_escalation` | Honolulu MFI / HUD income limit growth |
| Hold rent growth | 4%/yr, all units | `HOLD.annual_rent_growth` | Income limits +4.4%/yr 2015–2026; market rents 3.6–4.3%/yr (was a 3.5% blend before v0.2.1) |
| Hold opex growth | 3.5%/yr | `HOLD.annual_opex_growth` | California TCAC LIHTC cash-flow rule (4 CCR § 10327(g)(1)) |
| Assessment growth | 2%/yr | `HOLD.annual_assessment_growth` | Same TCAC rule |
| Property tax | 0.35% | `FINANCING.rpt_residential` | Honolulu residential rate |
| Tax exemption | 15 years on improvements | `HOLD.tax_exemption_years` | Bill 7 (≤ 80% AMI), TOD |
| Vacancy | 7.4% residential, 12.6% retail | `VACANCY` | DBEDT; FRED |
| Disposition cost | 3.5% of sale | `FINANCING.disposition_cost` | |
| Cap rate | 5.1%, all zones; retail +1.5 points; exit +0.5 points | `DEFAULT_CAP_RATE`, `RETAIL_CAP_SPREAD`, `HOLD.exit_cap_spread` | |
| Target ROC spread | +1.5 points over cap rate | `DEFAULT_TARGET_SPREAD` | |
| Land carry / target return | 11.5% | `DEFAULT_TARGET_RETURN` | |
| Developer profit (RLV) | 15% of project cost | `DEFAULT_PROFIT_MARGIN` | |
| Affordable rent | 30% of HHFDC income limit | `AFFORDABLE_RENT_INCOME_SHARE`, `INCOME_LIMITS` | 2025 HHFDC Honolulu County schedule (median $129,300) |
| Lease-up absorption | 20 units/month to 90% occupancy, + 3 months | `TIMING` | Used when `leaseup_months` isn't set |
| Bill 7 grants | $50,000 per unit pre-construction; $12,000 per affordable unit post-construction ($40/sqft under 300 sqft) | `GRANT`, `postGrantPerUnit`, `mixPostGrant` | ROH §§ 32-4.2, 32-5.2 |
| Prevailing wage premium | 45% on on-site labor, which is 25% of hard cost (11.25% of hard cost) | `PREVAILING_WAGE` | Median of 11 trades: DLIR Bulletin 509 (basic + fringe) vs BLS OEWS Honolulu median wage + 30% benefits. Docs suggest testing 35–55%. Labor share from the 2022 Economic Census (22–27%) |
| Water system facilities charge | per fixture unit, by stories | `WSFC_PER_FU` | BWS schedule |

## Model scope

- Oʻahu only (City and County of Honolulu zoning, fees, rents and costs).
- A first-pass feasibility screen, not an appraisal or a building design.
- 100% rental. For-sale is not active in v0.3.0.
- One pathway, one lot, one developer per run: no stacking Bill 7 with 201H, no lot assembly.
- The only public funding modeled is the Bill 7 grants (no LIHTC, RHRF, HMMF or DURF).

## Documentation of sources

The repo's `docs/` folder (https://github.com/UHERO/feasim/tree/main/docs) gives the source for each value: `variables.md` for every constant, `simulator-logic.md` for the method, and the pathway pages for the law behind each rule. It is not in the npm package, so link to GitHub. Point users there for citations rather than guessing.
