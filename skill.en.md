---
description: Discover all agent rule files on this machine, merge and deduplicate into ~/.claude/merge-rules.md, then generate agent-specific rules on project init. Supports Claude Code, Cursor, Windsurf, Cline, Copilot, Gemini CLI, Aider, and OpenAI Codex.
---

# skill: merge-rules

> Scan all agent rule files on this machine → merge and deduplicate into `~/.claude/merge-rules.md` → generate agent-specific rules on project init.

---

## Commands

| Command | Action |
|---|---|
| `/merge-rules` | Incremental check → merge if changed → write `~/.claude/merge-rules.md` |
| `/merge-rules --refresh` | Force full re-scan (picks up newly added files) |
| `/merge-rules -r` | Short alias for `--refresh` |
| `/merge-rules --test` | Same, but **include** rule files inside test directories |
| `/merge-rules -t` | Short alias for `--test` |
| `/merge-rules init` | Apply to current project (default `--scope base`) |
| `/merge-rules init --scope base` | Inject BASE rules only (universal behavior, default) |
| `/merge-rules init --scope spec` | Inject SPEC rules only (tech-stack specific) |
| `/merge-rules init --scope all` | Inject BASE + SPEC rules |
| `/merge-rules init -s <scope>` | Short alias for `--scope` |
| `/merge-rules init --agent <name>` | Force a specific agent target |
| `/merge-rules status` | Show source count, BASE/SPEC rule counts, last merge date |

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

## Phase 0 — Parse arguments & detect OS

### Argument parsing

Extract all flags before any operations:

| Flag | Alias | Default | Used in |
|---|---|---|---|
| `--refresh` | `-r` | off | Phase 1: force full scan, ignore manifest |
| `--test` | `-t` | off | Phase 1: include test directories in discovery |
| `--scope base\|spec\|all` | `-s` | `base` | Phase 4: rule scope filter (`init` only) |
| `--agent <name>` | — | auto-detect | Phase 4: target agent (`init` only) |

### Detect OS

```bash
uname -s 2>/dev/null || echo "Windows"
```

- Output contains `Linux` or `Darwin` → **Unix mode**
- Output is `Windows` or command not found → **Windows mode**

---

## Phase 1 — Discovery (incremental mode)

**Always excluded (all modes):**
- Current working directory `$PWD` (prevents circular pollution)
- `node_modules/` · `.git/` · `vendor/` · `dist/` · `build/` · `__pycache__/`
- Test dirs (default only): `tests/` · `test/` · `__tests__/` · `fixtures/` · `spec/` · `specs/`

---

### Manifest path

| OS | Path |
|---|---|
| macOS / Linux | `~/.claude/merge-rules.manifest.json` |
| Windows | `%APPDATA%\Claude\merge-rules.manifest.json` |

Manifest format:

```json
{
  "generated": "ISO-DATE",
  "sources": {
    "/absolute/path/to/CLAUDE.md": 1746600000,
    "/absolute/path/to/.cursorrules": 1746500000
  }
}
```

Values in `sources` are **Unix mtime timestamps** (integer seconds).

---

### 1.1 Decision: cached mode vs full scan

```
if manifest missing OR --refresh / -r set:
    → full scan mode (1.2)
else:
    → cached check mode (1.3)
```

---

### 1.2 Full scan mode

Run `find ~` (on `--refresh` or first run):

**Unix / macOS:**

```bash
CURRENT_DIR="$(pwd)"

find ~ -maxdepth 10 \( \
  -name "CLAUDE.md"      -o \
  -name "AGENTS.md"      -o \
  -name "GEMINI.md"      -o \
  -name "CONVENTIONS.md" -o \
  -name ".cursorrules"   -o \
  -name ".windsurfrules" -o \
  -name ".clinerules"    -o \
  -name "copilot-instructions.md" \
\) \
  ! -path "$CURRENT_DIR/*" \
  ! -path "*/node_modules/*" \
  ! -path "*/.git/*" \
  ! -path "*/vendor/*" \
  ! -path "*/dist/*" \
  ! -path "*/build/*" \
  ! -path "*/__pycache__/*" \
  ! -path "*/tests/*" \
  ! -path "*/test/*" \
  ! -path "*/__tests__/*" \
  ! -path "*/fixtures/*" \
  ! -path "*/spec/*" \
  ! -path "*/specs/*" \
  2>/dev/null

# Cursor MDC
find ~ -maxdepth 10 -path "*/.cursor/rules/*.mdc" \
  ! -path "$CURRENT_DIR/*" \
  ! -path "*/node_modules/*" ! -path "*/.git/*" \
  ! -path "*/tests/*" ! -path "*/test/*" ! -path "*/fixtures/*" \
  2>/dev/null
```

With `--test / -t`: remove test dir exclusions; `$CURRENT_DIR` always stays.

**Windows (PowerShell):**

```powershell
$currentDir  = (Get-Location).Path
$exclude     = 'node_modules|\.git|vendor|dist|build|__pycache__'
$excludeTest = 'tests[/\\]|test[/\\]|__tests__[/\\]|fixtures[/\\]|specs?[/\\]'
$names = @("CLAUDE.md","AGENTS.md","GEMINI.md","CONVENTIONS.md",
           ".cursorrules",".windsurfrules",".clinerules",
           "copilot-instructions.md")

Get-ChildItem -Path $HOME -Recurse -Depth 10 -Include $names `
  -ErrorAction SilentlyContinue |
  Where-Object { $_.FullName -notlike "$currentDir*" -and
                 $_.FullName -notmatch $exclude -and
                 $_.FullName -notmatch $excludeTest }
