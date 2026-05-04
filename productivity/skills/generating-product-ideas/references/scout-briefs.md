# Scout Brief Library

## Contents

- **Core trio (always run)** — product-surface, monetization, UX-friction
- **Focus-area scouts (add when relevant)** — retention, onboarding, mobile, collaboration, admin/billing, marketplace, developer/API, B2B-sales
- **Scoping notes** — how many to run, when to write a custom brief

---

Each scout is a focused `Task` subagent (`Explore` or `codebase-scout`) spawned during Phase 1. All scouts receive the Phase 0b architecture brief as shared context. Each scout reports under 500 words, focused on **user experience and commercial surface — not code quality**. Cite file paths only as evidence.

Compose the swarm dynamically. Always include the **core trio** below; add focus-area scouts when relevant. Run all scouts in parallel in a single assistant turn.

## Core trio (always run)

### Product-surface scout

**Goal:** Understand what the product does for users.

Report on:
- Value loop — what does the user get out of using this?
- Onboarding flow — what happens between sign-up and first value?
- Activation moment — when does the user realize the product is worth keeping?
- Core flows — the 2–4 things users do most often
- Where free or basic users hit walls (rate limits, feature locks, dead ends)
- Main app surfaces (dashboards, workspaces, public pages)

### Monetization scout

**Goal:** Understand the commercial surface.

Report on:
- Pricing tiers and what each unlocks
- What's paywalled and where the paywall sits in the flow
- Upgrade triggers — what causes a free user to consider paying?
- Free-to-paid friction — what makes upgrading feel painful?
- Quota or usage-limit logic
- Billing integration shape (Stripe, Paddle, etc.) — what does it support?

### UX-friction scout

**Goal:** Find the leaks and dead ends.

Report on:
- Empty states — are they helpful or just empty?
- Dead-end flows — places users hit a wall and have to leave
- High-effort tasks — multi-step manual flows that should be automated
- Error-prone forms or interactions
- Places where users must leave the app to complete a task
- Half-built or hinted-at features that suggest abandoned scope

## Focus-area scouts (add when relevant)

### Retention scout (focus: retention)

Replaces or complements UX-friction scout when focus = retention.

Report on:
- Notification / email touchpoints — what gets sent and when?
- Re-engagement surfaces — what brings dormant users back?
- Habit-forming hooks — daily/weekly use cases
- Dormancy handling — what happens when a user disappears for 30/60/90 days?
- Streak / progress / status mechanics

### Onboarding scout (focus: onboarding, activation)

Replaces UX-friction scout when focus = onboarding or activation.

Report on:
- The first 5 minutes after sign-up — what's required vs optional?
- Time-to-first-value — how many steps before the user gets something useful?
- Demo data, templates, or starter content
- Empty-state CTAs in the first session
- Drop-off points in the funnel

### Mobile scout (focus: mobile)

Add when focus = mobile or when product clearly has mobile surface.

Report on:
- Responsive breakpoints and what breaks at small widths
- Touch targets, gesture support
- Mobile-specific surfaces or PWA setup
- Performance on slow connections
- Offline / connectivity handling

### Collaboration scout (focus: collaboration, teams, enterprise)

Add when focus = collaboration or product supports multi-user workflows.

Report on:
- Sharing model — who can see what?
- Permissions and roles — granularity and gaps
- Real-time presence, comments, mentions
- Team / workspace structure
- Invitation flows and friction

### Admin / billing scout (focus: monetization, enterprise)

Deepens the monetization scout for enterprise or billing-heavy products.

Report on:
- Admin panels — what can workspace owners do?
- Invoice / receipt / billing portal completeness
- Self-serve plan changes vs sales-assisted
- Enterprise contracts, SSO, SCIM, audit logs
- Usage analytics for the buyer (not just the user)

### Marketplace scout (focus: marketplace, two-sided products)

Add when product has buyers + sellers, creators + consumers, etc.

Report on:
- Supply-side flow (listing, onboarding sellers/creators)
- Demand-side flow (search, browse, filtering)
- Trust mechanisms (reviews, verification, dispute handling)
- Take-rate / pricing model for supply side
- Liquidity gaps (categories with too few listings or too few buyers)

### Developer / API scout (focus: developer, API, platform)

Add when target user includes developers or product has public API.

Report on:
- API surface — what's exposed, what's not?
- Documentation quality and completeness
- SDK / library availability
- Rate limits, auth model, key management
- Webhook / event surface
- Examples and quick-start friction

### B2B-sales scout (focus: enterprise sales, B2B)

Add when target user is enterprise or sales-led GTM.

Report on:
- Lead capture surfaces and gating
- "Book a demo" / "Talk to sales" flows
- Trial/pilot mechanics (length, conversion friction)
- Comparison/competitor pages
- ROI calculators, security pages, compliance docs

## Scoping notes

- Pick **3–5 scouts max** per run. More than that dilutes synthesis.
- The core trio always runs. Focus-area scouts replace or complement, not stack on top.
- If the user's focus area doesn't match a brief here, write a custom one inline — don't force-fit.
