---
description: 扫描本机所有 agent 规则文件，合并去重生成 ~/.claude/merge-rules.md，并在项目初始化时生成对应 agent 的规则文件。支持 Claude Code、Cursor、Windsurf、Cline、Copilot、Gemini CLI、Aider、OpenAI Codex。
---

# skill: merge-rules

## 命令

| 命令 | 操作 |
|---|---|
| `/merge-rules` | 增量检查 → 有变更时合并 → 写入 `~/.claude/merge-rules.md` |
| `/merge-rules --refresh` / `-r` | 强制全量重新扫描（发现新增文件） |
| `/merge-rules --test` / `-t` | 同上，但**包含**测试目录中的规则文件 |
| `/merge-rules init` | 应用到当前项目（默认 `--scope base`） |
| `/merge-rules init --scope base\|spec\|all` / `-s` | 注入 BASE、SPEC 或全部规则 |
| `/merge-rules init --agent <name>` | 强制指定目标代理 |
| `/merge-rules status` | 显示来源数量、BASE/SPEC 规则数量、上次合并日期（只读） |

## 支持的代理

| 代理 | 规则文件 | 格式 |
|---|---|---|
| Claude Code | `CLAUDE.md` | Markdown |
| OpenAI Codex CLI | `AGENTS.md` | Markdown |
| Cursor | `.cursorrules` · `.cursor/rules/*.mdc` | 纯文本 / MDC frontmatter |
| Windsurf | `.windsurfrules` | 纯文本 |
| Cline | `.clinerules` | 纯文本 |
| GitHub Copilot | `.github/copilot-instructions.md` | Markdown |
| Gemini CLI | `GEMINI.md` | Markdown |
| Aider | `CONVENTIONS.md` | Markdown |

---

## 阶段 0 — 参数 & 操作系统

参数表：

| 参数 | 短别名 | 默认 | 作用 |
|---|---|---|---|
| `--refresh` | `-r` | 关 | 阶段 1：强制全量扫描 |
| `--test` | `-t` | 关 | 阶段 1：包含测试目录 |
| `--scope base\|spec\|all` | `-s` | `base` | 阶段 4：注入范围 |
| `--agent <name>` | — | 自动 | 阶段 4：目标代理 |

运行 `uname -s 2>/dev/null || echo Windows` 判断系统；Windows 下所有命令改用 PowerShell 等价写法。

---

## 阶段 1 — 发现

