# feasim-skill

A [Claude Code plugin](https://code.claude.com/docs/en/plugins.md) that teaches Claude how UHERO's housing feasibility model, [`@uhero/feasim`](https://github.com/UHERO/feasim), works.

Once it's installed, you can ask Claude plain-language questions about the model, for example why a lot does or doesn't pencil under Bill 7. You don't need to read the code yourself.

**Describes `@uhero/feasim` v0.1.1.**

This is an unofficial community resource. It is not maintained by UHERO.

## Plugins

| Plugin | Description |
|--------|-------------|
| [`feasim`](feasim/) | Explains and runs UHERO's `@uhero/feasim` feasibility model for Oʻahu housing: its inputs, results and policy pathways (current zoning, pre-Ord. 26-7, Bill 7, 201H, TOD), and why a parcel does or doesn't pencil. |

## Install

### Option 1: Marketplace (recommended)

This installs the plugin once. It then stays installed across sessions.

1. Open Claude Code. Use the terminal app, the desktop app, or the VS Code extension.
2. Type this and press Enter:

   ```
   /plugin marketplace add https://github.com/sakuraet/feasim-skill.git
   ```

3. Type `/plugin` and press Enter.
4. Find the **feasim-skills** marketplace and install the **feasim** plugin.

You can also install it in one step:

```
/plugin install feasim@feasim-skills
```

The repo is public, so you don't need a GitHub login.

**To get updates later**, run:

```
/plugin marketplace update feasim-skills
```

### Option 2: Load it for one session from a local copy

```bash
git clone https://github.com/sakuraet/feasim-skill.git

# load the plugin for this session only
claude --plugin-dir ./feasim-skill/feasim
```

This doesn't install anything permanently. You need the `--plugin-dir` flag each time.

### Option 3: claude.ai (no Claude Code)

1. Download this repo as a zip from GitHub (**Code → Download ZIP**) and unzip it.
2. Zip just the `feasim/skills/feasim` folder.
3. On claude.ai, go to **Settings → Capabilities → Skills** and upload that zip.

In claude.ai, Claude can explain the model and write scripts for you, but it can't run the model itself.

### Prerequisites

- [Claude Code](https://code.claude.com/docs) installed and signed in (Options 1 and 2), or a claude.ai account (Option 3).
- Optional: [Node.js](https://nodejs.org) 18 or newer. With Node installed, Claude can run the model and report real numbers. Without it, Claude explains the model and gives you a script to run.

## How to use it

You don't need to name the skill. Mention the model and Claude loads it on its own. For example:

```
"Using feasim, why is a 20,000 sqft BMX-3 lot poor under Bill 7?"
"Run a 10,000 sqft A-2 lot with a 60 ft height limit under every pathway in the feasibility model"
"In feasim, what does rlv_sqft mean, and how is it calculated?"
"What does the policy simulator assume for construction cost and interest rates?"
"How does the TOD pathway in feasim change FAR and height?"
```

The skill only switches on when you refer to the model ("feasim", "the policy simulator", "the feasibility model", or one of its functions or results). General Hawaii housing questions, like home prices or rents, go to UHERO's data tools instead.

## Plugins in Detail

### feasim

Claude reads the model's documentation, which is bundled with the skill, and when it can, it runs the model itself.

What Claude can do with it:

- **Explain the model:** the inputs, their defaults for each pathway, what each result means, and how the verdict (strong / marginal / poor) is scored.
- **Run scenarios:** compare pathways for one lot, or test how land cost, construction cost, interest rates or a grant change the answer.
- **Trace a result:** follow residual land value and return on cost back to the cost, rent or financing step that drives them.
- **Help developers:** call `calcParcel` and `calcSingle` correctly, and avoid the model's known gotchas.

Guardrails built in:

- **Read-only:** Claude never edits, commits to or publishes the feasim package. It sends change requests to the UHERO/feasim repo.
- **No made-up numbers:** figures for a specific lot come from running the model. Hand calculations are labeled as such.
- **Not an appraisal:** results are presented as a first-pass screen, not a property valuation or UHERO's official position.
- **Version check:** Claude checks which feasim version you have. If it isn't v0.1.1, Claude says so and relies on the package's own docs.
- **Caveats:** every answer states the version, the pathway, whether the run is a hold or a flip, and any assumptions you changed.

What's inside:

```
feasim/
├── .claude-plugin/
│   └── plugin.json            ← plugin manifest
└── skills/
    └── feasim/
        ├── SKILL.md           ← when to use it, rules, workflow, answer format
        └── references/
            ├── inputs.md      ← every input, defaults per pathway, valid values
            ├── outputs.md     ← every result, how they fit together, the verdict
            ├── pathways.md    ← what each pathway changes; Bill 7 grants; TOD tiers
            ├── model-flow.md  ← step-by-step formulas, with the source function for each
            └── assumptions.md ← fixed constants and where they come from
```

## Updating for a new feasim release

1. Compare the releases in the feasim repo, e.g. `git diff v0.1.1 v0.2.0 -- src README.md`.
2. Update the affected files in `references/`, plus the version and "Last verified" lines in `SKILL.md` and each reference file.
3. Bump `version` in `feasim/.claude-plugin/plugin.json` and in `.claude-plugin/marketplace.json`.
4. Test locally with `claude --plugin-dir ./feasim`, and check the manifests with `claude plugin validate .`.

## License

MIT. Parts of the content are adapted from the `@uhero/feasim` README, © 2026 UHERO, under the MIT License.
