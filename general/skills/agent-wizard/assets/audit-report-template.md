# Audit report template

Use this format when reporting an audit to the user. Wait for approval before applying any change.

```markdown
# Audit: <agent-name>

## Strengths
- <What the agent gets right>
- <Frontmatter fields already correct>
- <Tool restrictions, model choice, output format that work>

## Gaps (prioritized)

### 🔴 Critical — fix before next use
- **<Gap title>**: <one-line problem>
  - **Why it matters**: <auto-spawn reliability / token cost / output quality / silent failure>
  - **Fix**: <concrete change, with exact frontmatter or content if applicable>

### 🟡 Worth fixing
- **<Gap title>**: <one-line problem>
  - **Why**: <impact>
  - **Fix**: <concrete change>

### 🔵 Nice to have
- **<Gap title>**: <one-line problem>
  - **Fix**: <concrete change>

## Recommended apply order
1. <Highest-leverage change first>
2. <Next>
3. <…>

## Items to defer
- <Things the audit flags but the user should explicitly decide on (e.g., adding `memory:`, splitting into multiple agents)>

---

**Apply these changes?** Reply with "apply all", "apply 🔴 only", a numbered subset, or "skip <item>".
```

## Audit report rules

- Lead with strengths.
- Every gap needs a concrete fix — never just "this is wrong".
- Prioritize by leverage, not by length of fix.
- Never silently apply changes. Always wait for approval.