**Manifest 路径：** `~/.claude/merge-rules.manifest.json`（Windows：`%APPDATA%\Claude\`）

**Manifest 格式：** `{ "generated": "ISO-DATE", "sources": { "/path/to/file": mtime_int } }`

**决策：**
- manifest 不存在 或 `--refresh`/`-r` → **全量扫描**
- 否则 → **缓存检查**（仅对 manifest 中已知路径比对 mtime，无法发现新文件）

**全量扫描目标文件名：**
`CLAUDE.md` `AGENTS.md` `GEMINI.md` `CONVENTIONS.md` `.cursorrules` `.windsurfrules` `.clinerules` `copilot-instructions.md` `.cursor/rules/*.mdc`

**固定排除路径（任何模式均生效）：**
`$PWD`（当前目录）· `node_modules/` · `.git/` · `vendor/` · `dist/` · `build/` · `__pycache__/`

**默认额外排除（`--test`/`-t` 时取消）：**
`tests/` · `test/` · `__tests__/` · `fixtures/` · `spec/` · `specs/`

**缓存检查结果：**
- 无变更 → 打印提示并退出（提示用 `--refresh` 扫描新增文件）
- 有变更 → 打印变更列表，用 manifest 路径继续阶段 2

---

## 阶段 2 — 合并流水线

1. **提取**：收集项目符号、编号项、加粗指令；MDC 文件剥离 frontmatter；剥离路径/包名/用户名等项目特定值
2. **标准化**（仅用于比较）：转小写，移除标点噪音
3. **去重**：完全匹配保留一条；token 重叠 ≥85% 保留更简洁版本
4. **合并相似规则**：同主题规则合并为一条祈使句，保留约束并集
5. **冲突解决**：优先级 更安全 > 更通用 > 更具体
6. **压缩**：改写为简短祈使句，无解释/理由/示例，每条 ≤12 英文单词；纯文本代理（Cursor 旧版 / Windsurf / Cline）输出时剥离 backtick、`**` 等 Markdown 格式符号
7. **分类（BASE vs SPEC）**：规则转小写，命中以下任意关键词 → SPEC，否则 → BASE

| 类别 | 关键词 |
|---|---|
| 语言 | `typescript` `javascript` `python` `rust` `go` `java` `kotlin` `swift` |
| 框架 | `react` `nextjs` `next.js` `vue` `svelte` `fastapi` `nestjs` `express` `django` `spring` |
| 工具链 | `pnpm` `npm` `yarn` `webpack` `vite` `eslint` `prettier` `tailwind` `prisma` `docker` `kubernetes` |
| 架构 | `ddd` `cqrs` `microservice` `monorepo` `mvc` |

---

## 阶段 3 — 写入 merge-rules.md

输出格式：

```markdown
# merge-rules
<!-- generated: {ISO-DATE} | sources: {N} files | base: {B} | spec: {S} -->

## BASE
- ...

## SPEC
- ...
```

约束：BASE ≤40 条，SPEC ≤30 条；不含绝对路径、令牌、个人信息；规则与代理无关。

写入成功后，立即将所有已发现文件的 mtime 写入 manifest。

---

## 阶段 4 — 项目初始化（`init`）

### 4.1 检测代理（`--agent` 优先；否则首次匹配）

| 信号 | 代理 |
|---|---|
| `.cursor/rules/` 或 `.cursorrules` 存在 | Cursor |
| `.windsurfrules` 存在 | Windsurf |
| `.clinerules` 存在 | Cline |
| `.github/copilot-instructions.md` 存在 | GitHub Copilot |
| `GEMINI.md` 存在 | Gemini CLI |
| `CONVENTIONS.md` 存在且无其他信号 | Aider |
| `AGENTS.md` 存在且无 `CLAUDE.md` | OpenAI Codex |
| `CLAUDE.md` 存在或无任何信号 | Claude Code |

### 4.2 检测项目类型

检测：`package.json` · `Cargo.toml` · `pyproject.toml` · `go.mod` · `Gemfile` · `pom.xml` · `build.gradle`

### 4.3 按 scope 过滤并渲染

加载 merge-rules.md（不存在则先运行阶段 0–3），按 scope 过滤后写入目标文件。

| 代理 | 输出文件 | 格式 |
|---|---|---|
| Claude Code | `CLAUDE.md` | Markdown，分组标题 |
| OpenAI Codex | `AGENTS.md` | Markdown，分组标题 |
| Cursor（旧版） | `.cursorrules` | 纯文本，无标题 |
| Cursor（新版） | `.cursor/rules/project.mdc` | MDC frontmatter + Markdown |
| Windsurf | `.windsurfrules` | 纯文本，无标题 |
| Cline | `.clinerules` | 纯文本，无标题 |
| GitHub Copilot | `.github/copilot-instructions.md` | Markdown，分组标题 |
| Gemini CLI | `GEMINI.md` | Markdown，分组标题 |
| Aider | `CONVENTIONS.md` | Markdown，分组标题 |

Cursor MDC frontmatter：`description: Project coding rules` + `alwaysApply: true`

检测到项目清单（package.json / Cargo.toml 等）时前置项目上下文块：`# Project: {name} ({type})` / `# Stack: {detected}` / `# Agent: {name}`

输出文件已存在时，显示差异并请求确认后再覆写。

---

## 约束

- 未经用户确认不覆写已有规则文件
- 输出中不含密钥、令牌、绝对路径、用户名
- `status` 为只读操作
