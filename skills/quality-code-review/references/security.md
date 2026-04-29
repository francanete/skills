# Security

- **Input validation** — user input, URL params, form data, and external API responses are validated/sanitized before use.
- **Injection** — no string interpolation in SQL, shell commands, or HTML. Use parameterized queries, proper escaping, or safe APIs.
- **Auth boundaries** — server actions and API routes check authentication and authorization before performing operations.
- **Sensitive data** — no secrets, tokens, or PII logged or exposed in client bundles. Environment variables used correctly.
- **Unsafe casts** — flag `dangerouslySetInnerHTML`, `eval()`, `Function()`, or unvalidated `JSON.parse` on user input.
