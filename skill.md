---
description: 扫描本机所有 agent 规则文件，合并去重生成 ~/.claude/merge-rules.md，并在项目初始化时生成对应 agent 的规则文件。支持 Claude Code、Cursor、Windsurf、Cline、Copilot、Gemini CLI、Aider、OpenAI Codex。
---

# skill: merge-rules

> 扫描本机上所有代理规则文件 → 合并去重到 `~/.claude/merge-rules.md` → 在项目初始化时生成代理专属规则文件。

---

## 命令

| 命令 | 操作 |
|---|---|
| `/merge-rules` | 发现 + 合并 + 分类 → 写入 `~/.claude/merge-rules.md`（排除测试目录） |
| `/merge-rules --test` | 同上，但**包含**测试目录中的规则文件 |
| `/merge-rules -t` | `--test` 的短别名 |
| `/merge-rules init` | 应用到当前项目（默认 `--scope base`） |
| `/merge-rules init --scope base` | 仅注入 BASE 规则（通用行为，默认值） |
| `/merge-rules init --scope spec` | 仅注入 SPEC 规则（技术栈专属） |
| `/merge-rules init --scope all` | 注入 BASE + SPEC 全部规则 |
| `/merge-rules init -s <scope>` | `--scope` 的短别名 |
| `/merge-rules init --agent <name>` | 强制指定目标代理 |
| `/merge-rules status` | 显示来源数量、BASE/SPEC 规则数量、上次合并日期 |

---

## 支持的代理

| 代理 | 规则文件 | 格式 |
|---|---|---|
| Claude Code | `CLAUDE.md` | Markdown（带标题） |
| OpenAI Codex CLI | `AGENTS.md` | Markdown（带标题） |
| Cursor | `.cursorrules` · `.cursor/rules/*.mdc` | 纯文本 / MDC frontmatter |
| Windsurf | `.windsurfrules` | 纯文本 |
| Cline | `.clinerules` | 纯文本 |
| GitHub Copilot | `.github/copilot-instructions.md` | Markdown（带标题） |
| Gemini CLI | `GEMINI.md` | Markdown（带标题） |
| Aider | `CONVENTIONS.md` | Markdown（带标题） |

---

## 阶段 0 — 解析参数 & 检测操作系统

### 参数解析

在执行任何操作前，从命令中提取所有参数：

| 参数 | 短别名 | 默认值 | 作用阶段 |
|---|---|---|---|
| `--test` | `-t` | 关闭 | 阶段 1：是否扫描测试目录 |
| `--scope base\|spec\|all` | `-s` | `base` | 阶段 4：注入规则范围（仅 `init`） |
| `--agent <name>` | — | 自动检测 | 阶段 4：目标代理（仅 `init`） |

### 检测操作系统

执行前先判断运行环境：

```bash
uname -s 2>/dev/null || echo "Windows"
```

- 输出包含 `Linux` 或 `Darwin` → **Unix 模式**
- 输出 `Windows` 或命令不存在 → **Windows 模式**

---

## 阶段 1 — 发现

默认排除以下测试相关目录（`--test` / `-t` 未指定时）：

```
tests/  test/  __tests__/  fixtures/  spec/  specs/
```

### Unix / macOS（bash / zsh）

**默认（排除测试目录）：**

```bash
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
  ! -path "*/node_modules/*" ! -path "*/.git/*" \
  ! -path "*/tests/*" ! -path "*/test/*" ! -path "*/fixtures/*" \
  2>/dev/null
```

**指定 `--test` / `-t` 时，去掉所有 `! -path "*/test*/*"` 和 `! -path "*/fixtures/*"` 排除项，其余不变。**

### Windows（PowerShell 5.1+）

**默认（排除测试目录）：**

```powershell
$names = @("CLAUDE.md","AGENTS.md","GEMINI.md","CONVENTIONS.md",
           ".cursorrules",".windsurfrules",".clinerules",
           "copilot-instructions.md")
$exclude = 'node_modules|\.git|vendor|dist|build|__pycache__'
$excludeTest = 'tests[/\\]|test[/\\]|__tests__[/\\]|fixtures[/\\]|specs?[/\\]'

Get-ChildItem -Path $HOME -Recurse -Depth 10 -Include $names `
  -ErrorAction SilentlyContinue |
  Where-Object { $_.FullName -notmatch $exclude -and
                 $_.FullName -notmatch $excludeTest }

# Cursor MDC
Get-ChildItem -Path $HOME -Recurse -Depth 10 -Filter "*.mdc" `
  -ErrorAction SilentlyContinue |
  Where-Object { $_.FullName -match '\.cursor[/\\]rules' -and
                 $_.FullName -notmatch $exclude -and
                 $_.FullName -notmatch $excludeTest }
```

**指定 `--test` / `-t` 时，移除 `-and $_.FullName -notmatch $excludeTest` 条件，其余不变。**

读取每个发现的文件，记录其路径、来源代理和完整内容。

---

## 阶段 2 — 合并流水线

按以下顺序对所有收集到的内容进行处理：

**2.1 提取**
- 收集所有包含规则的行：项目符号（`-`、`*`）、编号项、加粗指令
- 对于 MDC 文件：剥离 frontmatter（`---` 块），仅提取正文规则
- 剥离项目特定值：文件路径、仓库名、包名、用户名

**2.2 标准化**（仅用于比较，不存储）
- 转为小写
- 移除标点噪音

**2.3 去重**
- 完全匹配 → 保留一条
- 近似相同（token 重叠度 ≥85%） → 保留更简洁的版本

**2.4 合并相似规则**
- 覆盖相同主题的规则 → 合并为一条祈使句
- 保留约束条件的并集

**2.5 冲突解决**
优先级顺序：**更安全 > 更通用 > 更具体**

