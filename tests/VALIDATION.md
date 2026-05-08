# Validation Guide

## Setup

```bash
# 1. Install the skill
mkdir -p ~/.claude/skills/merge-rules
cp SKILL.md ~/.claude/skills/merge-rules/SKILL.md

# 2. Point discovery at the test fixtures (run from repo root)
# During testing, temporarily set HOME or run from the fixtures dir
```

---

## Test 1 — Discovery

**Goal**: Confirm all rule files are found and none are missed.

**Input**: `tests/fixtures/` contains:
- `project-a/CLAUDE.md`
- `project-b/.cursorrules`
- `project-c/AGENTS.md`

**Run**: `/merge-rules`

**Pass criteria**:
- [ ] Output header reports `sources: 3 files`
- [ ] No fixture file is silently skipped

---

## Test 2 — Deduplication

**Goal**: Near-identical rules across files are collapsed to one.

**Input**: All three fixture files contain a variation of "write unit tests" and "never commit secrets".

**Pass criteria**:
- [ ] Exactly **one** rule about testing in the output
- [ ] Exactly **one** rule about secrets in the output
- [ ] The kept rule is the most concise variant

---

## Test 3 — Merge similar rules

**Goal**: Thematically related rules are grouped and compressed.

**Input**: "Use TypeScript strict mode" appears in all three files with different wording.

**Pass criteria**:
- [ ] Only one TypeScript-strict rule survives
- [ ] Rule is ≤ 12 words
- [ ] Rule starts with a verb (imperative)

---

## Test 4 — Conflict resolution

**Goal**: Conflicting rules are resolved by safer/more general wins.

**Input**: `project-c/AGENTS.md` contains "Use tabs for indentation" — a specific, potentially conflicting style rule not present in other files.

**Pass criteria**:
- [ ] If no other indentation rule exists → the rule is included as-is
- [ ] If a conflicting rule exists → the more general/safer rule is kept
- [ ] No two rules in the output directly contradict each other

---

## Test 5 — Output format per agent

**Goal**: Same `merge-rules.md` renders correctly for each agent format.

**Run**: `/merge-rules init --agent <name>` for each agent in a fresh temp directory.

| Agent | Expected file | Pass criteria |
|---|---|---|
| `claude` | `CLAUDE.md` | Has `##` headers, starts with project context block |
| `cursor` | `.cursorrules` | Plain text only, no `#` headers, no markdown |
| `cursor-modern` | `.cursor/rules/project.mdc` | Has `---` frontmatter with `alwaysApply: true` |
| `windsurf` | `.windsurfrules` | Plain text only, no headers |
| `cline` | `.clinerules` | Plain text only, no headers |
| `copilot` | `.github/copilot-instructions.md` | Has `##` headers |
| `gemini` | `GEMINI.md` | Has `##` headers |
| `codex` | `AGENTS.md` | Has `##` headers |
| `aider` | `CONVENTIONS.md` | Has `##` headers |

---

## Test 6 — Overwrite protection

**Goal**: Existing files are never silently overwritten.

**Setup**: Create a `CLAUDE.md` in the current directory with arbitrary content.

**Run**: `/merge-rules init`

**Pass criteria**:
- [ ] Claude shows a diff before writing
- [ ] Claude asks for confirmation
- [ ] File is unchanged if user says no

---

## Test 7 — Status is read-only

**Run**: `/merge-rules status`

**Pass criteria**:
- [ ] Reports source file count, last generated date, rule count
- [ ] No files are created or modified

---

## Quick sanity check (compare with expected output)

After running `/merge-rules` against the test fixtures, compare the result against
`tests/expected-merge-rules.md`:

- All rules in `expected-merge-rules.md` should appear (or a semantically equivalent rule)
- No rule in the output should duplicate another
- Every rule should start with a verb
- No rule should exceed 12 words
