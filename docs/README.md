# Claude 官方 Skills 文档

> **注意：** 本仓库包含 Anthropic 的 Claude Skills 实现。关于 Agent Skills 标准，请访问 [agentskills.io](http://agentskills.io)。

# Skills 简介

Skills 是包含指令、脚本和资源的文件夹，Claude 会动态加载它们以提高特定任务的工作效率。Skills 教会 Claude 如何以可重复的方式完成特定任务，无论是使用公司品牌指南创建文档、使用组织特定的工作流程分析数据，还是自动化个人任务。

更多信息请参考：
- [什么是 Skills？](https://support.claude.com/en/articles/12512176-what-are-skills)
- [在 Claude 中使用 Skills](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
- [如何创建自定义 Skills](https://support.claude.com/en/articles/12512198-creating-custom-skills)
- [使用 Agent Skills 为现实世界装备智能体](https://anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

# 关于本仓库

本仓库包含展示 Claude Skills 系统可能性的 Skills。这些 Skills 涵盖创意应用（艺术、音乐、设计）到技术任务（测试 Web 应用、MCP 服务器生成）再到企业工作流程（通信、品牌等）。

每个 Skill 都自包含在各自的文件夹中，包含一个 `SKILL.md` 文件，其中包含 Claude 使用的指令和元数据。浏览这些 Skills 以获取创建自己 Skills 的灵感，或了解不同的模式和方案。

本仓库中的许多 Skills 都是开源的（Apache 2.0）。我们还包含了为 [Claude 的文档功能](https://www.anthropic.com/news/create-files) 提供支持的文档创建和编辑 Skills，存放在 [`skills/docx`](./skills/docx)、[`skills/pdf`](./skills/pdf)、[`skills/pptx`](./skills/pptx) 和 [`skills/xlsx`](./skills/xlsx) 子文件夹中。这些是源码可用（source-available）而非开源，但我们希望与开发者分享这些作为复杂 Skills 的参考，这些 Skills 在生产级 AI 应用中被积极使用。

## 免责声明

**这些 Skills 仅供演示和教育目的提供。** 虽然 Claude 可能会提供其中某些功能，但您从 Claude 收到的实现和行为可能与这些 Skills 中显示的不同。这些 Skills 旨在说明模式和可能性。在依赖它们完成关键任务之前，请务必在您自己的环境中彻底测试 Skills。

# Skills 分类

- [./skills](./skills)：创意与设计、开发与技术、企业与通信以及文档 Skills 的技能示例
- [./spec](./spec)：Agent Skills 规范
- [./template](./template)：Skill 模板

# 在 Claude Code、Claude.ai 和 API 中使用

## Claude Code

您可以通过在 Claude Code 中运行以下命令将此仓库注册为 Claude Code Plugin 市场：

```
/plugin marketplace add anthropics/skills
```

然后，安装特定的 Skills 集合：
1. 选择「浏览并安装插件」(Browse and install plugins)
2. 选择「anthropic-agent-skills」
3. 选择「document-skills」或「example-skills」
4. 选择「立即安装」(Install now)

或者，直接安装任一插件：
```
/plugin install document-skills@anthropic-agent-skills
/plugin install example-skills@anthropic-agent-skills
```

安装插件后，您可以直接提及它来使用该 Skill。例如，如果您安装了来自市场的 `document-skills` 插件，您可以要求 Claude Code：「使用 PDF Skill 从 `path/to/some-file.pdf` 中提取表单字段」

## Claude.ai

这些示例 Skills 都已可在 Claude.ai 的付费计划中使用。

要使用本仓库中的任何 Skill 或上传自定义 Skills，请按照 [在 Claude 中使用 Skills](https://support.claude.com/en/articles/12512180-using-skills-in-claude#h_a4222fa77b) 中的说明进行操作。

## Claude API

您可以通过 Claude API 使用 Anthropic 的预构建 Skills，并上传自定义 Skills。请参阅 [Skills API 快速入门](https://docs.claude.com/en/api/skills-guide#creating-a-skill) 了解更多。

# 创建基础 Skill

创建 Skill 非常简单 - 只需一个包含 `SKILL.md` 文件的文件夹，其中包含 YAML 头部和指令。您可以使用本仓库中的 **template-skill** 作为起点：

```markdown
---
name: my-skill-name
description: 清晰描述这个 Skill 的功能以及何时使用它
---

# My Skill Name

[在此处添加 Claude 在此 Skill 激活时将遵循的指令]

## 示例
- 示例用法 1
- 示例用法 2

## 指南
- 指南 1
- 指南 2
```

头部只需要两个字段：
- `name` - 您的 Skill 的唯一标识符（小写，空格用连字符）
- `description` - 完整描述 Skill 的功能以及何时使用它

下方的 markdown 内容包含 Claude 将遵循的指令、示例和指南。更多详细信息，请参阅[如何创建自定义 Skills](https://support.claude.com/en/articles/12512198-creating-custom-skills)。

# 合作伙伴 Skills

Skills 是教 Claude 更好地使用特定软件的好方法。当我们看到来自合作伙伴的精彩示例 Skills 时，我们可能会在此处重点介绍其中一些：

- **Notion** - [Notion Skills for Claude](https://www.notion.so/notiondevs/Notion-Skills-for-Claude-28da4445d27180c7af1df7d8615723d0)
