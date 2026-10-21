# Fixed assumptions (v0.1.1)

These are constants in `src/calculations/calc-shared.ts` (and `DEFAULT_INPUTS` in `src/parcels.ts`). Users can't change the fixed ones without editing the code.

| Constant | Value | Where | Basis (from code comments / docs) |
| --- | --- | --- | --- |
| Building efficiency | 80% | `EFFICIENCY` | Net rentable ÷ gross |
| Unit sizes | studio 300, 1BR 500, 2BR 700, 3BR 900, 4BR 1,200 sqft | `UNIT_SIZE` | |
| Hard cost contingency | 10% | `FINANCING.contingency` | |
| Construction cost escalation | 5.6%/yr to the construction midpoint | `FINANCING.cost_inflation` | |
| Retail cost premium | +$15/sqft | `RETAIL_PREMIUM` | |
| Construction management | 3% of hard cost | `CONSTRUCTION_MGMT_RATE` | |
| Structured parking | $190/sqft, 330 sqft/stall | `PARKING` | RLB Q2 2026 Honolulu (low end); Ulupono 2020 |
| Surface parking | $30/sqft, 360 sqft/stall | `PARKING` | Ulupono 2020, adjusted to 2026 |
| Parking income | $100/stall/month | `PARKING.monthly_rent_per_stall` | |
| Construction loan rate | 8% | `FINANCING.construction_rate` | |
| Loan fee | 0.75% | `FINANCING.loan_fee` | |
| Pre-development spend | 10% of construction cost per year of permitting | `FINANCING.cost_carried` | |
| AMI / rent escalation | 4%/yr | `FINANCING.ami_escalation` | 2015–2026 Honolulu MFI CAGR |
| Hold rent growth | 3.5%/yr (affordable 4%, market 3.2% blended) | `HOLD.annual_rent_growth` | Market: Apartments.com Honolulu YoY |
| Hold opex growth | 3.5%/yr | `HOLD.annual_opex_growth` | |
| Assessment growth | 2%/yr | `HOLD.annual_assessment_growth` | |
| Property tax | 0.35% | `FINANCING.rpt_residential` | Honolulu residential rate |
| Tax exemption | 15 years on improvements | `HOLD.tax_exemption_years` | Bill 7 (≤ 80% AMI), TOD |
| Vacancy | 7.4% residential, 12.6% retail | `VACANCY` | |
| Disposition cost | 3.5% of sale | `FINANCING.disposition_cost` | |
| Cap rate | 5.1%, all zones; retail +1.5 points; exit +0.5 points | `DEFAULT_CAP_RATE`, `RETAIL_CAP_SPREAD`, `HOLD.exit_cap_spread` | |
| Target ROC spread | +1.5 points over cap rate | `DEFAULT_TARGET_SPREAD` | |
| Land carry / target return | 11.5% | `DEFAULT_TARGET_RETURN` | |
| Developer profit (RLV) | 15% of project cost | `DEFAULT_PROFIT_MARGIN` | |
| Affordable rent | 30% of HHFDC income limit | `AFFORDABLE_RENT_INCOME_SHARE`, `INCOME_LIMITS` | HHFDC guideline |
| Lease-up absorption | 20 units/month to 90% occupancy, + 3 months | `TIMING` | Used when `leaseup_months` isn't set |
| Bill 7 grants | $50,000 pre / $12,000 post per affordable unit | `GRANT` | ROH ch. 32 |
| Water system facilities charge | per fixture unit, by stories | `WSFC_PER_FU` | BWS schedule |

## Model scope

- Oʻahu only (City and County of Honolulu zoning, fees, rents and costs).
- A first-pass feasibility screen, not an appraisal or a building design.
- 100% rental. For-sale is not active in v0.1.1.

## Documentation of sources

The package README links to UHERO's Policy Simulator documentation and variable spreadsheets, which give the source for each value. Point users there for citations rather than guessing.
