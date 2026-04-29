# SKILL.md template

Copy this and fill in the placeholders. Delete sections you don't need. Keep the body ≤500 lines.

```markdown
---
name: <gerund-kebab-name>
description: <one-line action statement>. <Trigger phrases — what the user would say>. Use this skill whenever the user mentions <topic-1>, <topic-2>, or <topic-3>, even if they don't explicitly say "<keyword>".
allowed-tools: <minimal tool set, e.g., Read, Grep, Bash, Agent>
user-invocable: true
argument-hint: <[args]>
context: fork                    # delete if skill is light-weight
model: <sonnet | opus | haiku>
effort: <medium | high>
# disable-model-invocation: true # ONLY if the skill has real-world side effects
---

You are a <role>. <One-sentence purpose statement.>

## Determine scope

<If the skill operates on files, define mode detection here. See scope-patterns.md.>

## Workflow

1. **<Step 1 verb>**: <action>.
2. **<Step 2 verb>**: <action>.
   - For <domain X>, see [references/<x>.md](references/<x>.md)
   - For <domain Y>, see [references/<y>.md](references/<y>.md)
3. **<Step 3 verb>**: <action>.

4. **Format the output** using [assets/<output>-template.md](assets/<output>-template.md).

5. <If applicable: stop / wait for user / iterate condition.>

## Guidelines

- <Specific rule 1>
- <Specific rule 2>
- <Anti-pattern to avoid>
- Apply principles as heuristics, not dogma. Note when a trade-off is reasonable.
```
