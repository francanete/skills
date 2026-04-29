# Audience guide

How to tune explanation depth and style per audience. Pick one in phase 1; let it shape every section that follows.

## newcomer to this codebase

The reader has never seen this code and may be unfamiliar with its domain.

- **More of:** plain-language overviews, named entry points, glossary entries, "why does this exist" framing, simple architecture diagrams.
- **Less of:** internal tradeoffs, micro-optimization rationale, edge-case enumeration, jargon without definition.
- **Diagrams:** lean on the architecture graph. Sequence diagrams are useful but keep them short (≤6 actors).
- **Tone:** define the first use of any domain term inline. Assume general programming literacy, not codebase familiarity.

## experienced dev, going deep

The reader knows the language and codebase but wants the real story of this slice.

- **More of:** internals, invariants, tradeoffs, alternatives considered, performance notes, edge cases, version-specific behaviors.
- **Less of:** beginner framing, glossary, "what is a service" explanations.
- **Diagrams:** sequence diagram for non-trivial flows; state diagram if there's a real machine. Architecture diagram only if the slice spans multiple components.
- **Tone:** terse and technical. Cite files with line numbers. Say "we" only if the reader is on the team.

## I'm about to modify this

The reader will touch this code soon and needs to land changes safely.

- **More of:** hot spots, callers/consumers, blast radius, fragile contracts, untested branches, migration concerns, tests that will need updating.
- **Less of:** historical "why we built it" context (unless it constrains current changes), glossary.
- **Diagrams:** prioritize the architecture graph annotated with "modify-with-care" zones, plus a sequence diagram of the flow they're touching.
- **Tone:** action-oriented. Every gotcha should imply "do/don't do this when you change it."
- **Always include:** the `Hot spots if you're modifying this` section.

## freeform description

User specified something custom (e.g., "for a security review", "for a perf audit", "for a junior on my team but they know React").

- Mirror the user's framing back, then pick the closest preset above as a base and tilt it per the custom angle.
- Confirm the tilt with the user before drafting if it materially changes structure (e.g., a security framing demands a threat-model section the template doesn't have).
