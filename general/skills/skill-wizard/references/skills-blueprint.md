# The Blueprint for the Perfect Skill

Consolidated from Anthropic's official docs + skill-creator skill, OpenAI Codex, OpenCode, the AGENTS.md spec, and Cursor rules. The convergent picture is unusually clean: **all the major coding agents have settled on the same shape**, just with different filenames.

## Contents

1. [The convergent industry pattern](#1-the-convergent-industry-pattern-april-2026)
2. [The canonical skill file layout](#2-the-canonical-skill-file-layout)
3. [SKILL.md frontmatter — full cheatsheet](#3-skillmd-frontmatter--full-cheatsheet)
4. [Body structure](#4-body-structure-under-500-lines-always)
5. [Match degrees of freedom to task fragility](#5-match-degrees-of-freedom-to-task-fragility)
6. [Scripts and assets — when to bundle code](#6-scripts-and-assets--when-to-bundle-code)
7. [Progressive disclosure — three-tier loading model](#7-progressive-disclosure--the-three-tier-loading-model)
8. [Subagents and model selection](#8-subagents-and-model-selection)
9. [The other `.md` files — what each is for](#9-the-other-md-files--what-each-is-for)
10. [Build skills evaluation-first (the meta-trick)](#10-build-skills-evaluation-first-the-meta-trick)
11. [Anti-patterns](#11-anti-patterns-compiled-from-all-sources)
12. [The pre-flight checklist](#12-the-pre-flight-checklist-use-this-before-publishing-any-skill)
13. [TL;DR](#tldr--what-makes-a-skill-perfect)
14. [Sources](#sources)

---

## 1. The convergent industry pattern (April 2026)

Every serious agent now uses the same three-layer architecture. Knowing this lets you build skills that port across tools.

| Layer | Claude Code | OpenAI Codex | OpenCode | Cursor |
|---|---|---|---|---|
| **Always-loaded global rules** | `CLAUDE.md` (project) + `~/.claude/CLAUDE.md` (user) | `AGENTS.md` (walks dir tree) | `AGENTS.md` + `opencode.json` | `.cursor/rules/*.mdc` |
| **On-demand skills** (loaded only when relevant) | `.claude/skills/<name>/SKILL.md` | `.agents/skills/<name>/` | `.opencode/skills/` + `commands/*.md` | rule files with `description` glob |
| **Subagents / specialists** | `.claude/agents/<name>.md` | (subagents via Codex API) | `.opencode/agents/` | (no first-class equivalent) |

**The crucial insight:** the always-loaded layer carries facts you always need (build commands, conventions); the skill layer carries playbooks loaded only when triggered. Don't conflate them. CLAUDE.md/AGENTS.md should *not* contain step-by-step procedures — those belong in skills.

## 2. The canonical skill file layout

```
.claude/skills/<skill-name>/
├── SKILL.md              # required — frontmatter + overview + navigation
├── reference.md          # optional — API specs, schemas, deep facts
├── examples.md           # optional — input/output pairs
├── workflow.md           # optional — long step-by-step procedures
├── references/           # optional — domain-split docs (finance.md, sales.md…)
│   ├── domain-a.md
│   └── domain-b.md
├── scripts/              # optional — executable helpers
│   ├── validate.py
│   └── generate.sh
└── assets/               # optional — templates, fixtures, icons
    └── template.html
```

This layout is identical in Codex (`.agents/skills/<name>/`) and OpenCode. **Use this layout even if you only have one file today** — future-you will thank you.

## 3. SKILL.md frontmatter — full cheatsheet

```yaml
---
name: processing-pdfs                # required, lowercase-kebab, ≤64 chars, no "claude"/"anthropic"
description: >                       # required, ≤1024 chars, third person, "what + when"
  Extracts text and tables from PDFs, fills forms, merges documents.
  Use when the user mentions PDFs, forms, document extraction, or any
  .pdf file. Make sure to use this skill whenever PDFs come up, even if
  the user doesn't explicitly say "extract."

# --- Claude Code only (skip if you want max portability) ---
allowed-tools: "Read Grep Bash(python:*)"   # pre-approves these tools
disable-model-invocation: false              # true = user-only invocation (deploy, commit)
user-invocable: true                         # false = hidden from "/" menu, model-only
model: sonnet                                # override session model for this skill
effort: high                                 # low | medium | high | xhigh | max
context: fork                                # run in isolated subagent
agent: Explore                               # required when context: fork
paths: "src/**/*.py,tests/**/*.py"           # auto-load only when editing these
argument-hint: "[file] [output]"
arguments: [file, output]
---
```

### The two non-negotiables: `name` and `description`

Everything else is sugar. The description is **the only thing** the model sees at startup to decide whether to pull your skill in. Anthropic's official guidance:

1. **Third person, never first/second.** "Extracts PDF text…" — not "I can extract…" or "You can use this to…".
2. **What + When.** Always include trigger phrases: "Use when the user mentions X, Y, Z, or any .xxx file."
3. **Be pushy.** Claude under-triggers by default. Add: *"Make sure to use this skill whenever the user mentions [topics]."*
4. **Front-load the action verb.** "Generates" / "Validates" / "Deploys" — not "A skill for…".
5. **Avoid vague words** ("helps", "useful", "assists").

Bad: `description: "Helps with documents"`
Good: `description: "Generates descriptive commit messages by analyzing git diffs. Use when the user asks for a commit message, mentions staged changes, or runs git commit workflows."`

## 4. Body structure (under 500 lines, always)

```markdown
# <Skill Name>

## Quick start
[The single most common usage, with a working example. ~10 lines.]

## When to use this skill
[1–3 bullets that mirror the description's trigger phrases.]

## Workflow
1. Step one
2. Step two
3. Step three
   - For X, see [reference.md](reference.md)
   - For Y, run `scripts/validate.py`

## Output format
[Strict template OR flexible guidance — match strictness to fragility.]

## Additional resources
- **Detailed API**: [reference.md](reference.md)
- **Examples**: [examples.md](examples.md)
- **Domain X**: [references/x.md](references/x.md)
```

### Hard rules from Anthropic's best-practices doc

- **≤500 lines in SKILL.md body.** If you overflow, split into supporting files.
- **One level of reference depth.** SKILL.md → `reference.md` is fine. SKILL.md → `a.md` → `b.md` is **broken** — Claude does partial reads (`head -100`) on nested links and misses content.
- **Long reference files (>100 lines) need a TOC at the top** so partial reads still see the structure.
- **Forward slashes only** in paths, even on Windows.
- **Imperative voice + explain why.** "Run validation before packaging — it catches XML errors that pack.py would silently corrupt."
- **Consistent terminology.** Pick one term ("field" vs "box" vs "control") and stick with it.
- **No time-sensitive content.** Put deprecated stuff under a `## Old patterns` section in `<details>`.

## 5. Match degrees of freedom to task fragility

The "robot on a path" metaphor from Anthropic:

| Task type | Freedom | Pattern |
|---|---|---|
| Open field (code review, explanation) | **High** | Prose guidance, principles, checklists |
| Some constraints (report generation) | **Medium** | Pseudocode, parametrized scripts |
| Narrow bridge (DB migration, packaging) | **Low** | "Run exactly: `python migrate.py --verify`. Do not modify." |

Don't write a 200-line bash recipe for a code review skill. Don't write "use your judgment" for a destructive migration.

## 6. Scripts and assets — when to bundle code

Skills can ship scripts that Claude **executes** (preferred) or **reads as reference**. Always make the intent explicit:

- "Run `scripts/analyze.py input.pdf` to extract fields" (execute, output goes into context)
- "See `scripts/analyze.py` for the extraction algorithm" (read as reference)

Why bundle scripts instead of asking Claude to generate code:
- More reliable (no hallucinated APIs)
- Saves tokens (script body never enters context, only output does)
- Deterministic across runs

**Script quality rules:**
- Solve, don't punt — handle errors in the script, don't bubble exceptions up to Claude
- No voodoo constants — every magic number gets a comment with the reason
- Validate intermediate state with feedback loops: `validate → fix → repeat`

## 7. Progressive disclosure — the three-tier loading model

```
Tier 1: Metadata (name + description)        ALWAYS in context
        ↓ (model decides skill is relevant)
Tier 2: SKILL.md body                        Loaded on trigger
        ↓ (skill body references file)
Tier 3: reference.md, examples.md, scripts   Loaded on demand
```

This is why the layout matters: **everything below tier 1 is free until accessed.** Bundle generously — large reference files cost nothing until Claude actually reads them. The only thing that should be lean is the description (Tier 1) and SKILL.md (Tier 2).

For multi-domain skills, **split by domain not by document type**:

```
bigquery/
├── SKILL.md
└── references/
    ├── finance.md      # only loaded for revenue queries
    ├── sales.md        # only loaded for pipeline queries
    └── product.md      # only loaded for usage queries
```

A user asking about revenue never pays the token cost of `sales.md`.

## 8. Subagents and model selection

### Skills cannot spawn subagents directly

A skill runs in the main conversation. To get isolation, use frontmatter:

```yaml
context: fork
agent: Explore       # or Plan, general-purpose, or a custom subagent name
```

This makes the *skill itself* execute in a forked subagent context, returning only the result. Use this when the skill does heavy exploration (greps the whole repo, reads many files) and you don't want that pollution in main context.

### Inverse pattern: subagents that preload skills

Define a subagent and inject skill content at startup via the subagent's `skills:` field:

```yaml
# .claude/agents/api-developer.md
---
name: api-developer
description: Implements API endpoints following team conventions
model: sonnet
skills:
  - api-conventions
  - error-handling-patterns
---
```

Now any time you spawn `api-developer`, both skills are pre-loaded. Use this for **specialist agents** that always need the same domain knowledge.

### Model selection rules of thumb

| Task | Model | Why |
|---|---|---|
| Heavy code generation, planning, multi-file refactors | **Opus** | Best reasoning |
| Most skill bodies (review, explain, generate) | **Sonnet** | Best price/perf, default |
| Simple lookups, formatting, routing | **Haiku** | 5–10× cheaper, plenty smart |
| Skill that calls other skills / orchestrates | **inherit** | Don't fight the parent's choice |

**Test every skill on all three models.** Anthropic's own checklist requires Haiku/Sonnet/Opus testing. A skill that works on Opus often under-specifies for Haiku — Haiku reveals where you assumed too much.

### How many subagents?

- **One per discrete persona/role** (reviewer, scout, planner, writer). Don't make a kitchen-sink agent.
- **Spawn in parallel** when work is independent (3 lookups → 3 agents in one message).
- **Sequential** when one's output feeds the next.
- **Cap at ~5 concurrent** for a typical task. More than that and synthesis becomes the bottleneck.

## 9. The other `.md` files — what each is for

Don't dump everything in CLAUDE.md. Each file has a different job:

| File | Lifecycle | Contents | Example |
|---|---|---|---|
| **`AGENTS.md`** (repo root) | Always loaded by every agent | Build/test commands, conventions, security constraints | `npm test`, "use dayjs for dates" |
| **`CLAUDE.md`** (repo root) | Always loaded by Claude Code | Same as AGENTS.md but Claude-specific | Identical content; many teams symlink |
| **`~/.claude/CLAUDE.md`** | Always loaded, every session | User-level prefs (descriptive vars, never co-author Claude) | Your personal Git rules |
| **`SKILL.md`** | Loaded on trigger | Procedure for one task | "How to deploy", "How to review PRs" |
| **`reference.md`** (in skill dir) | Loaded when skill body links to it | Deep facts, API specs, schemas | OpenAPI snippet, GraphQL schema |
| **`examples.md`** (in skill dir) | Loaded when skill body links to it | Input/output pairs | Commit message before/after |
| **Subagent `.md`** | Loaded only when that subagent is spawned | Persona prompt + which skills to preload | "You are a security reviewer…" |
| **`README.md`** | Never loaded by agents | Human onboarding | Standard repo readme |

**Cross-tool tip:** if you maintain `AGENTS.md`, you cover Codex, Cursor, Copilot, OpenCode, Aider, Devin, Factory, Junie, Warp, Zed in one file. Reserve `CLAUDE.md` for Claude-only things (skill references, plugin notes) and let `AGENTS.md` carry the universal stuff.

## 10. Build skills evaluation-first (the meta-trick)

The single highest-leverage practice from Anthropic's docs and the skill-creator skill:

1. **Pick 3 representative tasks** the skill should improve.
2. **Run them without the skill.** Capture the failures.
3. **Write the minimum SKILL.md** that fixes those failures.
4. **Re-run the same tasks with the skill loaded.** Compare.
5. **Iterate.** Add only what fixes a measured failure.

This stops you from writing 400 lines of guidance Claude doesn't need ("Claude is already very smart" — Anthropic's literal phrasing). Every paragraph must justify its tokens.

The "two-Claude" loop:
- **Claude A** (a chat with you) helps you *write* and refine the skill.
- **Claude B** (fresh session with the skill loaded) tests it on real tasks.
- Watch Claude B. Where it stumbles, take notes back to Claude A.

## 11. Anti-patterns (compiled from all sources)

- Description in first/second person ("I can…", "You can…")
- Vague descriptions without trigger phrases
- SKILL.md > 500 lines
- Reference chains more than 1 level deep
- Multiple competing options ("use pypdf, or pdfplumber, or PyMuPDF…") — pick a default
- Magic numbers in scripts without justifying comments
- Time-sensitive instructions ("after August 2025…")
- Inconsistent terminology
- Backslash paths
- Letting scripts punt errors back to Claude instead of handling them
- Stuffing AGENTS.md/CLAUDE.md with procedures that should be skills
- Putting skills behind `disable-model-invocation: true` unless the action has real-world side effects (deploy, send, commit)

## 12. The pre-flight checklist (use this before publishing any skill)

Adapted from Anthropic's official checklist:

**Discovery**
- [ ] `name` is gerund-form, lowercase-kebab, descriptive, no reserved words
- [ ] `description` is third person, ≤1024 chars, has both *what* and *when*
- [ ] Description includes pushy trigger phrases ("use whenever…")
- [ ] Description tested: type a natural query — does the model invoke the skill?

**Structure**
- [ ] SKILL.md body ≤500 lines
- [ ] Files referenced from SKILL.md are exactly 1 level deep
- [ ] Reference files >100 lines have a TOC
- [ ] Multi-domain content split into `references/<domain>.md`
- [ ] Scripts isolated in `scripts/`, intent (execute vs read) is explicit

**Content quality**
- [ ] Concise — no PDF/HTTP/library 101 explanations
- [ ] Imperative voice; "why" given for non-obvious rules
- [ ] One term per concept, used consistently
- [ ] No time-sensitive content (or quarantined under `## Old patterns`)
- [ ] Forward slashes only
- [ ] Examples are concrete I/O pairs, not abstract descriptions

**Robustness**
- [ ] Validation/feedback loops on critical operations
- [ ] Scripts handle errors instead of throwing to Claude
- [ ] No magic constants
- [ ] MCP tools (if used) are fully qualified: `Server:tool_name`

**Testing**
- [ ] At least 3 evaluation tasks created and run
- [ ] Tested on Haiku, Sonnet, and Opus
- [ ] Tested by someone other than the author (or a fresh Claude session)

---

## TL;DR — what makes a skill "perfect"

A perfect skill is **one tightly-scoped procedure**, with a **third-person, pushy description that mirrors how users actually phrase the trigger**, a **≤500-line SKILL.md** that points to deeper material via **one level of references**, **bundled scripts for anything deterministic**, **frontmatter that pins model and tool permissions**, and **3+ evaluations proving it beats no-skill on real tasks** across Haiku, Sonnet, and Opus.

Pair it with a clean repo-root `AGENTS.md` (universal rules) and a `CLAUDE.md` (Claude-specifics), and you have a setup that works in Claude Code, Codex, OpenCode, Cursor, and most of the rest with no rework.

---

## Sources

- [Extend Claude with skills — Claude Code Docs](https://code.claude.com/docs/en/skills)
- [Skill authoring best practices — Claude API Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Agent Skills overview — Claude API Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [skill-creator SKILL.md — anthropics/skills](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)
- [The Complete Guide to Building Skills for Claude (Anthropic PDF)](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf?hsLang=en)
- [Agent Skills — OpenAI Codex](https://developers.openai.com/codex/skills)
- [Custom instructions with AGENTS.md — OpenAI Codex](https://developers.openai.com/codex/guides/agents-md)
- [Agent Skills — OpenCode](https://opencode.ai/docs/skills/)
- [Commands — OpenCode](https://opencode.ai/docs/commands/)
- [Rules — OpenCode](https://opencode.ai/docs/rules/)
- [AGENTS.md spec](https://agents.md/)
- [agentsmd/agents.md — GitHub](https://github.com/agentsmd/agents.md)
- [How to write a great agents.md — GitHub Blog](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/)
- [Cursor Rules docs](https://cursor.com/docs/rules)
- [Best practices for coding with agents — Cursor Blog](https://cursor.com/blog/agent-best-practices)
