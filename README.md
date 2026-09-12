# yifanfengshun-s_skills

个人 Agent Skills 技能仓库，包含日常学习与工作中开发的可复用 AI 智能体技能。

遵循 [Agent Skills Open Standard](https://agentskills.io/specification)。

## 技能列表

| 技能名称 | 描述 |
|---------|------|
| [pdf-to-md](./pdf-to-md/) | 将 PDF 课件或教材转换为同名的 Markdown 文件，支持文本、公式和图表提取 |

## 安装方式

```bash
# Claude Code marketplace 安装
/plugin marketplace add yifanfengshun517/yifanfengshun-s_skills

# 或直接复制技能文件夹到本地
# Windows (Agnes Code)
copy /s pdf-to-md C:\Users\%USERNAME%\.agnes\skills\pdf-to-md

# macOS / Linux (Claude Code)
cp -r pdf-to-md ~/.claude/skills/pdf-to-md
```

## 仓库结构

```
yifanfengshun-s_skills/
├── README.md           # 仓库说明
├── .gitignore          # Git 忽略配置
└── pdf-to-md/
    └── SKILL.md        # 技能定义（YAML frontmatter + Markdown 正文）
```

## 规范说明

每个技能目录遵循 Agent Skills 标准：

- `SKILL.md` — 技能元数据与指令（YAML frontmatter + Markdown 正文）
  - `name`: 技能唯一标识符（与目录名一致）
  - `description`: 触发条件描述，用于自动发现
  - `metadata.argument-hint`: 参数提示

## 贡献

欢迎提交 Issue 和 Pull Request。