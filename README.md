# feasim-skill

A [Claude Code](https://claude.com/claude-code) plugin with one skill, `feasim`. The skill explains how [`@uhero/feasim`](https://github.com/UHERO/feasim) works: UHERO's development feasibility model for housing on Oʻahu.

With it installed, Claude can explain inputs, outputs and pathways, trace why a parcel is or isn't feasible, and help you call the package correctly.

**Describes `@uhero/feasim` v0.1.1.**

This is an unofficial community resource. It is not maintained by UHERO.

## Install

In Claude Code:

```
/plugin marketplace add sakuraet/feasim-skill
/plugin install feasim@feasim-skills
```

To try a local copy:

```
/plugin marketplace add C:/path/to/feasim-skill
/plugin install feasim@feasim-skills
```

After pulling changes, run `/plugin marketplace update feasim-skills`.

For claude.ai, zip `plugins/feasim/skills/feasim` and upload it under Settings → Capabilities → Skills.

## Layout

```
.claude-plugin/marketplace.json        marketplace listing
plugins/feasim/.claude-plugin/plugin.json
plugins/feasim/skills/feasim/
  SKILL.md                             overview, entry points, key identities, gotchas
  reference/inputs.md                  every input, defaults per pathway, valid values
  reference/outputs.md                 output fields, identities, the verdict
  reference/pathways.md                what each pathway changes; Bill 7 grants; TOD tiers
  reference/model-flow.md              step-by-step formulas with source functions
  reference/assumptions.md             fixed constants and their basis
```

## Updating for a new feasim release

1. Diff the package between tags, e.g. `git diff v0.1.1 v0.2.0 -- src README.md` in the feasim repo.
2. Update the affected reference files and the version line in `SKILL.md`.
3. Bump `version` in `plugins/feasim/.claude-plugin/plugin.json`.

## License

MIT. Parts of the content are adapted from the `@uhero/feasim` README, © 2026 UHERO, under the MIT License.
