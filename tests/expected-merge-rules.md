# merge-rules
<!-- generated: {ANY} | sources: 3 files | rules: {ANY} -->

## General
- Prefer `const` over `let`; avoid mutation by default.
- Use descriptive names for variables and functions.
- Avoid deeply nested logic; prefer early returns.

## Code Quality
- Keep functions small; each should do one thing.
- Use `async`/`await`; avoid promise chains or callbacks.
- Never use `any` in TypeScript; use proper types.
- Enable TypeScript strict mode in all projects.

## Safety
- Never commit secrets, tokens, or credentials.
- Validate all user input at system boundaries.

## Testing
- Write tests for every public function.
- Run the full test suite before opening a pull request.
