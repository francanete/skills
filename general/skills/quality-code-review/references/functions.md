# Functions / Methods

- **Small** — ideally 5–20 lines. Flag functions that require scrolling.
- **Do one thing** — if you can extract another meaningful function, it's doing too much.
- **One level of abstraction** — don't mix orchestration with implementation details.
- **Descriptive names** — long descriptive name > short cryptic one.
- **Few arguments** — 0–2 is fine, 3 is suspicious, 4+ needs justification or grouping into an object.
- **No hidden side effects** — function does only what its name promises.
- **Command-Query Separation** — a function should either change state OR return data, not both.
