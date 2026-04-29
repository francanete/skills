# The Blueprint for the Perfect Agent

Consolidated from Anthropic's Claude Code subagents docs (April 2026), OpenAI Codex subagents (GA March 14 2026), and the broader multi-agent framework landscape (CrewAI, LangGraph, AutoGen, OpenAI Swarm).

## Contents

1. [Agents vs skills — the core distinction](#1-agents-vs-skills--the-core-distinction)
2. [The canonical agent file layout](#2-the-canonical-agent-file-layout)
3. [Frontmatter cheatsheet](#3-frontmatter-cheatsheet)
4. [Body / system prompt structure](#4-body--system-prompt-structure)
5. [Built-in agents — when to use each](#5-built-in-agents--when-to-use-each)
6. [Description writing — getting auto-spawn right](#6-description-writing--getting-auto-spawn-right)
7. [Tool restrictions and permissions](#7-tool-restrictions-and-permissions)
8. [Model selection](#8-model-selection)
9. [Composing agents with skills](#9-composing-agents-with-skills)
10. [Plugin-bundled agents and namespacing](#10-plugin-bundled-agents-and-namespacing)
11. [Multi-agent patterns (parallel / sequential / hierarchical)](#11-multi-agent-patterns)
12. [Anti-patterns](#12-anti-patterns)
13. [Pre-flight checklist](#13-pre-flight-checklist)
14. [TL;DR](#tldr--what-makes-an-agent-perfect)
15. [Sources](#sources)

---

## 1. Agents vs skills — the core distinction

| | Agent (subagent) | Skill |
|---|---|---|
| **What it is** | A separate Claude instance spawned to do work | A playbook the current Claude follows |
| **Context** | Fresh, isolated — doesn't see main conversation | Inline — runs in main conversation context |
| **Returns** | A single summary message back to caller | Whatever it does inline |
| **System prompt** | Replaced entirely by agent body | Appended to existing context |
| **Trigger** | Auto-spawn from description match, `@mention`, or Agent tool call | Auto-trigger from description match, or `/skill-name` |
| **Multi-file** | ❌ Single `.md` file only | ✅ Supports `references/`, `assets/`, `scripts/` |
| **Best for** | Heavy exploration, parallel work, isolating verbose output | Step-by-step procedures, knowledge injection, workflows |

**One-line rule:** skill = teaching this Claude how to do something; agent = sending another Claude to do something.

The skill layer carries playbooks. The agent layer carries workers. They compose: skills can spawn agents (`Agent` tool) and agents can preload skills (`skills:` field).

## 2. The canonical agent file layout

```
.claude/agents/
└── <agent-name>.md          # the entire agent
```

Or directory form (less common but supported):

```
.claude/agents/
└── <agent-name>/
    └── agent.md             # any .md filename works
```

**Critical difference from skills:** agents are **single-file**. There's no `references/`, no `assets/`, no `scripts/`. Everything lives in one `.md` file: frontmatter + system prompt body. If your agent needs deep knowledge, either:
- Bake it into the body (under ~2,000 words), or
- Preload it via the `skills:` frontmatter field (see § 9), or
- Have the agent read external files at runtime via its allowed tools.

### Storage scopes (priority order)

| Priority | Location | Scope |
|---|---|---|
| 1 (highest) | Managed enterprise settings | Org-wide |
| 2 | CLI flag `--agents` | Single session |
| 3 | `.claude/agents/<name>.md` | Project (commit to repo) |
| 4 | `~/.claude/agents/<name>.md` | Personal (all your projects) |
| 5 | `<plugin>/agents/<name>.md` | When plugin is active (namespaced) |

## 3. Frontmatter cheatsheet

```yaml
---
name: code-reviewer                  # required, lowercase-kebab, ≤64 chars
description: >                       # required, ≤1024 chars (truncated to 1536 in listings)
  Expert code reviewer for quality, security, and best practices.
  Use proactively after code changes, when reviewing PRs, or when the
  user asks "is this code good?" Make sure to delegate to this agent
  whenever changes are committed or PRs are opened.

# --- Tool access ---
tools: Read, Grep, Glob, Bash         # ALLOWLIST. If set, only these.
disallowedTools: Write, Edit          # DENYLIST. Applied first if both set.

# --- Behavior ---
model: sonnet                         # opus | sonnet | haiku | inherit | full ID
effort: high                          # low | medium | high | xhigh | max
permissionMode: default               # default | acceptEdits | auto | dontAsk | bypassPermissions | plan
maxTurns: 20                          # cap on agentic loop iterations

# --- Composition ---
skills:                               # preload full skill content into agent's system prompt
  - api-conventions
  - testing-best-practices
mcpServers:                           # MCP servers exposed to agent (scoped, kept out of main context)
  - github
  - playwright:
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]

# --- Lifecycle ---
hooks:                                # PreToolUse, PostToolUse, Stop (becomes SubagentStop)
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate.sh"
memory: project                       # user | project | local — persistent cross-session memory
background: false                     # true = always run as background task
isolation: worktree                   # spawn in temp git worktree (isolated repo copy)
initialPrompt: "Start by reviewing the latest commit"   # auto-submit on agent startup

# --- UI ---
color: blue                           # red | blue | green | yellow | purple | orange | pink | cyan
---
```

### The two non-negotiables: `name` and `description`

Same as for skills, but with two agent-specific twists:

1. **Description doubles as a routing rule.** When Claude decides "do I have an agent for this?", it scans descriptions. Phrases like *"Use proactively when…"* and *"Use this agent whenever…"* dramatically improve auto-delegation.
2. **Models tend to under-delegate.** Agents are more expensive than running inline, so Claude is conservative. Counter this by being pushier in the description than you would be for a skill.

### Field semantics that bite

- **`tools` is an allowlist, not a hint.** If you set `tools: Read, Bash`, the agent literally cannot call anything else, even if the parent session has more permissions.
- **`disallowedTools` applies first** when both fields are set. Then the allowlist is resolved against what's left.
- **Restricting subagent spawning:** `tools: Agent(worker, researcher)` only lets this agent spawn those two named subagents.
- **`model: inherit`** is the default if omitted — the agent uses whatever the parent uses.
- **`memory: project`** writes to `.claude/agent-memory/<name>/` and is git-tracked. `memory: local` writes to `.claude/agent-memory-local/<name>/` and is NOT tracked. `memory: user` writes to `~/.claude/agent-memory/<name>/`.

## 4. Body / system prompt structure

The markdown after the frontmatter **replaces** the Claude Code system prompt for the agent (skills are different — they append). Treat it like designing a job description.

```markdown
You are a <role>. <One-sentence mission statement.>

## Your job
<What this agent does, scope boundaries, what it does NOT do>

## Workflow
1. <Step 1>
2. <Step 2>
3. <Step 3>

## Output format
<Exact structure of the report you return to the caller>
[Templates beat prose here. The caller's main Claude only sees the final summary.]

## Rules
- <Specific behavioral rules>
- <Anti-patterns to avoid>
- <When to ask for clarification vs proceed>
```

### Length recommendations

- **Under 2,000 words** keeps startup latency low.
- **One responsibility per agent.** "Code reviewer" beats "code reviewer and refactorer and test writer."
- **Imperative voice. Explain why for non-obvious rules.** Same as skill bodies.
- **Output format is critical.** The agent's value comes from the *summary* it returns. Specify it precisely.

## 5. Built-in agents — when to use each

Don't write a custom agent if a built-in fits. Claude Code ships with three:

| Agent | Model | Tools | Use when |
|---|---|---|---|
| **`Explore`** | Haiku | Read-only (Read, Grep, Glob, Bash) | Fast codebase search, "where is X defined", listing files matching a pattern |
| **`Plan`** | Inherits | Read-only | Gathering context during plan mode before proposing implementation |
| **`general-purpose`** | Inherits | All tools | Complex multi-step tasks needing both research and modification |

`Explore` is the workhorse — it's cheap, fast, and parallel-friendly. Reach for it first. Build custom agents only when you need:
- A specialized persona / system prompt
- A specific output format
- Preloaded domain knowledge via `skills:`
- A locked-down tool set
- Persistent memory across sessions

## 6. Description writing — getting auto-spawn right

Same rules as skill descriptions, with agent-specific emphasis:

1. **Third person, action-led.** "Reviews code for…" not "I review code."
2. **Front-load the use case.** Truncation hits at ~200 chars in listings.
3. **Include the word "proactively"** if you want auto-delegation. It's the strongest signal.
4. **List trigger phrases.** "Use when reviewing PRs, asking 'is this code good?', or after committing changes."
5. **Be pushier than for skills.** Models under-delegate to agents.

### The "Examples:" pattern (common but not official)

Many published agent files include embedded user/assistant examples in the description:

```yaml
description: |
  Codebase exploration specialist. Use BEFORE planning a feature/fix/refactor.

  Examples:
  - User: "I want to plan adding Google OAuth"
    Assistant: "Let me use codebase-scout to explore the auth system first."
  - User: "Plan the payment refactor"
    Assistant: "I'll launch codebase-scout to map dependencies and tests."
```

**This is NOT an official Anthropic convention.** It's not in the published frontmatter spec. But it's widely used (the `/agents` "Generate with Claude" UI tends to produce it) and it works — examples prime auto-delegation. Use it when you need to disambiguate edge cases, but don't rely on it for the main trigger logic. Front-load the action statement first.

## 7. Tool restrictions and permissions

### Allowlist (`tools`) vs denylist (`disallowedTools`)

| Scenario | Result |
|---|---|
| `tools` only | Agent uses only those tools |
| `disallowedTools` only | Agent inherits parent's tools minus the denied ones |
| Both set | Denylist applied first, then allowlist resolved against remainder |
| Neither set | Inherits everything from parent (default) |

### Interplay with global permissions

Agent restrictions are layered **on top of** parent permissions. The narrower set wins:

| Global says | Agent says | Result |
|---|---|---|
| Allow `Bash` | `tools: Read, Bash` | Both allowed |
| Deny `Bash` | `tools: Bash` | Denied (parent deny wins) |
| `bypassPermissions` | `tools: Read` | Agent still restricted to Read |

### MCP server scoping via `mcpServers`

Use this to keep verbose MCP tools (Playwright screenshots, GitHub API responses) out of the main conversation:

```yaml
mcpServers:
  - playwright:
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]
```

The server connects when the agent starts and disconnects when it finishes. Main conversation never sees the verbose output.

## 8. Model selection

| Model | Speed | Cost | Best for |
|---|---|---|---|
| **Haiku** | Very fast | Lowest | Read-only exploration (this is what `Explore` uses), simple verification, parallel scouts |
| **Sonnet** | Fast | Mid | Code review, debugging, refactoring suggestions, default for most agents |
| **Opus** | Slow | Highest | Architecture decisions, multi-faceted analysis, hard reasoning |

### Resolution order

1. `CLAUDE_CODE_SUBAGENT_MODEL` env var (overrides everything)
2. Per-invocation `model` parameter passed by Claude
3. Agent's `model` field
4. Main conversation's model

### Cost-saving heuristic

Default to **Sonnet**. Drop to **Haiku** if the agent is purely read-only or runs many in parallel. Promote to **Opus** only when reasoning depth genuinely justifies the cost — and even then, often a Sonnet agent with a sharper system prompt beats a vague Opus agent.

## 9. Composing agents with skills

There are two directions of composition:

### Skill → Agent (skill spawns an agent)

In a skill's frontmatter:
```yaml
context: fork
agent: Explore           # or a custom agent name
```

The skill body becomes the task; the agent runs it in isolation. Returns a summary to main context.

### Agent → Skill (agent preloads skill content)

In an agent's frontmatter:
```yaml
skills:
  - api-conventions
  - testing-best-practices
```

When the agent starts, the **full content** of those skills is injected into its system prompt. Token cost paid at startup, every time. Use for foundational knowledge the agent always needs.

**Constraint:** you cannot preload a skill with `disable-model-invocation: true`.

**When to preload vs reference:**

| Approach | When |
|---|---|
| Preload via `skills:` | Knowledge the agent needs every run, cheap to load (under ~1k tokens), foundational |
| Have the agent `Read` it at runtime | Knowledge that's conditional, large, or rarely needed |
| Bake into body | Behavior-defining rules; one-of-a-kind to this agent |

## 10. Plugin-bundled agents and namespacing

When agents ship inside a plugin, **they are namespaced**:

| Source | Invocation name |
|---|---|
| `~/.claude/agents/codebase-scout.md` | `codebase-scout` |
| `.claude/agents/codebase-scout.md` (project) | `codebase-scout` |
| `<plugin-name>/agents/codebase-scout.md` | `<plugin-name>:codebase-scout` |

Same pattern as plugin skills (`<plugin-name>:<skill-name>`).

### The portability gotcha

If a **skill inside a plugin** references an agent **inside the same plugin**, it must use the namespaced name:

```markdown
<!-- Inside fran-skills/skills/product-ideas/SKILL.md -->
Use `fran-skills:codebase-scout` to explore the auth system.
```

NOT `codebase-scout` (bare name). The bare name only works for user/project agents, not plugin-bundled ones.

### Restrictions on plugin agents

Plugin agents cannot use these frontmatter fields (security):
- `hooks`
- `permissionMode`
- `mcpServers`

If you need any of those, the agent must live in `~/.claude/agents/` or `.claude/agents/`, not in a plugin.

## 11. Multi-agent patterns

The 2026 industry has converged on three composition patterns. Claude Code supports all three; pick by task shape.

### Parallel (swarm)

Spawn N agents in **one message** with **multiple Agent tool calls**. They run concurrently. Main Claude synthesizes their reports.

```
You: "Audit the codebase for product opportunities"
→ Main spawns 5 agents in parallel:
   - Product-surface scout (Explore)
   - Monetization scout (Explore)
   - UX-friction scout (Explore)
   - Competitive scout (Explore)
   - Tech-debt scout (Explore)
→ All 5 return reports
→ Main synthesizes into a ranked list
```

This is the OpenAI Swarm / OpenAI Codex subagents (March 2026) pattern, and how the `product-ideas` skill works. Cap concurrent agents at ~5 — synthesis becomes the bottleneck above that.

### Sequential (pipeline)

Agent A's output feeds agent B. Used when stages have hard dependencies.

```
codebase-scout → planner → implementer → reviewer
```

### Hierarchical (manager + workers)

A manager agent decomposes a task and delegates to specialist workers. This is OpenAI Codex's GA pattern (March 2026) — they shipped explicit `explorer` (read-only), `worker` (read-write), and `default` roles, with a manager coordinating up to 6 concurrent workers.

In Claude Code, you build hierarchical setups by giving an agent the `Agent` tool restricted to specific subagent names:

```yaml
tools: Read, Grep, Agent(explorer-1, explorer-2, worker-impl)
```

### Cross-tool industry context

| Framework | Strength |
|---|---|
| **OpenAI Codex subagents** (GA March 2026) | Hierarchical manager + parallel workers, isolated cloud sandboxes |
| **CrewAI** | Role-based teams, easiest to model business workflows |
| **LangGraph** | Graph-based, best for production durability + state management |
| **AutoGen** | Conversational group chat, good for debate/consensus tasks |
| **OpenAI Swarm** | Lowest latency, native function-call routing |
| **Claude Code subagents** | Best ergonomics for codebase work, integrated tool/permission model, plugin distribution |

## 12. Anti-patterns

- Multi-purpose agents. "code-reviewer-and-refactorer-and-test-writer" — split into three.
- Vague descriptions ("helper agent", "general utility").
- All tools by default. A read-only reviewer with `Edit` and `Write` is a footgun.
- Verbose system prompts. Over 2,000 words slows startup and dilutes focus.
- Preloading 5+ skills at the top of the agent — pay the token cost every run.
- Forgetting "Use proactively" when you want auto-delegation.
- Bare-name references inside plugins. Skill says "use `codebase-scout`" but the agent is `fran-skills:codebase-scout`.
- Overriding built-in agent names (`Explore`, `Plan`, `general-purpose`).
- Using an agent when a skill would do. Don't spawn isolation just to follow a checklist.
- No memory for recurring agents. A debugger that re-learns the codebase every session is wasteful.
- No output format spec. "Tell me what you find" produces unstructured slop.

## 13. Pre-flight checklist

**Discovery**
- [ ] `name` is lowercase-kebab, descriptive, no reserved names
- [ ] `description` is third person, ≤1024 chars, has both *what* and *when*
- [ ] Description includes pushy trigger phrases ("Use proactively when…")
- [ ] If used inside a plugin, references to this agent use the `plugin-name:agent-name` form

**Body**
- [ ] System prompt under 2,000 words
- [ ] Single responsibility — one job, well-defined
- [ ] Output format specified with a concrete template
- [ ] Imperative voice, "why" given for non-obvious rules
- [ ] No time-sensitive content

**Tools**
- [ ] `tools` allowlist explicitly minimum-needed
- [ ] Read-only agents don't have Edit/Write
- [ ] MCP servers scoped via `mcpServers` if their output would pollute main context

**Composition**
- [ ] If preloading skills, all listed skills exist and don't have `disable-model-invocation: true`
- [ ] Preloaded skill total under ~3k tokens (otherwise prefer runtime read)

**Model & resources**
- [ ] Model picked deliberately: Haiku (read-only/parallel), Sonnet (default), Opus (deep reasoning)
- [ ] `memory:` set if the agent has cross-session value

**Testing**
- [ ] Triggers automatically when you'd expect (test natural-language queries)
- [ ] Returns a clean, structured summary (not a wall of unstructured text)
- [ ] Doesn't blow up the main context with debug output

---

## TL;DR — what makes an agent "perfect"

A perfect agent is **a single tightly-scoped worker**, with a **third-person, pushy description that tells the model exactly when to delegate**, a **system prompt under 2,000 words** that defines one job and one output format, a **minimal `tools` allowlist**, the **right model for the workload** (Haiku for read-only/parallel, Sonnet for most, Opus only when justified), and **either preloaded skills or runtime file reads** for any domain knowledge it needs.

When in doubt: try `Explore` first. Build custom only when no built-in fits.

---

## Sources

### Claude Code

- [Create custom subagents — Claude Code Docs](https://code.claude.com/docs/en/sub-agents)
- [Agent Skills — Claude API Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [Skill authoring best practices — Claude API Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Equipping agents for the real world with Agent Skills — Anthropic](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- [Best practices for Claude Code subagents — PubNub](https://www.pubnub.com/blog/best-practices-for-claude-code-sub-agents/)
- [Claude Code Advanced Best Practices — SmartScope](https://smartscope.blog/en/generative-ai/claude/claude-code-best-practices-advanced-2026/)

### OpenAI Codex

- [Subagents — Codex / OpenAI Developers](https://developers.openai.com/codex/subagents)
- [Codex Gets Subagents — Spillwave Solutions (Mar 2026)](https://medium.com/@richardhightower/codex-gets-subagents-the-parallel-ai-coding-pattern-is-now-industry-standard-how-does-it-stack-35bd217ef11f)
- [Introducing the Codex app — OpenAI](https://openai.com/index/introducing-the-codex-app/)

### Multi-agent frameworks

- [Best Multi-Agent Frameworks in 2026 — gurusup](https://gurusup.com/blog/best-multi-agent-frameworks-2026)
- [LangGraph vs CrewAI vs AutoGen — DataCamp](https://www.datacamp.com/tutorial/crewai-vs-langgraph-vs-autogen)
- [Multi-Agent AI in 2026 — DEV Community](https://dev.to/ottoaria/multi-agent-ai-in-2026-build-production-systems-with-crewai-langgraph-autogen-5e40)
