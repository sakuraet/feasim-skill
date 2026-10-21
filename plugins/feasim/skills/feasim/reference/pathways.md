# Pathways (v0.1.1)

Envelope rules are in `src/parcels.ts` (`pathwayEnvelope`). The pro forma rules are in `src/calculations/calc-<program>.ts`. The ids are historical: `"bill6"` is current zoning and `"standard"` is the rules before Ord. 26-7.

| | `bill6` (current zoning) | `standard` (pre-Ord. 26-7) | `bill7` (ROH ch. 32) | `h201` (HRS § 201H) | `tod` (ROH § 21-9.100) |
| --- | --- | --- | --- | --- | --- |
| FAR | `calcFarBill6`: Ord. 26-7 apartment FARs (low density 2.0, A-2/AMX-2 3.0, …); business zones keep Table 21-3.4 | `calcFAR`: the old lot-size formulas | 4.0 | Same as `bill6` | Tier-based (see below) |
| Open-space bonus | Yes, where the zone offers it | Yes | No | Yes | IPD-T only |
| Height | Parcel height (default 30 ft if unknown) | Same | min(parcel height, 60 ft) | Same as `bill6` | Base, or a doubled / plan / PD-T height by tier |
| Lot coverage | By zone | By zone | 80% (Waikiki open-space rule can bind) | By zone | By zone |
| Car parking | Article 6 minimum, unless `parking_exempt` | Same | None (ROH 32-2.3) | Same as `bill6` | None (bicycle parking only) |
| Affordable units | 0 | 0 | 80% of units, rental | `affordability` (≥ 51%), rental | 0 |
| Fees | Waivable fees on market units | Same | Waived (`"none"`); park dedication waived | Waivable fees on market units; impact fee 0 | Waivable fees on market units, plus a $15,000 TOD application and special district permit fees |
| Property tax | Full from year 1 | Full | Improvements exempt for 15 years if `affordable_ami` ≤ 80% | Full | Improvements exempt for 15 years |
| Grants | None | None | Pre- or post-construction (below) | None | None |
| Opex default | 30% of EGI | 30% | 19% (GET-exempt) | 19% | 30% |

**Header comments are stale.** Some file headers (e.g. "5% affordable rental, 15% affordable sale" in `calc-standard.ts`, or "15% affordable rental + 30% affordable sale" in `calc-tod.ts`) describe rules the code no longer applies. In v0.1.1 those pathways set the affordable units to 0.

## Bill 7 grants (v0.1.1 behavior)

Both grant types pay `grant_amount × affordable_units` (80% of units).

- **`post_construct`** (default; $12,000 in `DEFAULT_INPUTS`):
  - **RLV:** added to the residual at stabilization, then discounted with it.
  - **Flip:** added to the exit proceeds.
  - **Hold:** added to year-1 cash flow.
- **`pre_construct`** ($50,000 internal fallback):
  - **Financing:** treated as a source at loan closing that displaces equity.
  - **RLV:** added at face value after discounting (as cash at t = 0).
- **ROC:** both grants are netted out of the cost basis (`dev_capital_subsidy`).
- **Not modeled in v0.1.1:** prevailing wage requirements and the timing of grant payments.

## TOD

Check eligibility first: `todIneligibleReason(parcel)` returns `null` when the lot qualifies. Eligibility needs `tod_regime`, an eligible district, a lot of at least 20,000 sqft, and `slu_urban`.

Industrial zones may also return `todNotApplicableReason(parcel, tier)`. Results with `status` `"not_modeled"` or `"not_applicable"` should be left out of totals.

| Tier (`opts.tier`) | PD-T (inside a TOD special district) | IPD-T (elsewhere) |
| --- | --- | --- |
| `"base"` | Underlying zoning, no open-space bonus | Underlying zoning + open-space bonus |
| `"minor_sd"`, `"major_sd"` | `calcFarPdt` FAR ladder; `todHeightPdt` height | Same as base |
| `"with_benefits"` | PD-T approval: ladder FAR; map bonus height if given | `calcFarTod` formula (or the station-area plan FAR if higher); plan height or 2× zoned height, up to 450 ft |

- **Front strip:** PD-T lots get a 40 ft front strip with no ground-floor parking.
- **IMX-1 cap:** under PD-T, IMX-1 is capped at 10 units.
- **`calcSingle` with `"tod"`:** pass `max_far` (`calcParcel` always does).
