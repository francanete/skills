# Audit report template

Use this format when reporting an audit to the user. Wait for approval before applying any change.

```markdown
# Audit: <skill-name>

## Strengths
- <What the skill gets right>
- <Frontmatter fields already correct>
- <Layout decisions that work>

## Gaps (prioritized)

### 🔴 Critical — fix before next use
- **<Gap title>**: <one-line problem>
  - **Why it matters**: <token cost / trigger reliability / output quality / silent failure>
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
- <Things the audit flags but the user should explicitly decide on, e.g., adding examples.md, eval suite>

---

**Apply these changes?** Reply with "apply all", "apply 🔴 only", a numbered subset, or "skip <item>".
```

## Audit report rules

- Lead with strengths. Audits feel adversarial otherwise.
- Every gap needs a concrete fix — never just "this is wrong".
- Prioritize by leverage, not by length of fix. A pushy description rewrite beats a 300-line content reorg.
- Never silently apply changes. Always wait for approval.
