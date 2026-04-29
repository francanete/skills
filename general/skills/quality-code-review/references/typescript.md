# TypeScript Quality

- Flag `any` usage — prefer `unknown` with proper narrowing, or a concrete type.
- Flag unsafe `as` casts that suppress real type errors instead of fixing them.
- Flag missing return types on exported functions and public APIs.
- Flag non-null assertions (`!`) that mask potential null/undefined bugs.
- Prefer discriminated unions over type-checking strings.
