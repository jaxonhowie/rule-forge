---
description: Discover all agent rule files on this machine, merge and deduplicate into ~/.claude/merge-rules.md, then generate agent-specific rules on project init. Supports Claude Code, Cursor, Windsurf, Cline, Copilot, Gemini CLI, Aider, and OpenAI Codex.
---

# skill: merge-rules

> Scan all agent rule files on this machine → merge and deduplicate into `~/.claude/merge-rules.md` → generate agent-specific rules on project init.

---

## Commands

| Command | Action |
|---|---|
| `/merge-rules` | Full discovery + merge → write `~/.claude/merge-rules.md` |
| `/merge-rules init` | Apply `merge-rules.md` to current project |
| `/merge-rules init --agent <name>` | Force a specific agent target |
| `/merge-rules status` | Show source count, last merge date, rule count |

---

## Supported Agents

| Agent | Rule File(s) | Format |
|---|---|---|
| Claude Code | `CLAUDE.md` | Markdown with headers |
| OpenAI Codex CLI | `AGENTS.md` | Markdown with headers |
| Cursor | `.cursorrules` · `.cursor/rules/*.mdc` | Plain text / MDC frontmatter |
| Windsurf | `.windsurfrules` | Plain text |
| Cline | `.clinerules` | Plain text |
| GitHub Copilot | `.github/copilot-instructions.md` | Markdown with headers |
| Gemini CLI | `GEMINI.md` | Markdown with headers |
| Aider | `CONVENTIONS.md` | Markdown with headers |

---

## Phase 1 — Discovery

Scan the machine for all known rule files:

```bash
find ~ -maxdepth 10 \( \
  -name "CLAUDE.md"     -o \
  -name "AGENTS.md"     -o \
  -name "GEMINI.md"     -o \
  -name "CONVENTIONS.md" -o \
  -name ".cursorrules"  -o \
  -name ".windsurfrules" -o \
  -name ".clinerules"   -o \
  -name "copilot-instructions.md" \
\) \
  ! -path "*/node_modules/*" \
  ! -path "*/.git/*" \
  ! -path "*/vendor/*" \
  ! -path "*/dist/*" \
  ! -path "*/build/*" \
  ! -path "*/__pycache__/*" \
  2>/dev/null
```

Also scan for Cursor MDC rules:

```bash
find ~ -maxdepth 10 -path "*/.cursor/rules/*.mdc" \
  ! -path "*/node_modules/*" ! -path "*/.git/*" 2>/dev/null
```

Read every discovered file. Record its path, source agent, and full content.

---

## Phase 2 — Merge Pipeline

Process all collected content through this pipeline in order:

**2.1 Extract**
- Collect all rule-bearing lines: bullet points (`-`, `*`), numbered items, bold directives
- For MDC files: strip frontmatter (`---` blocks), extract body rules only
- Strip project-specific values: file paths, repo names, package names, usernames

**2.2 Normalize** *(comparison only, not stored)*
- Lowercase
- Remove punctuation noise

**2.3 Deduplicate**
- Exact match → keep one
- Near-identical (≥85% token overlap) → keep the more concise variant

**2.4 Merge similar**
- Rules covering the same topic → merge into one imperative sentence
- Preserve the union of constraints

**2.5 Conflict resolution**
Priority order: **safer > more general > more specific**

**2.6 Compress**
- Rewrite each rule as a short imperative sentence
- No explanations, no rationale, no examples
- Max 12 words per rule

---

## Phase 3 — Write `~/.claude/merge-rules.md`

```markdown
# merge-rules
<!-- generated: {ISO-DATE} | sources: {N} files | rules: {R} -->

## General
- ...

## Code Quality
- ...

## Safety
- ...

## Workflow
- ...

## Testing
- ...
```

Constraints:
- Group rules by theme (General / Code Quality / Safety / Workflow / Testing / Naming / Git)
- Max 8 rules per group
- Total rules ≤ 60
- Never store absolute paths, tokens, or personal information
- Rules must be agent-agnostic; formatting is applied only at init time

---

## Phase 4 — Project Init (`init` arg)

### 4.1 Detect agent context

If `--agent <name>` is passed, skip detection and use that agent directly.

Otherwise, detect by presence of existing files (first match wins):

| Signal | Agent |
|---|---|
| `.cursor/rules/` dir exists or `.cursorrules` exists | Cursor |
| `.windsurfrules` exists | Windsurf |
| `.clinerules` exists | Cline |
| `.github/copilot-instructions.md` exists | GitHub Copilot |
| `GEMINI.md` exists | Gemini CLI |
| `CONVENTIONS.md` exists and no other signals | Aider |
| `AGENTS.md` exists and no `CLAUDE.md` | OpenAI Codex |
| `CLAUDE.md` exists or no signal found | Claude Code |

### 4.2 Detect project type

Check for: `package.json` · `Cargo.toml` · `pyproject.toml` · `go.mod` · `Gemfile` · `pom.xml` · `build.gradle`

### 4.3 Load merge-rules.md

Load `~/.claude/merge-rules.md`. If it doesn't exist, run Phase 1–3 first.

### 4.4 Render agent-specific output

| Agent | Output file | Format notes |
|---|---|---|
| Claude Code | `CLAUDE.md` | Markdown, grouped headers |
| OpenAI Codex | `AGENTS.md` | Markdown, grouped headers |
| Cursor (legacy) | `.cursorrules` | Plain text rules, no headers, no markdown |
| Cursor (modern) | `.cursor/rules/project.mdc` | MDC frontmatter + markdown body |
| Windsurf | `.windsurfrules` | Plain text rules, no headers |
| Cline | `.clinerules` | Plain text rules, no headers |
| GitHub Copilot | `.github/copilot-instructions.md` | Markdown, grouped headers |
| Gemini CLI | `GEMINI.md` | Markdown, grouped headers |
| Aider | `CONVENTIONS.md` | Markdown, grouped headers |

**Cursor MDC frontmatter template:**

```
---
description: Project coding rules
alwaysApply: true
---
```

**Project context block** (prepend when a manifest file is detected):

```
# Project: {name} ({type})
# Stack: {detected stack}
# Agent: {agent name}
```

### 4.5 Confirmation rule

If the output file already exists → show a diff and ask for confirmation before overwriting.

---

## Constraints

- Never overwrite an existing rule file without user confirmation
- Never include secrets, tokens, absolute paths, or usernames in any output
- Skip `node_modules`, `.git`, `vendor`, `dist`, `build`, `__pycache__` during discovery
- `/merge-rules status` is read-only — no file writes

---

## Output Quality Checklist

Before writing any file, verify:
- [ ] No duplicate rules (exact or semantic)
- [ ] No rule exceeds 12 words
- [ ] No personal/sensitive data
- [ ] Rules are imperative (start with a verb)
- [ ] Groups are coherent and non-overlapping
