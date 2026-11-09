# Pathways (v0.3.0)

**Source:** `@uhero/feasim` v0.3.0 (https://github.com/UHERO/feasim, tag `v0.3.0`). **Last verified: November 9, 2026.**

Envelope rules are in `src/parcels.ts` (`pathwayEnvelope`). The pro forma rules are in `src/calculations/calc-<program>.ts`. The ids are historical: `"bill6"` is current zoning and `"standard"` is the rules before Ord. 26-7. Each pathway has a methodology page with its legal basis at `docs/pathways/<id>.md` in the repo (`h201.md` for 201H, plus `special-districts.md` for the Waikiki precincts).

| | `bill6` (current zoning) | `standard` (pre-Ord. 26-7) | `bill7` (ROH ch. 32) | `h201` (HRS § 201H) | `tod` (ROH § 21-9.100) |
| --- | --- | --- | --- | --- | --- |
| FAR | `calcFarBill6`: Ord. 26-7 apartment FARs (low density 2.0, A-2/AMX-2 3.0, A-3/AMX-3/Waikiki 4.0); business zones keep Table 21-3.4 (B-1 1.0, B-2/BMX-3 2.5, BMX-4 4.0) | `calcFAR`: the old lot-size formulas | 4.0, but the building stops at its unit cap (`lot × 4.0 ÷ 800` units) | Same as `bill6` | Tier-based (see below) |
| Open-space bonus | B-2, BMX-3 (to 3.5), BMX-4 (to 7.5) | Same | No | Same as `bill6` | IPD-T only |
| Height | Parcel height (default 30 ft if unknown) | Same | min(parcel height, 60 ft); a `height_limit` lever can go higher | Parcel height (30 ft if unknown in the code; `docs/pathways/h201.md` says 60) | Base, or a doubled / plan / PD-T height by tier |
| Height step-backs | By zone (A-2, A-3, AMX-2, AMX-3, B-2, BMX-3, BMX-4, IMX-1 above 40 ft; Resort above 30 ft; Waikiki incl. front yard) | Same | **Code applies the zone's rule** (see note) | Same as `bill6` | Underlying zone's rule |
| Lot coverage | 60% in apartment/AMX (Ord. 26-7 flat), 100% business | Apartment/AMX tiered by lot size (60/50/40%), 100% business | 80% (Waikiki open-space rule can bind) | Same as `bill6` | Underlying zone, by zone class |
| Car parking | Article 6 minimum is the legal floor; default lever 0.95 stalls/unit | Same | None required (ROH 32-2.3); default lever 0.5 | Same as `bill6` | None required; default lever 0.85 |
| Loading / bicycle | Loading on lots > 7,500 sqft; bicycle parking required | Same | None | Same as `bill6` | Loading required; bicycle except industrial |
| Affordable units | 0 | 0 | `ceil(0.8 × units)`, rental | `ceil(affordability × units)` (≥ 51%), rental | 0 |
| Fees | Waivable fees on market units | Same | Waived (`"none"`); park dedication waived | Waivable fees on market units; impact fee 0 | Waivable fees on market units, plus a $15,000 TOD application and the special district permit fee, at every tier |
| Property tax | Full from year 1 | Full | Improvements exempt for 15 years if `affordable_ami` ≤ 80% | Full | Improvements exempt for 15 years |
| Grants | None | None | Pre- or post-construction (below) | None | None |
| Opex default | 30% of EGI | 30% | 19% (GET-exempt) | 19% | 30% |
| Hold / perm rate / DSCR | 8 yr / 6.75% / 1.20 | Same | 10 yr / 7.0% / 1.15 | Same as `bill6` | Same as `bill6` |

**Bill 7 step-backs (docs/code mismatch):** `docs/pathways/bill7.md` says Chapter 32 sets "Height setbacks: None" and the model applies none. The v0.3.0 code (`calc-bill7.ts`, `stepBackOf(sliders, zoneStepBack(zone_class))`) does apply the zone's step-back. At the 60 ft default this narrows the storeys above 40 ft by 1–2 ft per side in step-back zones, which rarely binds. It matters more when a height lever goes above 60 ft: a 15,000 sqft A-2 lot at 150 ft with FAR not binding loses about 14% of its floor area. Report what the code does, and mention the docs say otherwise if it's relevant.

**Header comments are stale.** Some file headers describe rules the code no longer applies: "5% affordable rental, 15% affordable sale" and "Zone-based permitting months" in `calc-standard.ts`, "Mandatory 15% affordable rental + 30% affordable sale" in `calc-tod.ts`. In v0.3.0 those pathways set the affordable units to 0, because Chapter 29 is triggered by for-sale development and for-sale is off, and timing is 18/12 months everywhere.

**Parking exemption:** Standard, pre-Ord. 26-7 and 201H read `parking_exempt`. Because the default `parking_ratio` is a flat 0.95 that also applies on exempt parcels, the exemption only removes retail stalls at the defaults. Its full effect shows only when `parking_ratio` is left unset in `calcSingle` (legal minimum, 0 on exempt parcels).

## Bill 7 grants (v0.3.0 behavior)

A project takes one grant, set by `grant_type` (ROH § 32-5.2(c)(1) bars both). Full legal basis and the prevailing-wage derivation are in `docs/pathways/bill7.md`.

| | `post_construct` (default) | `pre_construct` |
| --- | --- | --- |
| Law | ROH § 32-4.2 | ROH § 32-5.2 (Ord. 25-19) |
| Amount | $12,000 per affordable unit; $40/sqft for units under 300 sqft (`mixPostGrant`). No preset mix triggers the small-unit rule, since the studio is exactly 300 sqft | $50,000 per unit |
| Units paid | Affordable units (80%, rounded up) | All units by default; `grant_units: "affordable"` for affordable only |
| When paid | Lump sum in the month construction ends | Evenly over the construction months |
| Prevailing wages | None | Required. Hard cost rises by `onsite_labor_share × prevailing_wage_premium` = 25% × 45% = **11.25%** of market-wage hard cost (`pw_premium_cost`) |
| Cap | None | Subsidy ≤ on-site labor at prevailing wages = 25% × 1.45 = 36.25% of market-wage hard cost (`subsidy_labor_cap`; `subsidy_cap_binds`). Rarely binds |

How the grants enter the results:

- **Financing:** neither grant displaces equity or the loan. Full equity goes in at closing, and grants come back as developer cash in the month they're paid.
- **RLV:** each payment is discounted from its month at `target_return` (`rlv_grants_pv`).
- **IRR and equity multiple:** grants are in the monthly cash flows (flip and hold).
- **ROC:** both grants come off the cost basis at face value (`dev_capital_subsidy`), undiscounted.
- **Property tax:** assessed on the market-wage hard cost, so the wage premium doesn't raise the tax.

Worked example from the docs (Project View defaults, 5,000 sqft lot, 10 units, 8 affordable): post-construction ROC 5.92%, RLV $89/sqft, IRR 10.6%, marginal. Pre-construction with the 45% premium: ROC 5.98%, RLV $94/sqft, IRR 10.7%, marginal. Without the premium it would be strong. Most of the pre-construction subsidy's advantage goes to wages.

**Not modeled:** stacking Bill 7 with 201H, the $10M program fund cap, the 24-month certificate-of-occupancy deadline, certified-payroll/escrow compliance costs, and overtime.

## TOD

Check eligibility first: `todIneligibleReason(parcel)` returns `null` when the lot qualifies. Eligibility needs `tod_regime`, an eligible district, a lot of at least 20,000 sqft (PD-T minor and major permits have no minimum), and `slu_urban`. An ineligible lot still runs, on its base zoning envelope.

Industrial zones may also return `todNotApplicableReason(parcel, tier)`. Results with `status` `"not_modeled"` or `"not_applicable"` should be left out of totals.

| Tier (`opts.tier`) | PD-T (inside a TOD special district) | IPD-T (elsewhere) |
| --- | --- | --- |
| `"base"` | Underlying zoning, no open-space bonus | Underlying zoning + open-space bonus |
| `"minor_sd"`, `"major_sd"` | `calcFarPdt` FAR ladder; `todHeightPdt` height | Same as base (PD-T only tiers) |
| `"with_benefits"` | PD-T approval: ladder FAR; map bonus height if given | `calcFarTod` formula (or the station-area plan FAR if higher); plan height or 2× zoned height, up to 450 ft |

- **Every tier above base is a best case:** the community benefits that buy the bonus are not priced.
- **Front strip:** PD-T lots get a 40 ft front strip with no ground-floor parking or loading.
- **IMX-1 cap:** under PD-T, IMX-1 is capped at 10 units.
- **Parking:** none is legally required on any TOD parcel; the 0.85 default is an assumption.
- **`calcSingle` with `"tod"`:** pass `max_far` (`calcParcel` always does).
