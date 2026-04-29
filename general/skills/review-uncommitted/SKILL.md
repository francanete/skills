---
name: review-uncommitted
description: Reviews uncommitted git changes for correctness, security, naming, patterns, and TypeScript quality before committing. Lighter and faster than quality-code-review. Use whenever the user mentions "review my changes", "check before I commit", "look at uncommitted", "pre-commit review", "is this ok to commit", or has staged/unstaged changes they want a quick gate on. Make sure to use this skill any time uncommitted work needs a quality pass — do not default to deep architectural review.
allowed-tools: Bash, Read, Glob, Grep
user-invocable: true
context: fork
agent: general-purpose
model: sonnet
effort: medium
---

Pre-commit review of uncommitted changes. Catch real problems quickly; skip deep architectural analysis. Always reviews the full uncommitted diff — any path argument passed by the user is ignored.

## Workflow

1. **Read project conventions**: Check `CLAUDE.md` (project root and `.claude/` if present) to understand project patterns and intentional decisions. Never flag something that aligns with documented conventions.

2. **Gather the diff**: Run `git diff` and `git diff --cached` to see all staged and unstaged changes. Run `git status` to understand which files are affected.

3. **Stop if no diff**: If both `git diff` and `git diff --cached` are empty, say "No uncommitted changes to review." and exit. Do not proceed.

4. **Understand the intent**: Read the full diff and determine:
   - What is this change trying to accomplish? (bug fix, feature, refactor, etc.)
   - Summarize in 1-2 sentences at the top of your review.

5. **Read surrounding context**: For each changed file, read:
   - The full file to understand the change in context
   - Imported modules and types to verify correct usage
   - Sibling files to check pattern consistency
   - Related test files if they exist

6. **Focus on changed lines**: Concentrate your review on what actually changed and its immediate context. Only flag pre-existing issues if they directly affect the correctness of the new changes.

7. **Review checklist** — priority order: security > correctness > TypeScript quality > naming > patterns > style. Evaluate each area and only report issues found:

   **Correctness**
   - Logic is sound and handles edge cases
   - No potential runtime errors (null access, missing awaits, unhandled promise rejections)
   - API contracts are respected (correct return types, proper error responses)
   - No race conditions or timing issues
   - No accidentally removed or broken functionality

   **Security**
   - User input is validated/sanitized before use (form data, URL params, API payloads)
   - No string interpolation in SQL, shell commands, or HTML — use parameterized queries and safe APIs
   - Server actions and API routes check auth before performing operations
   - No secrets, tokens, or PII exposed in client bundles or logs
   - No `dangerouslySetInnerHTML`, `eval()`, or unvalidated `JSON.parse` on user input

   **TypeScript**
   - No `any` — use `unknown` with narrowing or a concrete type
   - No unsafe `as` casts that suppress real type errors
   - No non-null assertions (`!`) that mask potential null/undefined bugs
   - Correct use of generics and type narrowing

   **Naming & Clean Code**
   - Names are descriptive, unambiguous, and follow codebase conventions
   - No dead code, commented-out code, or leftover debug statements (console.log, etc.)
   - No duplicated logic that should be extracted
   - Functions are focused — one clear purpose
   - Imports are clean (no unused imports, correct paths)

   **Patterns & Consistency**
   - Changes follow the same patterns as sibling/surrounding files
   - Error handling matches project conventions
   - File structure and exports match the project's established patterns

   **Tests** (if test files are in the diff)
   - Tests test meaningful behavior, not implementation details
   - Test descriptions are clear
   - Edge cases are covered

8. **Present your review** in this format:

   ```
   ## Pre-commit Review: [1-2 sentence summary of what the changes do]

   ### Files changed
   - `path/to/file.ts` — [brief description of change]

   ### Issues

   [Critical] — must fix before committing (bugs, security, data loss)
   [Warning]  — should fix, real quality issue
   [Nit]      — minor, take it or leave it

   For each issue:
   - **File**: `path/to/file.ts:line`
   - **Problem**: [what's wrong]
   - **Fix**: [concrete code suggestion]

   ### Verdict
   ready / fix-issues-first / needs-rework
   ```

9. If everything looks clean, say so concisely. Don't invent problems.

## Guidelines

- Be direct. No fluff or excessive praise.
- Only flag real issues. Working code that follows project conventions is fine.
- Show actual code fixes, not vague descriptions.
- Never make changes or commits yourself. This is review only.
