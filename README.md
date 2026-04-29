# franc-skills

Personal collection of Claude Code skills, packaged as an installable plugin.

## Skills

| Skill | Purpose |
|---|---|
| `/franc-skills:skill-wizard` | Create new skills from scratch or audit/optimize existing ones against the canonical skills blueprint |
| `/franc-skills:quality-code-review` | Deep clean-code review of a specific file, the current branch's commits, or uncommitted changes |
| `/franc-skills:review-uncommitted` | Quick pre-commit review — lighter and faster than `quality-code-review` |
| `/franc-skills:seo-content-writer` | SEO blog posts that rank in traditional search and AI engines |
| `/franc-skills:copy-writer-expert` | Direct-response copy review using Gary Halbert's principles |
| `/franc-skills:product-ideas` | Generate validated commercial product improvement opportunities |
| `/franc-skills:pm-issue` | Create a Linear issue from conversation context |
| `/franc-skills:test-coverage` | Analyze test coverage and suggest test cases |

## Install

From the repo URL (after pushing to GitHub):

```bash
# In Claude Code
/plugin install https://github.com/<your-username>/skills
```

From a local clone:

```bash
/plugin install /Users/francanete/workspace/skills
```

## Authoring new skills

Use `/franc-skills:skill-wizard` to scaffold or audit skills. The canonical blueprint lives at:

```
skills/skill-wizard/references/skills-blueprint.md
```

## Layout

```
.claude-plugin/
└── plugin.json            # plugin manifest
skills/
├── <skill-name>/
│   ├── SKILL.md           # required — frontmatter + workflow
│   ├── references/        # optional — domain knowledge loaded on demand
│   └── assets/            # optional — templates and output formats
└── ...
```
# skills
