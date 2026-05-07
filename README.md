# rule-forge

一个 Claude Code skill，用于发现、合并和管理你所有项目中的 AI 编码代理规则。

## 背景

每次开启新项目时，都要花时间四处收集散落在各个仓库里的 `CLAUDE.md`、`.cursorrules` 等规则文件，再手动整理成当前项目需要的格式——这件事重复且低价值。rule-forge 的目的就是把这个过程自动化：扫描本机所有规则、合并去重、一键注入到新项目里。

> 本 skill 使用 **Claude Code + Claude Sonnet 4.6** 编写。
> 使用前请仔细阅读生成结果，规则的正确性与适用性需自行判断，作者不作任何保证。

## 功能介绍

1. **发现** 你机器上所有 agent 的规则文件
2. **合并** — 去重、解决冲突、压缩为简短的祈使句规则
3. **分类** — 自动区分 BASE（通用行为）和 SPEC（技术栈专属）
4. **写入** `~/.claude/merge-rules.md` 作为唯一的规则来源
5. **生成** 代理专属的规则文件，在项目初始化时按需注入

## 规则范围（Scope）

合并后的规则分为两类：

| Scope | 含义 | 示例 |
|---|---|---|
| `BASE` | 与技术栈无关的通用编码行为 | 保持函数简短、禁止提交密钥、先写测试 |
| `SPEC` | 语言/框架/工具链专属约定 | 使用 TypeScript strict、用 pnpm、用 Tailwind |

`init` 默认只注入 `BASE`，避免把技术栈规则污染到不相关的项目。

## 支持的代理

| 代理 | 规则文件 |
|---|---|
| Claude Code | `CLAUDE.md` |
| OpenAI Codex CLI | `AGENTS.md` |
| Cursor | `.cursorrules` · `.cursor/rules/*.mdc` |
| Windsurf | `.windsurfrules` |
| Cline | `.clinerules` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Gemini CLI | `GEMINI.md` |
| Aider | `CONVENTIONS.md` |

## 使用方法

```bash
# 扫描全机 → 合并分类 → 生成 ~/.claude/merge-rules.md（自动排除测试目录）
/merge-rules

# 包含测试目录中的规则文件（tests/ fixtures/ spec/ 等）
/merge-rules --test
/merge-rules -t

# 初始化项目（默认只注入 BASE 规则）
/merge-rules init

# 按 scope 选择注入范围
/merge-rules init --scope base   # 仅通用规则（默认）
/merge-rules init --scope spec   # 仅技术栈规则
/merge-rules init --scope all    # 全部规则
/merge-rules init -s all         # --scope 的短别名

# 强制指定目标代理
/merge-rules init --agent cursor

# 查看状态
/merge-rules status              # 显示 BASE/SPEC 数量、上次运行时间
```

## 安装

Claude Code skill 需要放在 **目录 + `SKILL.md`** 的结构下：

**macOS / Linux**
```bash
mkdir -p ~/.claude/skills/merge-rules
cp skill.md ~/.claude/skills/merge-rules/SKILL.md
```

**Windows（PowerShell）**
```powershell
New-Item -ItemType Directory -Force "$env:APPDATA\Claude\skills\merge-rules"
Copy-Item skill.md "$env:APPDATA\Claude\skills\merge-rules\SKILL.md"
```

重启 Claude Code 后，`/merge-rules` 即可使用。

## 示例输出

`examples/` 目录包含一个完整的运行示例：

```
examples/
├── merge-rules.md              # /merge-rules 生成的合并规则（BASE + SPEC 两节）
├── output-claude-code/
│   └── CLAUDE.md               # /merge-rules init（默认 scope=base）
├── output-cursor/
│   └── .cursorrules            # /merge-rules init --agent cursor（旧版格式）
└── output-cursor-modern/
    └── .cursor/rules/project.mdc  # /merge-rules init --agent cursor（MDC 格式）
```

## 设计原则

- `merge-rules.md` 与代理无关 — 格式仅在初始化时应用
- 分类基于关键词匹配，无需外部 API 或嵌入模型
- 冲突解决优先级：**更安全 > 更通用 > 更具体**
- 默认 scope 为 `base`，防止技术栈规则污染无关项目
- 绝不在未获得确认的情况下覆写已有文件
- 绝不存储密钥、令牌或绝对路径