**2.6 压缩**
- 将每条规则改写为简短的祈使句
- 无解释、无理由、无示例
- 每条规则最多 12 个英文单词

**2.7 分类（BASE vs SPEC）**

将规则转为小写，逐条检查是否包含以下关键词。命中任意一个 → **SPEC**；否则 → **BASE**。

| 类别 | 关键词 |
|---|---|
| 语言 | `typescript` `javascript` `python` `rust` `go` `java` `kotlin` `swift` |
| 框架 | `react` `nextjs` `next.js` `vue` `svelte` `fastapi` `nestjs` `express` `django` `spring` |
| 工具链 | `pnpm` `npm` `yarn` `webpack` `vite` `eslint` `prettier` `tailwind` `prisma` `docker` `kubernetes` |
| 架构 | `ddd` `cqrs` `microservice` `monorepo` `mvc` |

**BASE 示例**（不含上述关键词）：
- Write clean, readable code
- Keep functions focused and small
- Run tests before committing
- Never expose secrets or credentials

**SPEC 示例**（含关键词）：
- Use TypeScript strict mode
- Prefer React hooks over class components
- Use pnpm as the package manager
- Apply Tailwind utility classes for styling

---

## 阶段 3 — 写入 merge-rules.md

存储路径因操作系统而异：

| 系统 | 路径 |
|---|---|
| macOS / Linux | `~/.claude/merge-rules.md` |
| Windows | `%APPDATA%\Claude\merge-rules.md` |

```markdown
# merge-rules
<!-- generated: {ISO-DATE} | sources: {N} files | base: {B} | spec: {S} -->

## BASE
- (通用行为规则)
- ...

## SPEC
- (技术栈/项目专属规则)
- ...
```

约束条件：
- 只有两个顶级分组：`BASE` 和 `SPEC`
- BASE 规则总数 ≤ 40，SPEC 规则总数 ≤ 30
- 绝不存储绝对路径、令牌或个人信息
- 规则必须与代理无关；格式仅在初始化时应用

---

## 阶段 4 — 项目初始化（`init` 参数）

### 4.1 检测代理上下文

如果传入了 `--agent <name>`，则跳过检测，直接使用该代理。

否则，通过检测已存在的文件来判断（首次匹配生效）：

| 信号 | 代理 |
|---|---|
| `.cursor/rules/` 目录存在或 `.cursorrules` 存在 | Cursor |
| `.windsurfrules` 存在 | Windsurf |
| `.clinerules` 存在 | Cline |
| `.github/copilot-instructions.md` 存在 | GitHub Copilot |
| `GEMINI.md` 存在 | Gemini CLI |
| `CONVENTIONS.md` 存在且无其他信号 | Aider |
| `AGENTS.md` 存在且无 `CLAUDE.md` | OpenAI Codex |
| `CLAUDE.md` 存在或未找到任何信号 | Claude Code |

### 4.2 检测项目类型

检测以下文件：`package.json` · `Cargo.toml` · `pyproject.toml` · `go.mod` · `Gemfile` · `pom.xml` · `build.gradle`

### 4.3 加载并按 scope 过滤规则

按操作系统加载对应路径的 merge-rules.md（macOS/Linux：`~/.claude/merge-rules.md`；Windows：`%APPDATA%\Claude\merge-rules.md`）。如果文件不存在，先运行阶段 0–3。

根据 `--scope` 参数过滤要注入的规则：

| scope | 注入内容 |
|---|---|
| `base`（默认） | 仅 `## BASE` 中的规则 |
| `spec` | 仅 `## SPEC` 中的规则 |
| `all` | `## BASE` + `## SPEC` 全部规则 |

### 4.4 渲染代理专属输出

| 代理 | 输出文件 | 格式说明 |
|---|---|---|
| Claude Code | `CLAUDE.md` | Markdown，分组标题 |
| OpenAI Codex | `AGENTS.md` | Markdown，分组标题 |
| Cursor（旧版） | `.cursorrules` | 纯文本规则，无标题，无 Markdown |
| Cursor（新版） | `.cursor/rules/project.mdc` | MDC frontmatter + Markdown 正文 |
| Windsurf | `.windsurfrules` | 纯文本规则，无标题 |
| Cline | `.clinerules` | 纯文本规则，无标题 |
| GitHub Copilot | `.github/copilot-instructions.md` | Markdown，分组标题 |
| Gemini CLI | `GEMINI.md` | Markdown，分组标题 |
| Aider | `CONVENTIONS.md` | Markdown，分组标题 |

**Cursor MDC frontmatter 模板：**

```
---
description: Project coding rules
alwaysApply: true
---
```

**项目上下文块**（检测到清单文件时前置）：

```
# Project: {name} ({type})
# Stack: {detected stack}
# Agent: {agent name}
```

### 4.5 确认规则

如果输出文件已存在 → 显示差异，并在覆写前请求确认。

---

## 约束条件

- 绝不在未获得用户确认的情况下覆写已有规则文件
- 绝不在任何输出中包含密钥、令牌、绝对路径或用户名
- 发现阶段跳过 `node_modules`、`.git`、`vendor`、`dist`、`build`、`__pycache__`
- `/merge-rules status` 为只读操作 — 不写入任何文件

---

## 输出质量检查清单

在写入任何文件之前，验证以下项目：
- [ ] 无重复规则（完全匹配或语义匹配）
- [ ] 每条规则不超过 12 个英文单词
- [ ] 不包含个人/敏感数据
- [ ] 规则为祈使句（以动词开头）
- [ ] 每条规则归入正确的 scope（BASE 不含技术栈关键词，SPEC 含关键词）
- [ ] init 输出只包含所请求 scope 的规则
