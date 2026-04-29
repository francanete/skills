# Classes / Modules

- **Single Responsibility Principle** — one reason to change.
- **High cohesion** — every field used by most methods. If a subset pairs off, that's a second class.
- **Open/Closed** — open for extension, closed for modification.

## Objects and Data Structures

- Internal structure hidden — objects expose behaviour, not fields.
- Law of Demeter — no chained calls like `a.getB().getC().doThing()`.
- Polymorphism over type-checking conditionals (switch/case on a "type" field).
