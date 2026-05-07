# Project: saas-app (Node.js)
# Stack: TypeScript · Express · PostgreSQL
# Agent: Claude Code

## General
- Solve the actual problem; do not add unrequested features.
- Make the smallest change that satisfies the requirement.
- Prefer editing existing files over creating new ones.
- Ask before deleting or overwriting uncommitted work.
- Verify assumptions before acting on them.

## Code Quality
- Keep functions under 40 lines; extract if longer.
- Use descriptive names; avoid abbreviations except well-known ones.
- Avoid nested ternaries; use early returns instead.
- Prefer composition over inheritance.
- Delete dead code rather than commenting it out.
- Never use `any` in TypeScript; use `unknown` with type guards.
- Prefer immutable data; avoid mutations on function arguments.
- Limit a file to one primary responsibility.

## Safety
- Never commit secrets, tokens, or credentials.
- Validate all input at system boundaries, not internal functions.
- Never run destructive operations without user confirmation.
- Sanitize user input before interpolating into shell commands.
- Prefer read-only operations when exploring unfamiliar codebases.

## Workflow
- Run the full test suite before declaring a task complete.
- Check existing tests pass before writing new ones.
- Show a diff before overwriting any existing file.
- Prefer reversible actions; flag irreversible ones explicitly.
- Do not skip pre-commit hooks or linters.

## Testing
- Write tests for every new public function.
- Test edge cases: empty input, max bounds, null values.
- Use real dependencies in integration tests; avoid mocking internals.
- Name tests as: `should <behavior> when <condition>`.
- Keep unit tests isolated with no I/O or network calls.

## Naming
- Use `camelCase` for variables and functions.
- Use `PascalCase` for types, classes, and components.
- Use `SCREAMING_SNAKE_CASE` for constants and env vars.
- Prefix boolean variables with `is`, `has`, or `can`.

## Git
- Write commit messages in imperative mood, under 72 characters.
- Never force-push to `main` or `master`.
- Keep PRs focused; one concern per pull request.
- Squash fixup commits before merging.