```

With `--test / -t`: remove `$excludeTest` condition; others stay.

---

### 1.3 Cached check mode

Skip `find ~`. Check mtime of paths already in manifest:

**Unix / macOS:**

```bash
python3 -c "
import json, os, sys
m = json.load(open(sys.argv[1]))
changed = []
for path, stored_mtime in m['sources'].items():
    if not os.path.exists(path):
        changed.append(('removed', path))
    elif int(os.path.getmtime(path)) != stored_mtime:
        changed.append(('modified', path))
print(json.dumps(changed))
" "$HOME/.claude/merge-rules.manifest.json"
```

**Windows (PowerShell):**

```powershell
$manifest = Get-Content $manifestPath | ConvertFrom-Json
$changed  = @()
foreach ($entry in $manifest.sources.PSObject.Properties) {
    $path  = $entry.Name
    $mtime = $entry.Value
    $item  = Get-Item $path -ErrorAction SilentlyContinue
    if (-not $item) { $changed += @{status="removed"; path=$path} }
    elseif ([int]$item.LastWriteTimeUtc.Subtract([datetime]'1970-01-01').TotalSeconds -ne $mtime) {
        $changed += @{status="modified"; path=$path}
    }
}
```

**Decision:**

```
if changed is empty:
    → print "No rule files changed. Skipping merge. (use --refresh to scan for new files)"
    → EXIT (do not enter Phase 2)
else:
    → print list of changed/removed files
    → use manifest paths as discovered_files, continue to Phase 2
    → NOTE: new rule files added since last --refresh will NOT be found in this mode
```

---

Read every file in `discovered_files`. Record its path, source agent, and full content.

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

**2.7 Classify (BASE vs SPEC)**

Lowercase each rule and check for the following keywords. Any match → **SPEC**; otherwise → **BASE**.

| Category | Keywords |
|---|---|
| Languages | `typescript` `javascript` `python` `rust` `go` `java` `kotlin` `swift` |
| Frameworks | `react` `nextjs` `next.js` `vue` `svelte` `fastapi` `nestjs` `express` `django` `spring` |
| Tooling | `pnpm` `npm` `yarn` `webpack` `vite` `eslint` `prettier` `tailwind` `prisma` `docker` `kubernetes` |
| Architecture | `ddd` `cqrs` `microservice` `monorepo` `mvc` |

**BASE examples** (no keywords matched):
- Write clean, readable code
- Keep functions focused and small
- Run tests before committing
- Never expose secrets or credentials

**SPEC examples** (keyword matched):
- Use TypeScript strict mode
- Prefer React hooks over class components
- Use pnpm as the package manager
- Apply Tailwind utility classes for styling

---

## Phase 3 — Write merge-rules.md

Storage path varies by OS:

| OS | Path |
|---|---|
| macOS / Linux | `~/.claude/merge-rules.md` |
| Windows | `%APPDATA%\Claude\merge-rules.md` |

```markdown
# merge-rules
<!-- generated: {ISO-DATE} | sources: {N} files | base: {B} | spec: {S} -->

## BASE
- (universal agent behavior rules)
- ...

## SPEC
- (tech-stack / project-specific rules)
- ...
```

Constraints:
- Exactly two top-level sections: `BASE` and `SPEC`
- BASE rules ≤ 40, SPEC rules ≤ 30
- Never store absolute paths, tokens, or personal information
- Rules must be agent-agnostic; formatting is applied only at init time

### 3.1 Write manifest

Immediately after writing `merge-rules.md`, write the manifest:

**Unix / macOS:**

```bash
python3 -c "
import json, os, sys
paths = sys.argv[1:]
sources = {p: int(os.path.getmtime(p)) for p in paths if os.path.exists(p)}
manifest = {'generated': __import__('datetime').datetime.utcnow().isoformat()+'Z',
            'sources': sources}
print(json.dumps(manifest, indent=2))
" $DISCOVERED_FILES > "$HOME/.claude/merge-rules.manifest.json"
```

**Windows (PowerShell):**

```powershell
$sources = @{}
foreach ($path in $discoveredFiles) {
    $item = Get-Item $path -ErrorAction SilentlyContinue
    if ($item) {
        $sources[$path] = [int]$item.LastWriteTimeUtc.Subtract([datetime]'1970-01-01').TotalSeconds
    }
}
@{ generated = (Get-Date).ToUniversalTime().ToString("o"); sources = $sources } |
    ConvertTo-Json -Depth 3 | Set-Content $manifestPath
```

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

### 4.3 Load and filter by scope

Load merge-rules.md from the OS-appropriate path (macOS/Linux: `~/.claude/merge-rules.md`; Windows: `%APPDATA%\Claude\merge-rules.md`). If it doesn't exist, run Phase 0–3 first.

Filter rules according to `--scope`:

| scope | Rules to inject |
|---|---|
| `base` (default) | `## BASE` only |
| `spec` | `## SPEC` only |
| `all` | `## BASE` + `## SPEC` |

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
- [ ] Each rule is in the correct scope (BASE has no tech keywords; SPEC has one)
- [ ] init output contains only the requested scope's rules
