# rule-forge

A Claude Code skill that discovers, merges, and manages AI coding agent rules across all your projects.

## What it does

1. **Discovers** all agent rule files on your machine
2. **Merges** them — deduplicates, resolves conflicts, compresses into short imperative rules
3. **Writes** `~/.claude/merge-rules.md` as your single source of truth
4. **Generates** agent-specific rule files on project init

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
/merge-rules                      # Scan machine → build ~/.claude/merge-rules.md
/merge-rules init                 # Auto-detect agent → generate rule file
/merge-rules init --agent cursor  # Force a specific agent
/merge-rules status               # Show source count, last run, rule count
```

## Install

Claude Code skills require a **directory + `SKILL.md`** structure:

**macOS / Linux**
```bash
mkdir -p ~/.claude/skills/merge-rules
cp skill.md ~/.claude/skills/merge-rules/SKILL.md
```

**Windows (PowerShell)**
```powershell
New-Item -ItemType Directory -Force "$env:APPDATA\Claude\skills\merge-rules"
Copy-Item skill.md "$env:APPDATA\Claude\skills\merge-rules\SKILL.md"
```

Restart Claude Code — `/merge-rules` will be available immediately.

## Example output

The `examples/` directory contains a complete end-to-end sample:

```
examples/
├── merge-rules.md              # Output of /merge-rules (merged from 11 source files)
├── output-claude-code/
│   └── CLAUDE.md               # /merge-rules init → Claude Code
├── output-cursor/
│   └── .cursorrules            # /merge-rules init --agent cursor (legacy format)
└── output-cursor-modern/
    └── .cursor/rules/project.mdc  # /merge-rules init --agent cursor (MDC format)
```

## Design principles

- `merge-rules.md` is **agent-agnostic** — formatting is applied only at init time
- Conflict resolution: **safer > more general > more specific**
- Max 12 words per rule · max 60 rules total
- Never overwrites existing files without confirmation
- Never stores secrets, tokens, or absolute paths
