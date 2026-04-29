# Review output template

Use this exact format when presenting the review.

```
## Clean Code Review: [1–2 sentence summary of what was reviewed]

### Scope
- [List of files reviewed]
- Conventions checked: [CLAUDE.md found / not found]

### Issues

🔴 **[Critical]** — violates a core principle, likely to cause bugs or serious maintenance pain
🟡 **[Warning]** — meaningful clean code violation worth fixing
🔵 **[Suggestion]** — minor improvement opportunity

For each issue:
- **Principle**: [which clean code principle is violated]
- **File**: `path/to/file.ts:line`
- **Problem**: [what's wrong]
- **Fix**: [concrete code suggestion]
- **Blast radius**: [high — shared util/core module | medium — feature module | low — leaf component/helper]

### Summary
- Total issues: X critical, Y warnings, Z suggestions
- Cleanest file: `file.ts` — [why]
- Most attention needed: `file.ts` — [why]

### Verdict
✅ Clean / ⚠️ Needs cleanup / ❌ Needs significant rework
```
