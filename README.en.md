# rule-forge

A Claude Code skill that discovers, merges, and manages AI coding agent rules across all your projects.

## Background

Every time a new project starts, you end up hunting through old repos for `CLAUDE.md`, `.cursorrules`, and similar rule files, then manually reformatting them for the new context. It's repetitive and low-value. rule-forge automates that: scan every rule file on the machine, merge and deduplicate, inject into the new project in one command.

> This skill was written using **Claude Code + Claude Sonnet 4.6**.
> Always review the generated output carefully. The correctness and applicability of merged rules are your responsibility — no guarantees are made.

## What it does

1. **Discovers** all agent rule files on your machine
2. **Merges** — deduplicates, resolves conflicts, compresses into short imperative rules
3. **Classifies** — separates BASE (universal behavior) from SPEC (tech-stack specific)
4. **Writes** `~/.claude/merge-rules.md` as your single source of truth
5. **Generates** agent-specific rule files on project init, injecting only the requested scope

## Rule scopes

Merged rules are split into two scopes:

| Scope | Meaning | Examples |
|---|---|---|
| `BASE` | Universal coding behavior, stack-agnostic | Keep functions small, never commit secrets, run tests first |
| `SPEC` | Language / framework / tooling conventions | Use TypeScript strict, use pnpm, use Tailwind |

`init` defaults to `BASE` only, preventing tech-stack rules from polluting unrelated projects.

## Supported agents

| Agent | Rule file(s) |
|---|---|
| Claude Code | `CLAUDE.md` |
| OpenAI Codex CLI | `AGENTS.md` |
| Cursor | `.cursorrules` · `.cursor/rules/*.mdc` |
| Windsurf | `.windsurfrules` |
| Cline | `.clinerules` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Gemini CLI | `GEMINI.md` |
| Aider | `CONVENTIONS.md` |

## Usage

```bash
# Incremental check → merge if changed → write ~/.claude/merge-rules.md
/merge-rules

# Force full re-scan (picks up newly added rule files)
/merge-rules --refresh
/merge-rules -r

# Include rule files found inside test directories (tests/ fixtures/ spec/ etc.)
/merge-rules --test
/merge-rules -t

# Init project (BASE rules only by default)
/merge-rules init

# Choose scope
/merge-rules init --scope base   # Universal rules only (default)
/merge-rules init --scope spec   # Tech-stack rules only
/merge-rules init --scope all    # BASE + SPEC
/merge-rules init -s all         # Short alias for --scope

# Force a specific agent
/merge-rules init --agent cursor

# Check status
/merge-rules status              # Show BASE/SPEC counts, last run time
```

## Install

Claude Code skills require a **directory + `SKILL.md`** structure:

**macOS / Linux**
```bash
mkdir -p ~/.claude/skills/merge-rules
cp SKILL.md ~/.claude/skills/merge-rules/SKILL.md
```

**Windows (PowerShell)**
```powershell
New-Item -ItemType Directory -Force "$env:APPDATA\Claude\skills\merge-rules"
Copy-Item SKILL.md "$env:APPDATA\Claude\skills\merge-rules\SKILL.md"
```

Restart Claude Code — `/merge-rules` will be available immediately.

## Example output

The `examples/` directory contains a complete end-to-end sample:

```
examples/
├── merge-rules.md              # Output of /merge-rules (BASE + SPEC sections)
├── output-claude-code/
│   └── CLAUDE.md               # /merge-rules init (default scope=base)
├── output-cursor/
│   └── .cursorrules            # /merge-rules init --agent cursor (legacy format)
└── output-cursor-modern/
    └── .cursor/rules/project.mdc  # /merge-rules init --agent cursor (MDC format)
```

## Design principles

- `merge-rules.md` is **agent-agnostic** — formatting is applied only at init time
- Classification uses keyword matching only — no embeddings, no external APIs
- Conflict resolution: **safer > more general > more specific**
- Default scope is `base` to prevent tech-stack pollution in unrelated projects
- Never overwrites existing files without confirmation
- Never stores secrets, tokens, or absolute paths
