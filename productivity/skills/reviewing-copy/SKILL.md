---
name: reviewing-copy
description: Reviews and rewrites marketing copy using Gary Halbert's direct-response principles, evaluating headlines, CTAs, hero sections, value propositions, and landing pages against AIDA/PAS/BAB frameworks and Cialdini's persuasion principles. Use whenever the user mentions copy, headlines, landing pages, CTAs, hero sections, value props, conversion copy, marketing copy, or asks to improve, review, critique, or rewrite anything customer-facing. Make sure to use this skill any time copy quality, persuasion, or conversion comes up — do not default to generic writing advice.
allowed-tools: Read, Glob, Grep, Edit, Write, Agent
user-invocable: true
argument-hint: "[file-path or section-name]"
model: sonnet
effort: high
---

Apply Gary Halbert's direct-response copywriting principles (with Cialdini, Kennedy, Schwartz) to review and rewrite the user's marketing copy. Stay in the main conversation so the user can iterate on rewrites.

## Determine scope

- **File path argument** → read and review that file's user-facing strings.
- **Section name** (e.g., "hero", "pricing", "CTA") → grep for the relevant component(s) and review.
- **Site-wide audit** ("review the whole marketing site") → spawn an `Explore` subagent to map user-facing copy first, then review the findings here. Do not grep the whole repo from the main thread.
- **No argument** → ask the user what to review: a specific page, component, section, or the whole site.

Focus on `app/**/*.{tsx,jsx,mdx}`, `pages/**/*.{tsx,jsx}`, `content/**/*.md*`, and components containing user-facing strings. Skip tests, types, and config.

## Review process

For each piece of copy, evaluate against these dimensions in order. Stop early if the offer or audience is broken — words can't fix that.

### 1. Market & offer fit (the 40/40/20 check)

- Does the copy speak to a specific, identifiable audience already feeling the pain?
- Is the offer itself compelling — specific promise, credible reason why, risk reversed?
- If either is weak, name it. Rewriting words won't fix it. See [references/halbert-rules.md](references/halbert-rules.md) for the 40/40/20 rule and starving-crowd principle.

### 2. Headline / hook

- Does it stop the reader? Curiosity, dramatic benefit, bold promise.
- 80% of people only read the headline — it deserves disproportionate attention.

### 3. Slippery slide

- Does every line earn the next? Where would a reader drop off?
- Headline → first line → first paragraph → section → CTA. Find the leaks.

### 4. Framework fit

Pick the right structure for the format — see [references/frameworks.md](references/frameworks.md):

- **AIDA** — long-form sales pages, multi-section landing pages
- **PAS** — short copy, ads, cold outreach, hero sections
- **BAB** — case studies, transformation pitches, story-driven copy

### 5. Persuasion levers

Check which of Cialdini's 7 principles are present and whether they're stacked well — see [references/cialdini-principles.md](references/cialdini-principles.md). Most pages use 1–2 weakly; great pages stack 3–4 naturally.

### 6. Proof & specificity

- Are claims backed by specific evidence?
- "Lost 23.5 pounds in 47 days" beats "lost weight". Vague claims read as marketing; specific claims read as facts.

### 7. CTA

- Crystal clear next step. Restated benefit. Risk-free. Frictionless.
- Button copy specifically: action verb + benefit, not "Submit" or "Click here".

### 8. Writing quality

- Could a 12-year-old understand this? Cut jargon.
- Active voice, varied sentence length, no "I think" / "in my opinion".
- Read it aloud — if you'd never say it to a human, rewrite it.
- Full language rules in [references/halbert-rules.md](references/halbert-rules.md).

## Output

Use the structure in [assets/review-template.md](assets/review-template.md):

1. **Overall Assessment** — verdict in 2–3 sentences
2. **What's Working** — specific elements + which principle they leverage
3. **Priority Improvements** — ordered by impact, with Issue / Current / Suggested / Why
4. **Quick Wins** — small tweaks
5. **Rewritten Version** — only if the user asks for a full rewrite

Quote the existing copy verbatim before suggesting changes. Always name the principle behind each rewrite.

## Iteration

After delivering the review, stay ready for follow-ups: "make headline #2 punchier", "shorter CTA", "try a B2B angle". Don't re-run the full review on every iteration — apply the same principles to the specific request.

## Key reminders

- **People buy on emotion, justify with logic.** Lead with feeling, follow with facts.
- **Pain pulls harder than pleasure.** Loss aversion is real.
- **You are not your customer.** Write from the reader's perspective.
- **Clear beats clever.** If they have to think for a second, you've lost.
- **The best copy can't save a bad offer.** Always check the offer first.
