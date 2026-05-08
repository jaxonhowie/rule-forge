---
description: Discover all agent rule files on this machine, merge and deduplicate into ~/.claude/merge-rules.md, then generate agent-specific rules on project init. Supports Claude Code, Cursor, Windsurf, Cline, Copilot, Gemini CLI, Aider, and OpenAI Codex.
---

# skill: merge-rules

## Commands

| Command | Action |
|---|---|
| `/merge-rules` | Incremental check → merge if changed → write `~/.claude/merge-rules.md` |
| `/merge-rules --refresh` / `-r` | Force full re-scan (picks up newly added files) |
| `/merge-rules --test` / `-t` | Same, but **include** rule files inside test directories |
| `/merge-rules init` | Apply to current project (default `--scope base`) |
| `/merge-rules init --scope base\|spec\|all` / `-s` | Inject BASE, SPEC, or all rules |
| `/merge-rules init --agent <name>` | Force a specific agent target |
| `/merge-rules status` | Show source count, BASE/SPEC rule counts, last merge date (read-only) |

## Supported Agents

| Agent | Rule File(s) | Format |
|---|---|---|
| Claude Code | `CLAUDE.md` | Markdown |
| OpenAI Codex CLI | `AGENTS.md` | Markdown |
| Cursor | `.cursorrules` · `.cursor/rules/*.mdc` | Plain text / MDC frontmatter |
| Windsurf | `.windsurfrules` | Plain text |
| Cline | `.clinerules` | Plain text |
| GitHub Copilot | `.github/copilot-instructions.md` | Markdown |
| Gemini CLI | `GEMINI.md` | Markdown |
| Aider | `CONVENTIONS.md` | Markdown |

---

## Phase 0 — Arguments & OS

Flag table:

| Flag | Alias | Default | Used in |
|---|---|---|---|
| `--refresh` | `-r` | off | Phase 1: force full scan |
| `--test` | `-t` | off | Phase 1: include test directories |
| `--scope base\|spec\|all` | `-s` | `base` | Phase 4: rule scope filter |
| `--agent <name>` | — | auto | Phase 4: target agent |

Run `uname -s 2>/dev/null || echo Windows` to detect OS; on Windows use PowerShell equivalents for all commands.

---

## Phase 1 — Discovery

**Manifest path:** `~/.claude/merge-rules.manifest.json` (Windows: `%APPDATA%\Claude\`)

**Manifest format:** `{ "generated": "ISO-DATE", "sources": { "/path/to/file": mtime_int } }`

**Decision:**
- Manifest missing or `--refresh`/`-r` set → **full scan**
- Otherwise → **cached check** (compare mtime for known paths only; cannot find new files)

**Full scan target filenames:**
`CLAUDE.md` `AGENTS.md` `GEMINI.md` `CONVENTIONS.md` `.cursorrules` `.windsurfrules` `.clinerules` `copilot-instructions.md` `.cursor/rules/*.mdc`

**Always excluded (all modes):**
`$PWD` (current dir) · `node_modules/` · `.git/` · `vendor/` · `dist/` · `build/` · `__pycache__/`

**Default extra exclusions (removed with `--test`/`-t`):**
`tests/` · `test/` · `__tests__/` · `fixtures/` · `spec/` · `specs/`

**Cached check result:**
- No changes → print notice and exit (suggest `--refresh` to scan for new files)
- Changes found → print changed list, use manifest paths as discovered_files, continue to Phase 2

---

## Phase 2 — Merge Pipeline

1. **Extract**: collect bullet points, numbered items, bold directives; strip MDC frontmatter; strip project-specific values (paths, repo names, package names)
2. **Normalize** *(comparison only)*: lowercase, remove punctuation noise
3. **Deduplicate**: exact match → keep one; ≥85% token overlap → keep more concise variant
4. **Merge similar**: same-topic rules → one imperative sentence, preserving union of constraints
5. **Conflict resolution**: safer > more general > more specific
6. **Compress**: short imperative sentence, no explanations/rationale/examples, max 12 words; for plain-text agents (Cursor legacy / Windsurf / Cline), strip backticks, `**`, and other Markdown formatting
7. **Classify (BASE vs SPEC)**: lowercase rule, any keyword match → SPEC; otherwise → BASE

| Category | Keywords |
|---|---|
| Languages | `typescript` `javascript` `python` `rust` `go` `java` `kotlin` `swift` |
| Frameworks | `react` `nextjs` `next.js` `vue` `svelte` `fastapi` `nestjs` `express` `django` `spring` |
| Tooling | `pnpm` `npm` `yarn` `webpack` `vite` `eslint` `prettier` `tailwind` `prisma` `docker` `kubernetes` |
| Architecture | `ddd` `cqrs` `microservice` `monorepo` `mvc` |

---

## Phase 3 — Write merge-rules.md

Output format:

```markdown
# merge-rules
<!-- generated: {ISO-DATE} | sources: {N} files | base: {B} | spec: {S} -->

## BASE
- ...

## SPEC
- ...
```

Constraints: BASE ≤ 40 rules, SPEC ≤ 30 rules; no absolute paths, tokens, or personal data; rules must be agent-agnostic.

After writing, immediately write all discovered file mtimes to the manifest.

---

## Phase 4 — Project Init (`init`)

### 4.1 Detect agent (`--agent` takes precedence; otherwise first match wins)

| Signal | Agent |
|---|---|
| `.cursor/rules/` or `.cursorrules` exists | Cursor |
| `.windsurfrules` exists | Windsurf |
| `.clinerules` exists | Cline |
| `.github/copilot-instructions.md` exists | GitHub Copilot |
| `GEMINI.md` exists | Gemini CLI |
| `CONVENTIONS.md` exists and no other signals | Aider |
| `AGENTS.md` exists and no `CLAUDE.md` | OpenAI Codex |
| `CLAUDE.md` exists or no signal found | Claude Code |

### 4.2 Detect project type

Check for: `package.json` · `Cargo.toml` · `pyproject.toml` · `go.mod` · `Gemfile` · `pom.xml` · `build.gradle`

### 4.3 Filter by scope and render

Load merge-rules.md (run Phase 0–3 first if missing), filter by scope, write to target file.

| Agent | Output file | Format |
|---|---|---|
| Claude Code | `CLAUDE.md` | Markdown, grouped headers |
| OpenAI Codex | `AGENTS.md` | Markdown, grouped headers |
| Cursor (legacy) | `.cursorrules` | Plain text, no headers |
| Cursor (modern) | `.cursor/rules/project.mdc` | MDC frontmatter + Markdown |
| Windsurf | `.windsurfrules` | Plain text, no headers |
| Cline | `.clinerules` | Plain text, no headers |
| GitHub Copilot | `.github/copilot-instructions.md` | Markdown, grouped headers |
| Gemini CLI | `GEMINI.md` | Markdown, grouped headers |
| Aider | `CONVENTIONS.md` | Markdown, grouped headers |

Cursor MDC frontmatter: `description: Project coding rules` + `alwaysApply: true`

Prepend project context block when a project manifest is detected: `# Project: {name} ({type})` / `# Stack: {detected}` / `# Agent: {name}`

If the output file already exists, show a diff and ask for confirmation before overwriting.

---

## Constraints

- Never overwrite an existing rule file without user confirmation
- Never include secrets, tokens, absolute paths, or usernames in any output
- `status` is read-only — no file writes
