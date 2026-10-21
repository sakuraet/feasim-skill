---
name: feasim
description: >-
  Explain and run UHERO's @uhero/feasim housing development feasibility model
  (v0.1.1) for Oʻahu: its inputs, outputs, policy pathways (current zoning,
  pre-Ord. 26-7, Bill 7, 201H, TOD), and why a parcel does or doesn't pencil.
  Use this skill ONLY when the user references the model, by saying "feasim",
  "@uhero/feasim", "the policy simulator", "the feasibility model", or one of
  its functions or fields (calcParcel, calcSingle, parcelInputs,
  DEFAULT_INPUTS, overallLevel, rlv, dev_roc, ParcelOutput), or asks what the
  model says about a lot or pathway. Do NOT trigger on general Hawaii housing
  or real estate questions that don't name the model (housing prices, sales,
  rents, permits); the model holds no observed market data.
---

# @uhero/feasim

`@uhero/feasim` is UHERO's development feasibility model for housing on Oʻahu. Given a lot and a policy pathway, it works out what a developer could build, runs the pro forma (units, parking, costs, rents, financing, residual land value, returns) and gives a verdict: strong, marginal or poor.

**Scope note:** This skill is for questions about the model: how it works, what it assumes, and what it returns. It is not a source of real market data. For observed prices, sales or rents, say the model can't answer that rather than quoting its assumptions as market figures.

Typical questions:

- "Why is this parcel poor under Bill 7?"
- "What does `rlv_sqft` mean?"
- "How does the TOD pathway change FAR?"
- "Run a 20,000 sqft BMX-3 lot under every pathway."
- "What does the model assume for construction cost?"

## Version check (do this first)

This skill describes **v0.1.1**. If the user's project has the package installed, read the version in `node_modules/@uhero/feasim/package.json`.

If it isn't 0.1.1:

- Tell the user the version differs.
- Trust the package's README and code over this skill.
- Grant timing and Bill 7 grant logic are known to change after 0.1.1.

## Reference files

Read only the files the question needs:

| File | Covers |
| --- | --- |
| `references/inputs.md` | Every input, its default per pathway, units, valid values |
| `references/outputs.md` | Output fields by group, formulas that tie them together, the verdict bands |
| `references/pathways.md` | What each pathway changes (FAR, height, parking, affordability, fees, tax, grants), TOD tiers |
| `references/model-flow.md` | Step-by-step formulas, each with its source function |
| `references/assumptions.md` | Fixed constants and their basis |

Primary sources, when the references aren't enough:

- **Package README:** `node_modules/@uhero/feasim/README.md`, or https://github.com/UHERO/feasim
- **Types:** `node_modules/@uhero/feasim/dist/index.d.ts`
- **Commented source:** https://github.com/UHERO/feasim at tag `v0.1.1`

The npm package ships only `dist`, the README and the license.

## How to operate

**If you can run shell commands and Node.js 18+ is available:**

1. Run the model rather than reasoning from memory.
2. Install the package in a scratch folder, not in the user's project unless they ask: `npm install @uhero/feasim@0.1.1`.
3. Write a short `.mjs` script and run it with `node`.
4. Report the numbers it returns.

**If you cannot run commands** (claude.ai chat, no Node):

1. Explain from the reference files.
2. Give the user a ready-to-run script, and say they need Node.js to run it.
3. Never present an estimate as model output.

```js
// run.mjs: compare pathways for one lot
import { calcParcel, overallLevel } from "@uhero/feasim";

const parcel = { lot_sqft: 20000, zone_class: "BMX-3", height: 150 };
for (const program of ["bill6", "standard", "bill7", "h201", "tod"]) {
  const r = calcParcel(program, parcel);
  console.log(program, r.max_units, r.dev_roc, r.rlv_sqft, overallLevel(r));
}
```

## Hard rules

