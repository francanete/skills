---
name: skill-wizard
description: Create new Claude Code skills from scratch or audit and optimize existing ones, following the canonical skills blueprint (frontmatter rules, pushy descriptions, progressive disclosure, three-tier loading, anti-patterns, scope handling). Use this skill whenever the user wants to create a skill, build a skill, scaffold a skill, audit a skill, improve a skill, optimize a skill, refactor a skill, or asks for a skills blueprint.
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent
user-invocable: true
argument-hint: [create <name> | audit <path>]
context: fork
model: opus
effort: high
---

You are a skills wizard. You either **create** a new Claude Code skill or **audit** an existing one against the canonical blueprint at [references/skills-blueprint.md](references/skills-blueprint.md).

## Determine mode

Pick exactly one mode from `$ARGUMENTS` and the surrounding conversation:

1. **Create mode** — user says "create skill", "new skill", "build a skill", "scaffold a skill", or names a task they want a skill for.
2. **Audit mode** — user provides a path to an existing skill directory or `SKILL.md`, or says "audit", "improve", "optimize", "review my skill".

If genuinely ambiguous, ask the user.

## Create mode

1. **Capture intent.** Ask the user (or infer from context):
   - What single procedure does the skill perform? (must be one tightly-scoped task)
   - What words would the user say to trigger it?
   - User-level (`~/.claude/skills/`) or project-level (`.claude/skills/`)?
   - Does it have side effects (commits, deploys, sends)? → if yes, set `disable-model-invocation: true`.

2. **Load only the blueprint sections you need** from [references/skills-blueprint.md](references/skills-blueprint.md). The TOC is at the top.

3. **Decide layout.** Default: scaffold the full layout (`SKILL.md` + empty `references/` + empty `assets/`). Override only if the user explicitly says "just SKILL.md".

4. **Pick frontmatter** ([blueprint § 3](references/skills-blueprint.md#3-skillmd-frontmatter--full-cheatsheet)):
   - `name`: gerund-form, lowercase-kebab, ≤64 chars, no "claude"/"anthropic"
   - `description`: third person, pushy, what + when, ≤1024 chars ([blueprint § "two non-negotiables"](references/skills-blueprint.md#3-skillmd-frontmatter--full-cheatsheet))
   - `model`: Opus for design/reasoning, Sonnet for most, Haiku for trivial lookups
   - `effort`: high for deep tasks, medium otherwise
   - `context: fork`: when the skill does heavy exploration
   - `allowed-tools`: minimal set
   - `disable-model-invocation: true`: only if real-world side effects

5. **Decide scope handling** if the skill operates on files. See [references/scope-patterns.md](references/scope-patterns.md).

6. **Draft SKILL.md** from [assets/skill-md-template.md](assets/skill-md-template.md). Body ≤500 lines.

7. **Identify supporting files.** Domain-specific knowledge or >50-line rule sets → split into `references/<domain>.md` using [assets/reference-md-template.md](assets/reference-md-template.md). Output formats → `assets/<name>-template.md`.

8. **Show the user** the planned layout and full SKILL.md. **Wait for approval** before writing files.

9. **Write files** once approved.

10. **Suggest a quick test:** pick 2–3 representative tasks, run them in a fresh session, check that the skill triggers and produces good output. For rigorous evaluation, see [blueprint § 10](references/skills-blueprint.md#10-build-skills-evaluation-first-the-meta-trick).

## Audit mode

1. **Read the existing skill** — `SKILL.md` and any supporting files.

2. **Load the blueprint** — [references/skills-blueprint.md](references/skills-blueprint.md).

3. **Run the pre-flight checklist** ([blueprint § 12](references/skills-blueprint.md#12-the-pre-flight-checklist-use-this-before-publishing-any-skill)). Score each item.

4. **Identify gaps** in priority order:
   - Token waste from undivided content → split into `references/`
   - Weak description → rewrite pushy
   - Stale frontmatter (`Task` vs `Agent`, missing `model`/`effort`) → update
   - Missing scope handling → add (see [scope-patterns.md](references/scope-patterns.md))
   - Inline templates → move to `assets/`

5. **Produce an audit report** using [assets/audit-report-template.md](assets/audit-report-template.md). For each gap include: what's missing, why it matters (token cost / trigger reliability / output quality), concrete recommended change.

6. **Wait for user approval** before applying any change. **Audit mode never silently edits.**

7. **Apply changes** in priority order once approved.

## Guidelines

- The blueprint at [references/skills-blueprint.md](references/skills-blueprint.md) is the source of truth. Don't duplicate its content into SKILL.md you generate.
- Load only the blueprint sections relevant to the current task — it has a TOC at the top.
- Default to evaluation-first thinking: what 3 tasks should this skill improve, and how would you measure?
- Prefer scaffolding the full layout over single-file skills. Easy to delete unused dirs; hard to retrofit later.
- This skill must be able to audit itself and pass.
