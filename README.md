# rule-forge

一个 Claude Code skill，用于发现、合并和管理你所有项目中的 AI 编码代理规则。

## 功能介绍

1. **发现** 你机器上所有的代理规则文件
2. **合并** 它们 — 去重、解决冲突、压缩为简短的祈使句规则
3. **写入** `~/.claude/merge-rules.md` 作为唯一的规则来源
4. **生成** 代理专属的规则文件，在项目初始化时应用

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
/merge-rules                      # 扫描机器 → 构建 ~/.claude/merge-rules.md
/merge-rules init                 # 自动检测代理 → 生成规则文件
/merge-rules init --agent cursor  # 强制指定代理
/merge-rules status               # 显示来源数量、上次运行时间、规则数量
```

## 安装

Claude Code skill 需要放在 **目录 + `SKILL.md`** 的结构下：

```bash
mkdir -p ~/.claude/skills/merge-rules
cp skill.md ~/.claude/skills/merge-rules/SKILL.md
```

重启 Claude Code 后，`/merge-rules` 即可使用。

## 示例输出

`examples/` 目录包含一个完整的运行示例：

```
examples/
├── merge-rules.md              # /merge-rules 生成的合并规则（来自 11 个源文件）
├── output-claude-code/
│   └── CLAUDE.md               # /merge-rules init → Claude Code
├── output-cursor/
│   └── .cursorrules            # /merge-rules init --agent cursor（旧版格式）
└── output-cursor-modern/
    └── .cursor/rules/project.mdc  # /merge-rules init --agent cursor（MDC 格式）
```

## 设计原则

- `merge-rules.md` 与代理无关 — 格式仅在初始化时应用
- 冲突解决优先级：**更安全 > 更通用 > 更具体**
- 每条规则最多 12 个英文单词 · 规则总数不超过 60 条
- 绝不在未获得确认的情况下覆写已有文件
- 绝不存储密钥、令牌或绝对路径
