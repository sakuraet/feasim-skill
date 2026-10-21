---
name: feasim
description: Explains how the @uhero/feasim housing development feasibility package works, for developers calling it and for readers interpreting its results. Use when working with @uhero/feasim, calcParcel, calcSingle, parcelInputs, pathwayEnvelope, DEFAULT_INPUTS, overallLevel, or ParcelOutput fields; when asking why a parcel is or isn't feasible, what drives residual land value (RLV), return on cost (ROC), IRR or equity multiple; or when comparing Oahu policy pathways (current zoning, Bill 7, 201H, TOD).
---

# @uhero/feasim

`@uhero/feasim` is UHERO's development feasibility model for housing on Oʻahu. Given a lot and a policy pathway, it works out what a developer could build, runs the pro forma (units, parking, costs, rents, financing, residual land value, returns) and gives a verdict: strong, marginal or poor.

**This skill describes v0.1.1.** Check the installed version (`node_modules/@uhero/feasim/package.json`). If it differs, trust the package's README and code over this skill. Grant and timing logic are known to change after 0.1.1.

## Where to look

| What | Where |
| --- | --- |
| README (concepts, inputs, results, examples) | `node_modules/@uhero/feasim/README.md`, or https://github.com/UHERO/feasim |
| Types: `SliderInputs`, `ParcelOutput`, `Parcel` | `node_modules/@uhero/feasim/dist/index.d.ts` |
| Built code (unminified) | `node_modules/@uhero/feasim/dist/index.js` |
| Source | https://github.com/UHERO/feasim, tag `v0.1.1`: `src/calculations/calc-*.ts` (one file per pathway), `calc-shared.ts` (constants and shared finance), `src/parcels.ts` (envelopes, defaults), `src/feasibility.ts` (verdict) |

The npm package ships only `dist`, the README and the license. Read the GitHub source when you need the commented code.

## The two ways in

```ts
import { calcParcel, calcSingle, DEFAULT_INPUTS, overallLevel } from "@uhero/feasim";

// a real parcel under a pathway: envelope + defaults + your levers
const r = calcParcel("bill7", { lot_sqft: 10000, zone_class: "A-2", height: 60 }, { land_cost: 200 });

// the raw model: one flat set of inputs, you choose FAR and height
const s = calcSingle({ ...DEFAULT_INPUTS.bill7, lot_sqft: 10000, zone_class: "A-2", height_limit: 60, max_far: 4 });

overallLevel(r); // "strong" | "marginal" | "poor"
```

- `calcParcel(program, parcel, levers?, opts?)` = `calcSingle(parcelInputs(...))`. `parcelInputs` merges, in order: `DEFAULT_INPUTS[program]`, then the levers you set, then the parcel's envelope from `pathwayEnvelope` (height limit and FAR). `levers.height_limit` and `levers.max_far` override the envelope. Bill 7's 60 ft cap always applies.
- `calcSingle(inputs)` runs the model. Anything you leave out falls back to the pathway calculator's own internal defaults, which are **not** always the same as `DEFAULT_INPUTS`. Spread `DEFAULT_INPUTS[program]` first to match `calcParcel`.
- `opts.tier` (TOD only): `"base"`, `"minor_sd"`, `"major_sd"` or `"with_benefits"`.

## How one run works

Every pathway calculator follows the same steps (see `reference/model-flow.md` for the formulas):

