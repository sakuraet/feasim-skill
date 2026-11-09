# Outputs (v0.3.0)

**Source:** `@uhero/feasim` v0.3.0 (https://github.com/UHERO/feasim, tag `v0.3.0`). **Last verified: November 9, 2026.**

Every function returns a `ParcelOutput` (full type in `dist/index.d.ts`). Money is in dollars and rates are fractions. When nothing can be built, the result is all zeros (`max_units: 0`, `rlv_land_carry: 1`).

## By group

| Group | Fields |
| --- | --- |
| Envelope | `height_limit`, `open_space_bonus`, `buildable_sqft` (floor area counted toward FAR; parking, loading, bicycle parking and lobby excluded), `net_residential_sqft`, `net_commercial_sqft` (80% efficiency) |
| Built size | `built_floors`, `built_height`: storeys and feet actually built, parking included. A partly filled top floor counts as a full storey. Can be under `height_limit` when FAR or a unit cap stops the building first |
| Units | `max_units`, `affordable_units`, `afd_rent_units`, `mkt_rent_units`, `afd_sale_units`, `mkt_sale_units` |
| Parking | `parking_stalls` = `surface_stalls` + `ground_floor_stalls` + `podium_stalls`; `podium_levels`, `parking_sqft`, `parking_share`, `loading_spaces`, `bicycle_spaces` (`bicycle_short_term`, `bicycle_long_term`). Bill 7 returns no loading or bicycle fields (none required) |
| Costs | `hard_cost` (escalated, before contingency; includes `pw_premium_cost`), `soft_cost` with `soft_cost_breakdown`, `dev_cost` (= hard × 1.10 + soft), `total_financing_cost` (deal loan), `rlv_financing_cost` (loan without land) |
| Bill 7 grants | `pre_construction_subsidy`, `dev_post_grant`, `dev_capital_subsidy`; pre-construction runs also report `pw_premium_cost` (prevailing wage cost inside `hard_cost`), `subsidy_labor_cap` (most the subsidy may pay) and `subsidy_cap_binds` |
| Financing | `equity_amount`, `dev_equity_pct_realized`, `dev_dscr_binds`, `loan_balance_at_stab`, `total_predev_cost`, `construction_equity` |
| Timing | `permitting_months`, `const_months`, `leaseup_months`, `sales_months`, `total_carry_months`, `total_months` (permitting + construction + carry), `project_years` |
| Income | `pri` (rent), `pgi` (+ parking and admin income), `residential_egi`, `retail_pgi`, `retail_egi`, `egi`, `residential_opex`, `retail_opex`, `operating_expenses`, `property_tax`, `noi` |
| Value | `cap_rate`, `gdv` (NOI ÷ cap rate) |
| RLV | `rlv_selling_costs`, `rlv_project_cost`, `rlv_profit`, `rlv_land_carry`, `rlv_grants_pv`, `rlv`, `rlv_sqft` |
| Returns | `dev_strategy` (`"dev_flip"` / `"hold"`), `dev_hold_years`, `dev_roc`, `dev_target_roc`, `dev_investment_spread` (ROC − cap rate), `dev_is_feasible` (ROC ≥ target), `dev_levered_irr`, `dev_equity_multiple`, `dev_profit`, `dev_roi` |
| Deal | `dev_land_cost`, `dev_total_dev_cost` (dev cost + land + financing), `dev_capital_subsidy`, `dev_net_dev_cost`, `dev_equity_invested`, `dev_equity_out`, `dev_gdv`, `dev_selling_costs`, `dev_loan_payoff`, `dev_post_grant` |
| Hold only | `dev_perm_principal`, `dev_perm_rate`, `dev_annual_debt_svc`, `dev_dscr`, `dev_is_bankable`, `dev_cash_on_cash`, `dev_entry_cap`, `dev_exit_cap` (entry + 0.5 points), `dev_yr1_prop_tax`. These are `null` for flips |

**Removed since v0.1.1:** `rlv_post_grant`. Grants now enter the RLV through `rlv_grants_pv`.

## Identities

```
rlv = (gdv − rlv_selling_costs − rlv_project_cost − rlv_profit) / rlv_land_carry
      + rlv_grants_pv
rlv_land_carry = (1 + target_return)^(total_months / 12)
rlv_sqft = rlv / lot_sqft
dev_roc  = noi / dev_net_dev_cost
dev_net_dev_cost = dev_total_dev_cost − dev_capital_subsidy
```

- RLV rebuilds from its parts to within $1 (rounding).
- `rlv_project_cost` = `dev_cost` + `rlv_financing_cost`. `rlv_profit` = 15% of that.
- `rlv_grants_pv` is each Bill 7 grant payment discounted at `target_return` from the month it is paid: the pre-construction subsidy evenly over the construction months, the post-construction grant when construction ends. It is 0 outside Bill 7.
- `dev_capital_subsidy` is the Bill 7 grant total at face value, undiscounted, so ROC ignores grant timing.
- `rlv` can differ between hold and flip. For a hold, the RLV-basis loan is also capped by the DSCR take-out, which changes `rlv_financing_cost`. A flip has no take-out.
- Flip: `dev_equity_out` = GDV − 3.5% selling − loan payoff, plus any Bill 7 grants received. The equity multiple is `dev_equity_out` ÷ `dev_equity_invested`.
- Hold: `dev_equity_out` = the sum of all positive monthly cash flows, including grants as paid and the reversion. `dev_profit` = that sum − equity.

## The verdict (`src/feasibility.ts`)

| Metric | Poor | Marginal | Strong |
| --- | --- | --- | --- |
| `dev_roc` vs `dev_target_roc` | more than 1 point below | within 1 point | at or above |
| `rlv_sqft ÷ land_cost` (`rlvRatio`) | < 0.9 | 0.9–1.1 | ≥ 1.1 |
| `dev_equity_multiple` | < 2.0 | 2.0–2.5 | ≥ 2.5 |
| `dev_levered_irr` | < 8% | 8–12% | ≥ 12% |

The default target ROC is 6.6% (5.1% cap rate + 1.5 points). Non-finite metrics are skipped. `overallLevel` is:

- **poor** if half or more of the scored metrics are poor;
- **strong** if none are poor and at least 75% are strong;
- **marginal** otherwise.

Per-metric functions: `rocLevel`, `rlvLevel`, `equityMultipleLevel`, `irrLevel`, `scoredLevels`.

**Reading ROC:** it is a ratio, so anything that shrinks cost can raise it. A shorter building can post a higher ROC than a taller one simply because it costs less. Levered IRR is the better metric for timing questions (permitting delay, construction length, lease-up, grant timing).

## Reference run (v0.3.0 defaults)

`calcParcel("bill6", { lot_sqft: 20000, zone_class: "BMX-3", height: 150 })` returns 140 units, `dev_roc` 0.064, `rlv_sqft` 238.45 and `"strong"`, as in the package README. Use it to confirm an install behaves like v0.3.0.
