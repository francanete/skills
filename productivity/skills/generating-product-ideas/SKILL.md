---
name: generating-product-ideas
description: Generates a ranked list of validated, decision-ready commercial product opportunities (new features, monetization surfaces, retention hooks, UX wins) for the current codebase, with a fresh-context validator grading each idea for feasibility before presenting. Never proposes bug fixes, refactors, or tech debt. Use whenever the user asks for product ideas, growth ideas, feature ideas, monetization ideas, retention ideas, what to build next, opportunities to ship, or types /product-ideas. Make sure to use this skill any time the conversation is about commercial direction or product strategy for this codebase, not engineering quality.
allowed-tools: Read, Grep, Glob, WebSearch, Task
user-invocable: true
disable-model-invocation: false
argument-hint: "[count] [focus area]"
model: opus
effort: high
---

Act as a Product Manager with growth instincts. Surface **commercially valuable product changes** for the current project — not bug fixes, not refactors, not tech debt. Every idea must change what the user experiences or what they're willing to pay, must trace to real evidence in the codebase or market, and must be architecturally feasible.

**Scope**: operates on the current working directory's codebase. If no recognizable project is present (no `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod` / `README.md` / source tree), stop and ask the user to point at one.

## Invocation

Parse `$ARGUMENTS`:
- First numeric token = count (default **3**)
- Remaining text = optional focus area (freeform — e.g. "retention", "monetization", "activation", "onboarding", "admin ux", "mobile", "collaboration", "enterprise sales")

Examples:
- `/product-ideas` → 3 ideas, broad commercial scan
- `/product-ideas 5` → 5 ideas
- `/product-ideas retention` → 3 ideas biased toward retention
- `/product-ideas 5 activation` → 5 ideas biased toward activation

## Hard rules

1. **Never propose bug fixes, refactors, tech debt, performance tuning, test coverage, or "clean up X" items.** If an obvious bug surfaces during discovery, mention it once in a "noted separately" footer — do not count it as an idea.
2. Every idea must answer **all three**: what changes for the user, why it makes commercial sense (conversion / retention / ARPU / activation / expansion / virality / referrals), and how it fits the existing architecture.
3. Stop at validated proposals. **Never implement.** Never write code. Never plan in depth unless the user explicitly asks after seeing results.
4. Output must be compact and decision-ready. No implementation detail, no file paths unless critical, no code snippets.
5. **Never present an idea the validator dropped.** Surface dropped ideas in a separate section with the validator's reason.

## Pipeline — run in order

### Phase 0 — Context gate + architecture brief

Two parts. Both must complete before discovery.

