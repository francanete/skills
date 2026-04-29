# skills

Personal Claude Code skills, organized into two installable plugins.

## Plugins

### `fran-general` — general Claude Code tooling

| Skill | Purpose |
|---|---|
| `/fran-general:skill-wizard` | Create new skills from scratch or audit/optimize existing ones against the canonical skills blueprint |
| `/fran-general:agent-wizard` | Create new subagents from scratch or audit/optimize existing ones against the canonical agents blueprint |
| `/fran-general:quality-code-review` | Deep clean-code review of a file, the current branch's commits, or uncommitted changes |
| `/fran-general:review-uncommitted` | Quick pre-commit review — lighter and faster than `quality-code-review` |

### `fran-productivity` — content, copy, ideation, issues

| Skill | Purpose |
|---|---|
| `/fran-productivity:seo-content-writer` | SEO blog posts that rank in traditional search and AI engines |
| `/fran-productivity:copy-writer-expert` | Direct-response copy review using Gary Halbert's principles |
| `/fran-productivity:product-ideas` | Generate validated commercial product improvement opportunities |
| `/fran-productivity:pm-issue` | Create a Linear issue from conversation context |

## Install

Both plugins are listed in [`francanete/fran-marketplace`](https://github.com/francanete/fran-marketplace). Add the marketplace once, then install whichever plugins you want:

```bash
# In Claude Code
/plugin marketplace add francanete/fran-marketplace

# Install both
/plugin install fran-general@fran-marketplace
/plugin install fran-productivity@fran-marketplace

# Or just the general tools (e.g. on a work machine)
/plugin install fran-general@fran-marketplace
```

## Authoring new skills and agents

Use `/fran-general:skill-wizard` to scaffold or audit skills. Use `/fran-general:agent-wizard` for subagents. Canonical blueprints:

```
general/skills/skill-wizard/references/skills-blueprint.md
general/skills/agent-wizard/references/agents-blueprint.md
```

## Layout

```
general/
├── .claude-plugin/plugin.json    # plugin "fran-general"
└── skills/
    ├── skill-wizard/
    ├── agent-wizard/
    ├── quality-code-review/
    └── review-uncommitted/
productivity/
├── .claude-plugin/plugin.json    # plugin "fran-productivity"
└── skills/
    ├── seo-content-writer/
    ├── copy-writer-expert/
    ├── product-ideas/
    └── pm-issue/
```
