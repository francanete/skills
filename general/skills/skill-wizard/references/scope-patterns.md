# Scope handling patterns

For skills that operate on files or git state, decide the supported argument shapes upfront. Bake the decision into both `argument-hint` and the SKILL.md body so Claude never silently picks the wrong scope.

## Pattern A — Single-file skills

```
$ARGUMENTS = path/to/file.ts → operate on that file
$ARGUMENTS empty             → ask the user, or error out
```

Frontmatter:
```yaml
argument-hint: "[file-path]"
```

## Pattern B — File-set skills (review, refactor, audit)

Three modes, **default = branch**:

| Input | Scope |
|---|---|
| `$ARGUMENTS` is a file path | That single file |
| User mentions "uncommitted", "staged", "working changes" | `git diff` + `git diff --cached` |
| Default / "branch" / "PR" | `git diff $(git merge-base HEAD main)...HEAD` |

Implementation:
```bash
BASE=$(git merge-base HEAD main)
git diff --name-only $BASE...HEAD
git diff $BASE...HEAD
```

If the chosen mode produces an empty diff, **stop and say so**. Don't invent content.

## Pattern C — Glob / directory

Avoid unless necessary. If supported:
- `$ARGUMENTS = "src/auth/"` → list files in that dir
- `$ARGUMENTS = "src/**/*.ts"` → glob expansion via the `Glob` tool

State the supported shapes explicitly in `argument-hint` and reject ambiguous input.

## Pattern D — Multi-arg skills

If the skill needs multiple positional args (file + format, source + target):

```yaml
arguments: [file, format]
argument-hint: "[file] [format]"
```

Reference them as `$file` and `$format` in the body.

## Anti-pattern: silent fallthrough

Don't let your skill silently produce empty output when no input matches. **Always declare the chosen mode at the start of execution** and stop with a clear message if there's nothing to operate on:

> "No mode matched. Detected scope: branch. Branch is identical to main — nothing to review."

This is the difference between "I checked and there's nothing" (good) and "I produced empty output" (bug).

## Choosing among patterns

- Skill always operates on one file? → Pattern A.
- Skill reviews/audits a set of changes? → Pattern B (default branch is almost always what users want).
- Skill operates on a logical area of code? → Pattern C (only if you've thought through the glob semantics).
- Skill takes structured inputs? → Pattern D.
