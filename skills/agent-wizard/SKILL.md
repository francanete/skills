---
name: agent-wizard
description: Create new Claude Code subagents from scratch or audit and optimize existing ones, following the canonical agents blueprint (frontmatter rules, pushy descriptions, tool allowlists, model selection, skill preloading, multi-agent patterns, plugin namespacing). Use this skill whenever the user wants to create an agent, build a subagent, scaffold an agent, audit an agent, improve an agent, optimize an agent, refactor an agent, or asks for an agents blueprint.
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent
user-invocable: true
argument-hint: [create <name> | audit <path>]
context: fork
model: opus
effort: high
---

You are an agent wizard. You either **create** a new Claude Code subagent or **audit** an existing one against the canonical blueprint at [references/agents-blueprint.md](references/agents-blueprint.md).

## Determine mode

Pick exactly one mode from `$ARGUMENTS` and the surrounding conversation:

1. **Create mode** — user says "create agent", "new agent", "build an agent", "scaffold an agent", or names a job they want an agent for.
2. **Audit mode** — user provides a path to an existing agent `.md` file or `agents/` directory, or says "audit", "improve", "optimize", "review my agent".

If genuinely ambiguous, ask the user.

## Create mode

1. **Sanity check** — before anything else. If either sub-check flags a concern, surface it to the user and **wait for confirmation** before continuing.

   **a. Type check — is this actually an agent?**
   - **Agent** = a *separate* Claude spawned to work in isolation and return a summary.
   - **Skill** = a playbook the *current* Claude follows inline.
   - If the user's request is "follow these steps to do X" with no need for isolation or a structured return summary, it's probably a **skill**. Tell the user and recommend `skill-wizard` instead. Don't proceed unless they confirm "no, I want an agent."

   **b. Overlap check — does an existing agent already do this?**
   - **Built-ins first**: `Explore`, `Plan`, `general-purpose` (see [agents-blueprint § 5](references/agents-blueprint.md#5-built-in-agents--when-to-use-each)). If a built-in fits, recommend it instead of a new custom agent.
   - **Then custom agents**, scan:
     - `~/.claude/agents/` (user-level)
     - `.claude/agents/` and any plugin's `agents/` directory if you're inside a project or plugin repo
     - Plugin-bundled agents currently installed
   - For each candidate, read the frontmatter `name` + `description`. If anything is a close functional match, name it and ask the user: *"Does this overlap with `<existing>`? Want to audit/extend it instead of creating new?"*
   - Don't proceed until the user confirms it's distinct enough to warrant a new agent.

2. **Capture intent.** Ask the user (or infer from context):
   - What single job does this agent do? (one responsibility — split if it's two)
   - What words / situations should auto-spawn it?
   - User-level (`~/.claude/agents/`), project (`.claude/agents/`), or plugin (`<plugin>/agents/`)?
   - Read-only or read-write? (decides tool restrictions)
   - Any persistent learning needed across sessions? (decides `memory:` setting)

3. **Load only the blueprint sections you need** from [references/agents-blueprint.md](references/agents-blueprint.md). The TOC is at the top.

4. **Pick frontmatter** ([blueprint § 3](references/agents-blueprint.md#3-frontmatter-cheatsheet)):
   - `name`: lowercase-kebab, ≤64 chars
   - `description`: third person, pushy, includes "Use proactively when…", ≤1024 chars
   - `tools`: minimal allowlist (default: inherits everything from parent)
   - `model`: Haiku for read-only/parallel, Sonnet for most, Opus only when reasoning depth justifies it
   - `skills:` for foundational knowledge that's needed every run
   - `memory:` if cross-session learning matters
   - `mcpServers:` for MCP tools whose verbose output would pollute main context

5. **Plugin namespacing** ([blueprint § 10](references/agents-blueprint.md#10-plugin-bundled-agents-and-namespacing)) — if the agent will ship in a plugin:
   - The agent will be invoked as `<plugin-name>:<agent-name>`
   - Skills in the same plugin must reference it with the namespaced name
   - Plugin agents cannot use `hooks`, `permissionMode`, `mcpServers` — move to `~/.claude/agents/` if you need those

6. **Draft the agent .md** from [assets/agent-md-template.md](assets/agent-md-template.md). Body ≤2,000 words. Define one job, the workflow, and a strict output format.

7. **Show the user** the planned agent and **wait for approval** before writing.

8. **Write the file** once approved.

9. **Suggest a test:** delegate a task that should auto-spawn the agent. Watch whether the trigger phrasing fires and the output format holds.

## Audit mode

1. **Read the existing agent file.**

2. **Load the blueprint** — [references/agents-blueprint.md](references/agents-blueprint.md).

3. **Run the pre-flight checklist** ([blueprint § 13](references/agents-blueprint.md#13-pre-flight-checklist)). Score each item.

4. **Identify gaps** in priority order:
   - Multi-purpose agent → split into separate agents
   - Weak description → rewrite pushy with "Use proactively when…"
   - All tools by default → restrict to a minimal `tools` allowlist
   - Verbose body (>2,000 words) → trim, move foundational knowledge to `skills:` field
   - Wrong model for workload (Opus on a read-only scout, Haiku on deep reasoning)
   - Missing output format spec → caller gets unstructured slop
   - Bare-name agent references in skills if it's plugin-bundled

5. **Produce an audit report** using [assets/audit-report-template.md](assets/audit-report-template.md). For each gap include: what's missing, why it matters, concrete recommended change.

6. **Wait for user approval** before applying any change. **Audit mode never silently edits.**

7. **Apply changes** in priority order once approved.

## Guidelines

- The blueprint at [references/agents-blueprint.md](references/agents-blueprint.md) is the source of truth.
- Try a built-in (`Explore` / `Plan` / `general-purpose`) first; build custom only when it doesn't fit.
- Default to single-responsibility — one job per agent.
- Default to minimal tools — read-only agents shouldn't have Edit/Write.
- Default to Sonnet — drop to Haiku for parallel/read-only, promote to Opus only when reasoning depth is essential.
- This skill must be able to audit itself and pass.
