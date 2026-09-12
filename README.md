# yifanfengshun-s_skills

个人 Agent Skills 技能仓库，包含日常学习与工作中开发的可复用 AI 智能体技能。

## 技能列表

| 技能名称 | 描述 |
|---------|------|
| [pdf-to-md](./pdf-to-md/) | 将 PDF 课件或教材转换为同名的 Markdown 文件，支持文本、公式和图表提取 |

## 安装方式

```bash
# Claude Code
/plugin marketplace add yifanfengshun517/yifanfengshun-s_skills

# 或直接复制技能文件夹到 .claude/skills/
cp -r pdf-to-md ~/.claude/skills/pdf-to-md
```

## 规范

本仓库遵循 [Agent Skills Open Standard](https://agentskills.io/specification)，每个技能目录包含：

- `SKILL.md` — 技能元数据与指令（YAML frontmatter + Markdown 正文）
