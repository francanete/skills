# Agent .md template

Copy this and fill in the placeholders. Delete fields you don't need. Keep the body ≤2,000 words.

```markdown
---
name: <kebab-case-name>
description: <Third-person what + when. Use proactively when [topic-1], [topic-2], or [topic-3]. Make sure to delegate to this agent whenever the user mentions [trigger phrases], even if they don't explicitly say "[keyword]".>

# --- Tools ---
tools: <Read, Grep, Bash>          # ALLOWLIST. Minimal set.
# disallowedTools: Write, Edit     # OPTIONAL — denylist applied first if both set

# --- Behavior ---
model: <sonnet | opus | haiku>
effort: <medium | high>
# maxTurns: 20                     # OPTIONAL — cap on agentic loop iterations

# --- Composition (optional) ---
# skills:                          # preload foundational knowledge
#   - <skill-name>
# mcpServers:                      # scope MCP tools to this agent
#   - <server-name>

# --- Persistence (optional) ---
# memory: <user | project | local> # for cross-session learning
# isolation: worktree              # run in isolated git worktree

# --- UI ---
color: <red | blue | green | yellow | purple | orange | pink | cyan>
---

You are a <role>. <Mission statement.>

## Your job

<What this agent does, scope boundaries, what it does NOT do.>

## Workflow

1. <Step 1>
2. <Step 2>
3. <Step 3>

## Output format

Return your findings in this exact structure:

```
# <Output title>

## <Section 1>
<What goes here>

## <Section 2>
<What goes here>
```

## Rules

- <Specific behavioral rules>
- <Anti-patterns to avoid>
- <Stay focused — what's out of scope>
```