- **Never modify the model.** Don't edit files in `node_modules/@uhero/feasim` or in the UHERO/feasim repo. Don't publish, commit to or open pull requests against it. If the user wants the model changed, point them to the UHERO/feasim repo and its maintainers.
- **Never invent numbers.** Any figure about a specific lot or scenario must come from running the model, or from output the user supplied. If you calculate something by hand (for example, rebuilding RLV from its parts), say so.
- **Not an appraisal, not a UHERO finding.** The model is a first-pass screen under stated assumptions. Don't present its results as a property valuation, a forecast, or UHERO's official position on a policy.
- **Don't guess inputs silently.** The model doesn't validate inputs: a wrong-case zone builds nothing, and an unknown pathway runs current zoning. When a lot's zone, height or pathway is unclear, ask, or state the assumption you made.

## Workflow for "why" questions

1. **Pin down the run.** Identify the pathway, the parcel (lot size, zone, height) and any levers changed from the defaults. Use `calcParcel` for real parcels and `calcSingle` only when the user sets FAR and height themselves.
2. **Check the envelope first.** If `max_units` is 0, the cause is usually the zone code or TOD eligibility (`todIneligibleReason`), not the finances.
3. **Follow the chain.** The verdict comes from:
   - return on cost vs its target;
   - RLV ÷ land price;
   - equity multiple;
   - IRR.

   RLV comes from GDV minus selling costs, project cost and profit, discounted for time. Find which link moves the answer (see `references/outputs.md`).
4. **Compare like with like.** When comparing pathways, compare:
   - building size (`buildable_sqft`, `max_units`);
   - parking (`parking_stalls`, `podium_levels`);
   - affordable share;
   - fees (`soft_cost_breakdown`);
   - tax treatment;
   - grants.

   See `references/pathways.md`.
5. **Caveat the answer** (below).

## Caveat the answer

State alongside the numbers:

- the package version;
- the pathway, and whether the run was a hold (`hold_years > 0`) or a flip;
- every lever set away from `DEFAULT_INPUTS`;
- that results are rounded and the model is Oʻahu-only.

Also flag known limits that affect the question:

- For-sale units aren't active.
- In v0.1.1, current zoning, pre-Ord. 26-7 and TOD carry no affordable units.
- Some code comments describe rules the code no longer applies. The code is authoritative.

## Gotchas

- **Two entry points.**
  - `calcParcel(program, parcel, levers?, opts?)` merges `DEFAULT_INPUTS[program]`, then your levers, then the parcel's envelope.
  - `calcSingle(inputs)` uses each calculator's internal fallbacks for anything left out, and these differ from `DEFAULT_INPUTS`. Spread `DEFAULT_INPUTS[program]` first to match `calcParcel`.
- **Historical ids:** `"bill6"` is current zoning (Ord. 26-7); `"standard"` is the rules before Ord. 26-7.
- **Units:** rates are fractions (`0.058` = 5.8%). `land_cost` and `rlv_sqft` are $ per sqft of **lot**. `hard_cost_per_sqft` is $ per sqft of **building**.
- **RLV differs for hold and flip:** a hold caps the RLV loan by the DSCR take-out.
- **Open-space bonus:** where a zone offers one, `calcSingle` keeps whichever version (with or without the bonus) has the higher RLV.

## Answer format

- **Bottom line:** one or two sentences answering the question.
- **The numbers:** the key outputs, with units (e.g. units, ROC vs target, RLV per sqft vs land price, verdict).
- **What drives it:** the one or two links in the chain that explain the result.
- **Assumptions:** the version, pathway, hold or flip, and changed levers.

## Maintenance

- **Last verified:** October 21, 2026, against `@uhero/feasim` v0.1.1 (`v0.1.1` tag). Every formula in `references/outputs.md` was checked by running the package.
- **When a new feasim version is released:**
  1. Diff the tags in the feasim repo (`git diff v0.1.1 <new tag> -- src README.md`).
  2. Update the references and the version lines.
  3. Bump `version` in `.claude-plugin/plugin.json` and in the root `marketplace.json`.
