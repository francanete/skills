# Example Output

A worked end-to-end run for a fictional small B2B SaaS — an invoice-management tool for freelancers ("Billsy"). User invoked `/product-ideas 3 retention` after a 3-scout swarm + 4 web searches + validator pass.

This is what the **final user-visible output** looks like. Phase mechanics, scout reports, and validator briefs do **not** appear — the user sees the ranked cards plus the closing block.

---

### Cross-cutting concerns

- Ideas 1 and 2 both depend on a notification subsystem that doesn't exist yet. Ship a thin notification primitive first or budget extra scope into whichever idea ships first.

---

### Idea 1: Late-payment auto-nudge for clients

**Product change** — When an invoice goes unpaid past its due date, Billsy automatically sends the freelancer's client a polite reminder email on day +3, +7, and +14, with a one-click "pay now" link. Freelancer can mute or customize per-client.
**Commercial angle** — Retention. Freelancers churn when they stop closing invoices through the tool — auto-nudges turn Billsy from a passive ledger into an active money-collector, deepening the value loop.
**Architectural fit** — Slots into the existing invoice status model + Stripe payment links. Needs a scheduled-job runner (currently absent — would piggyback on Vercel cron or a small queue).
**Market position** — Already-shipped-by-FreshBooks, table-stakes-but-missing here.
**Tech difficulty** — M (~1–2 weeks): cron scaffolding + email templating + per-client mute settings.
**Recommended approach** — Add a lightweight scheduled job that scans overdue invoices nightly. Reuse existing email templates. Ship per-client mute as a v2 — start with global on/off.

*Revised after validation: dimension corrected from S to M because no cron infrastructure exists yet.*

---

### Idea 2: Tax-ready year-end export bundle

**Product change** — One-click "year-end export" generates a zip with categorized income CSV, expense summary, and a printable P&L — formatted for the freelancer's accountant or tax software.
**Commercial angle** — Conversion + retention. Freelancers consistently leave Billsy in January to assemble tax paperwork manually elsewhere. Owning the tax moment is sticky annually.
**Architectural fit** — Pure read-side feature. Aggregates existing invoice + expense data into a new export endpoint. No schema changes.
**Market position** — Novel for this price tier (FreshBooks gates this behind their top plan).
**Tech difficulty** — S (~3–5 days): aggregation query + CSV/PDF generation.
**Recommended approach** — Build the CSV first (fastest to ship), add the PDF P&L as a v2. Surface the export prominently in December via the dashboard banner.

---

### Idea 3: Recurring-invoice templates with auto-send

**Product change** — Freelancers can mark any invoice as "recurring monthly/quarterly," and Billsy auto-generates and sends it on the schedule. Editable before send if "review first" is toggled.
**Commercial angle** — Retention + ARPU. Recurring revenue clients are the freelancer's stickiest income — making them effortless to bill keeps the freelancer logged in monthly. Justifies a higher tier.
**Architectural fit** — Builds on the same scheduled-job runner as Idea 1. Adds a `recurrence` field on the invoice model.
**Market position** — Table-stakes-but-missing — Wave, Zoho, FreshBooks all ship this.
**Tech difficulty** — M (~1–2 weeks): schema migration + scheduler + UI.
**Recommended approach** — Ship after Idea 1 so the cron infrastructure already exists. Start with monthly cadence only; add quarterly/custom in v2.

---

### Dropped by validator

- **Slack integration for invoice notifications** — Validator: no Slack OAuth scaffolding exists, would require building auth + bot infra (~3 weeks). Scope way larger than the dimension claimed (S). Revisit when team-account support exists.
- **AI-generated client follow-up email drafts** — Validator: no LLM integration, no API key management, no usage-billing surface. Idea is plausible but reads as "shoehorn AI in" rather than tracing to a real freelancer pain point in the discovery findings.

---

### Suggested ordering

Ship Idea 2 first (smallest, novel, no dependencies), then Idea 1 (introduces the cron primitive), then Idea 3 (reuses cron).

### Noted separately

Discovery surfaced one bug: invoices marked "paid" via the Stripe webhook occasionally double-fire and create duplicate payment records (visible in `stripe-webhook-handler.ts`). Not a product idea — flagging for engineering.

---

Want me to plan any of these in depth?