1. **Envelope.** Floor plates from height (10 ft floors), lot coverage, setback and step-backs, capped by FAR. Parking goes on a surface lot, the ground floor or podium levels. Units = residential floor area ÷ average unit size, after 80% efficiency.
2. **Units.** The affordable share depends on the pathway (Bill 7: 80%; 201H: `affordability`, 51% by default; current zoning, pre-Ord. 26-7 and TOD: 0 in v0.1.1).
3. **Hard cost.** Building sqft × `hard_cost_per_sqft` (retail +$15), plus parking, escalated 5.6%/yr to the construction midpoint.
4. **Soft cost.** Permits, plan review, wastewater, BWS water charges, park dedication, grading, 3% construction management, etc. Waivers follow `waivable_fee_basis`.
5. **Financing.** Construction loan at 8% on an S-curve draw, 0.75% fee, lease-up carry. Loan = the lower of loan-to-cost (1 − `equity`) and, for holds, the DSCR take-out.
6. **Revenue and NOI.** Rents from HHFDC income limits (30% of income), escalated 4%/yr. Vacancy 7.4%. Opex = `opex_ratio` × EGI. Property tax 0.35%.
7. **Value.** GDV = NOI ÷ cap rate (5.1%; retail +1.5 points).
8. **RLV.** The most a developer could pay for the land and still earn the target return.
9. **Returns.** A flip (`hold_years: 0`) or a hold with a permanent loan, yearly cash flows and an exit sale.

## Key identities (v0.1.1)

```
dev_cost (total_dev_cost) = hard_cost × 1.10 contingency + soft_cost
rlv_project_cost          = dev_cost + rlv_financing_cost      (loan sized without land)
rlv_profit                = rlv_project_cost × profit_margin   (15%)
residual                  = gdv − rlv_selling_costs (3.5%) + rlv_post_grant − rlv_project_cost − rlv_profit
rlv                       = residual / (1 + target_return)^(total_months/12) + pre_construction_subsidy
rlv_sqft                  = rlv / lot_sqft
dev_roc                   = noi / (dev_total_dev_cost − dev_capital_subsidy)
dev_target_roc            = cap_rate + target_spread           (5.1% + 1.5% = 6.6%)
```

`dev_total_dev_cost` = dev_cost + land + `total_financing_cost` (the deal loan, sized with land). RLV and ROC use different loans on purpose.

## Answering "why" questions

- **Is it feasible?** `overallLevel` scores ROC vs target, RLV ÷ land price (`land_cost`), equity multiple and levered IRR. See `reference/outputs.md` for the bands.
- **Why is RLV negative?** The residual is negative: value after selling costs doesn't cover cost plus 15% profit. Compare `gdv` with `rlv_project_cost + rlv_profit`.
- **Why did a pathway build nothing?** Check `max_units`. A misspelled `zone_class` (e.g. `"a-2"`) has no FAR. A TOD lot may be ineligible: call `todIneligibleReason(parcel)`.
- **Why do two pathways differ?** Compare envelope (`buildable_sqft`, `max_units`), parking (`parking_stalls`, `podium_levels`), affordable share, fees (`soft_cost_breakdown`), tax treatment, and grants. See `reference/pathways.md`.

When unsure, **run the model** with the user's inputs and read the output fields rather than reasoning from memory.

## Gotchas

- **No input validation.** An unknown `program` runs `"bill6"`. Wrong-case or unknown zones build nothing. Unknown `unit_type` is treated as all one-bedroom. `"ami_85"` is scaled from the 50% row. 201H with `affordability` below 0.51 throws. `permanent_rate` or `amort_years` of 0 gives `NaN` returns.
- **Historical ids:** `"bill6"` is current zoning (Ord. 26-7); `"standard"` is the rules before Ord. 26-7.
- **Units:** rates are fractions (`0.058` = 5.8%). `land_cost` and `rlv_sqft` are $ per sqft of **lot**. `hard_cost_per_sqft` is $ per sqft of **building**. `retail_rent` is $/sqft/month.
- **Open-space bonus:** where a zone offers one, `calcSingle` runs with and without it and keeps the higher RLV. `open_space_bonus` on the result is what was taken.
- **Code comments can be stale.** Some file headers describe affordable shares the code no longer uses. The code is authoritative.

## Reference files

Read only what the question needs:

- `reference/inputs.md`: every input, its default per pathway, units and valid values
- `reference/outputs.md`: output fields by group, and the verdict bands
- `reference/pathways.md`: what each pathway changes (FAR, height, parking, affordability, fees, tax, grants)
- `reference/model-flow.md`: step-by-step formulas and constants, with the source function for each
- `reference/assumptions.md`: the fixed constants and where they come from
