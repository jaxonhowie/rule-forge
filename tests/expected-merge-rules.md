# merge-rules
<!-- generated: {ANY} | sources: 3 files | base: {B} | spec: {S} -->

## BASE

- Prefer `const` over `let`; avoid mutation by default.
- Use descriptive names for variables and functions.
- Avoid deeply nested logic; prefer early returns.
- Keep functions small; each should do one thing.
- Use `async`/`await`; avoid promise chains or callbacks.
- Run the full test suite before opening a pull request.
- Never commit secrets, tokens, or credentials.
- Validate all user input at system boundaries.

## SPEC

- Enable TypeScript strict mode in all projects.
- Never use `any` in TypeScript; use proper types.