**0a. Business context gate.** Establish (don't assume):
- **Business stage** — pre-revenue / early revenue / scaling / mature
- **Target user** — consumer / SMB / enterprise / developer / internal / mixed
- **Current monetization** — free / freemium / paid / ads / marketplace / unsure

Try to infer from `package.json`, `README.md`, top-level routes, pricing pages, and obvious billing integrations (Stripe, Paddle, etc.). Then **echo the inferred context back to the user in one block and ask for confirmation or correction** before continuing. If signals are absent or ambiguous, ask directly. Do not guess silently — ideation built on the wrong stage/audience is the most common failure mode.

**0b. Architecture brief.** Spawn one `Explore` subagent to map:
- Tech stack (framework, language, major libraries, deployment target)
- Major surfaces (pages/routes, dashboards, public APIs, admin areas)
- Data model (top-level entities, relationships, source of truth)
- Integration points (auth provider, payments, email, analytics, third-party APIs)
- Rough scale signals (component count, route count, presence of tests, CI)

The brief becomes the shared foundation for every downstream phase. Keep it under 400 words.

**0c. Early-stage guard.** If the architecture brief shows < ~20 components, no real product surface, or "barely scaffolded" signals, stop and tell the user: "This codebase is too early for grounded product ideation — recommend running this again after MVP. Want generic market-driven suggestions instead?" Do not proceed with the full pipeline unless they confirm.

### Phase 1 — Product discovery swarm (parallel)

Spawn focused scouts in a **single message with multiple `Task` calls** so they run in parallel. All scouts receive the architecture brief from Phase 0b.

Pick scouts using [references/scout-briefs.md](references/scout-briefs.md) — always run the core trio (product-surface, monetization, UX-friction); add focus-area scouts (retention, onboarding, mobile, collaboration, admin/billing, marketplace, developer, B2B-sales) based on the user's focus argument. Cap at 3–5 scouts total; more dilutes synthesis.

Each scout reports **under 500 words**, focused on user experience and commercial surface, not code quality. Citations are evidence, not the main output.

### Phase 2 — Market + competitor scan

Run **3–5 targeted `WebSearch` calls** based on the product domain identified in Phase 0b and the focus area if provided. Goals:

1. Understand category norms — what do leaders ship?
2. **Get a verdict per candidate angle: novel, table-stakes, or already done by direct competitors.**

Example searches:
- "[product category] [focus area] best practices 2026"
- "competitors to [closest analog] feature comparison"
- "how does [direct competitor] handle [specific flow]"

Each idea later must reference whether it's novel / table-stakes-but-missing / already-shipped-by-X. "Already shipped by X" doesn't auto-disqualify (table-stakes parity matters), but it must be explicit, not silent.

### Phase 3 — Ideation

Synthesize candidates from the architecture brief + discovery + market scan. Generate roughly 2× the target count, then prune to N.

**Rank by: (user value × commercial upside) ÷ complexity** — rough qualitative weighing, not a numeric formula.

- **User value** — solves real friction or unlocks a real use case (must trace to a discovery finding).
- **Commercial upside** — moves conversion / retention / ARPU / activation / expansion / virality / referrals.
- **Complexity** — based on the architecture brief, not vibes. UI polish is cheap; new backend service is expensive; anything touching billing or auth is medium+.

Mix categories — don't cluster:
- **Extend an existing feature** — deepening compounds value more reliably than net-new
- **New feature unlocking a monetization surface** — new tier, add-on, expansion path, usage-based hook
- **Smooth a conversion-killing friction** — activation or trial-to-paid wins
- **Add a retention hook** — re-engagement, notifications, habit-forming surfaces
- **Close a competitor gap** — from market research

Each candidate must carry an **evidence trail**: which scout findings + which market signals back it. Ideas with no trail get cut before validation.

### Phase 4 — Independent viability validator (fresh context)

Spawn **one** `Task` subagent in fresh context — `general-purpose` for general feasibility, `Plan` if proposals lean architecturally heavy. The validator must run with no prior conversation context — it reads the brief cold and cross-checks the codebase itself.

Compose the brief using [assets/validator-brief.md](assets/validator-brief.md). It must include:

- Business context from Phase 0a
- Architecture brief from Phase 0b
- All candidate ideas with evidence trails
- **Specific files/paths to inspect per idea** — drawn from scout citations. Without these, the validator hallucinates feasibility verdicts instead of verifying.

The validator returns a structured per-idea verdict (KEEP / REVISE / DROP) with concerns across architecture feasibility, evidence grounding, commercial plausibility, competitor accuracy, and dimension correction — plus cross-cutting concerns spanning multiple ideas. See the brief template for the full rubric.

### Phase 5 — Apply validator + output

Apply the validator's verdicts before presenting:

- **DROP** ideas → remove from the ranked list, surface in a "Dropped by validator" section with the reason.
- **REVISE** ideas → apply suggested revisions, present the revised version with a one-line "Revised after validation: <change>" tag.
- **KEEP** ideas → present as-is.

If cross-cutting concerns exist, surface them in a single block above the ideas.

## Output — present to the user

Use the format defined in [assets/idea-card-template.md](assets/idea-card-template.md) — one idea card per validated idea (ranked, highest score first), then the closing block (cross-cutting concerns / dropped ideas / suggested ordering / noted separately / verbatim final ask).

For a worked end-to-end example, see [assets/example-output.md](assets/example-output.md).

## What NOT to do

- Do not write code
- Do not start implementing
- Do not produce file-level implementation plans
- Do not propose technical improvements dressed up as product ideas ("refactor the auth system" is not a product idea; "let users sign in with SSO" is)
- Do not propose ideas with no evidence trail — every idea traces to discovery, market scan, or both
- Do not skip the context gate — wrong stage/audience is the #1 failure mode
- Do not skip the validator — it catches arch hallucinations and ungrounded ideas
- Do not silently drop validator-rejected ideas — always surface them with reason
- Do not mention this skill file, the phases, or the process in the final output (besides the validator-rejected section) — the user sees ranked ideas and validator findings only

## Expected runtime

Expect 3–5 long legs (discovery swarm, web searches, validator). Parallelize Phase 1 aggressively — the validator is one call by design, no need to parallelize.
