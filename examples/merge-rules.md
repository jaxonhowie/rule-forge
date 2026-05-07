# merge-rules
<!-- generated: 2026-05-07 | sources: 11 files | base: 28 | spec: 19 -->
<!--
  Sources merged:
  ~/projects/saas-app/CLAUDE.md
  ~/projects/saas-app/.cursorrules
  ~/projects/api-server/CLAUDE.md
  ~/projects/api-server/AGENTS.md
  ~/projects/mobile-app/.cursorrules
  ~/projects/mobile-app/.cursor/rules/typescript.mdc
  ~/projects/data-pipeline/CONVENTIONS.md
  ~/projects/data-pipeline/AGENTS.md
  ~/dotfiles/CLAUDE.md
  ~/.cursor/rules/global.mdc
  ~/.claude/CLAUDE.md
-->

## BASE

- Solve the actual problem; do not add unrequested features.
- Make the smallest change that satisfies the requirement.
- Prefer editing existing files over creating new ones.
- Ask before deleting or overwriting uncommitted work.
- Verify assumptions before acting on them.
- Keep functions under 40 lines; extract if longer.
- Use descriptive names; avoid abbreviations except well-known ones.
- Avoid nested ternaries; use early returns instead.
- Prefer composition over inheritance.
- Delete dead code rather than commenting it out.
- Never commit secrets, tokens, or credentials.
- Validate all input at system boundaries, not internal functions.
- Never run destructive operations without user confirmation.
- Sanitize user input before interpolating into shell commands.
- Run the full test suite before declaring a task complete.
- Show a diff before overwriting any existing file.
- Do not skip pre-commit hooks or linters.
- Write tests for every new public function.
- Test edge cases: empty input, max bounds, null values.
- Use real dependencies in integration tests; avoid mocking internals.
- Name tests as: `should <behavior> when <condition>`.
- Use `camelCase` for variables and functions.
- Use `PascalCase` for types, classes, and components.
- Prefix boolean variables with `is`, `has`, or `can`.
- Write commit messages in imperative mood, under 72 characters.
- Never force-push to `main` or `master`.
- Keep PRs focused; one concern per pull request.
- Squash fixup commits before merging.

## SPEC

- Use TypeScript strict mode; never use `any`.
- Enable `noUncheckedIndexedAccess` in tsconfig.
- Prefer React hooks over class components.
- Co-locate component state; lift only when necessary.
- Use Tailwind utility classes; avoid inline styles.
- Keep Tailwind class lists sorted by category.
- Use pnpm as the package manager; do not use npm or yarn.
- Run `pnpm lint` and `pnpm typecheck` before committing.
- Use Vite for bundling; avoid Webpack configuration.
- Define database schema in Prisma; never write raw DDL.
- Use FastAPI dependency injection for shared resources.
- Apply Pydantic models at all API boundaries.
- Containerize services with Docker; pin base image versions.
- Use Docker Compose for local multi-service development.
- Follow DDD: separate domain logic from infrastructure.
- Keep bounded contexts independent; communicate via events.
- Structure monorepo packages with explicit public APIs.
- Use ESLint with project config; do not disable rules inline.
- Format with Prettier on save; commit only formatted files.
