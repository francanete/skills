# Verification protocol

How phase 3 of `explain-code` invokes the verifier and surfaces results.

## Why a separate agent

Explanations of code are uniquely prone to confident hallucination — paraphrased function names, slightly-wrong call chains, plausible-but-fake file paths. The verifier must read the .md cold to catch what the explainer missed. If it inherits the conversation, it inherits the explainer's blind spots. Treat this isolation as load-bearing.

## How to invoke

Spawn `fran-general:explain-verifier` via the `Agent` tool. The prompt must contain ONLY two facts:

1. The absolute path to the explanation .md
2. The repo root (the working directory the verifier should investigate against)

### Exact prompt template

```
Verify the explanation at <ABSOLUTE_PATH_TO_EXPLANATION_MD>.
Repo root: <ABSOLUTE_PATH_TO_REPO_ROOT>.

Read the explanation cold. Fact-check every concrete claim against the codebase. Return your structured findings report per your output spec.
```

That is the entire prompt. Do NOT include:

- Conversation history with the user
- Your own summary of what the explanation says
- Hints about what to look for ("watch out for X", "the diagrams might be off")
- Reassurance ("the user already knows about Y")
- The audience tag (the .md frontmatter already carries it)

Each of these defeats the purpose.

## How to surface findings

When the verifier returns:

1. Print a `## Verification findings` header.
2. Reproduce the verifier's report verbatim. Do not edit, summarize, or re-rank.
3. Below the report, ask: *"Want me to address any of these?"*

If the verifier returns "no findings", say so plainly — don't pad.

## When to skip phase 3

Only when the user explicitly opts out (e.g., "skip verification, just write the explainer"). Note in the chat that verification was skipped and leave `status: draft` in the frontmatter so the user remembers it's unverified.

## Failure modes

- **Verifier reads the wrong file** — confirm the absolute path you passed.
- **Verifier returns shallow findings** — likely the explanation was too vague for cross-referencing. Push the user to make phase 2 more concrete on the next iteration.
- **Verifier and user disagree on a finding** — surface the disagreement; don't suppress either side. The user decides.
- **Diagram-only errors** — mermaid blocks are still claims. The verifier must trace the depicted flow against actual code.
