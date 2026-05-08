# Project: saas-app (Node.js)
# Stack: TypeScript · Express · PostgreSQL
# Agent: GitHub Copilot
# Scope: base

## BASE

- Solve the actual problem; do not add unrequested features.
- Make the smallest change that satisfies the requirement.
- Prefer editing existing files over creating new ones.
- Ask before deleting or overwriting uncommitted work.
- Keep functions under 40 lines; extract if longer.
- Use descriptive names; avoid abbreviations except well-known ones.
- Avoid nested ternaries; use early returns instead.
- Delete dead code rather than commenting it out.
- Never commit secrets, tokens, or credentials.
- Validate all input at system boundaries, not internal functions.
- Never run destructive operations without user confirmation.
- Run the full test suite before declaring a task complete.
- Show a diff before overwriting any existing file.
- Do not skip pre-commit hooks or linters.
- Write tests for every new public function.
- Use real dependencies in integration tests; avoid mocking internals.
- Write commit messages in imperative mood, under 72 characters.
- Never force-push to `main` or `master`.
- Keep PRs focused; one concern per pull request.
