# Claude API 开发指南

## 概述

本指南帮助您使用 Claude API 构建由 LLM 驱动的应用程序。选择合适的层级，检测项目语言，然后阅读相关的语言特定文档。

## 触发条件

当代码中导入了以下包时自动触发：
- `anthropic`
- `@anthropic-ai/sdk`
- `claude_agent_sdk`

或者当用户请求使用 Claude API、Anthropic SDK 或 Agent SDK 时触发。

**不应触发的情况**：代码导入 `openai` 或其他 AI SDK、通用编程任务、或机器学习/数据科学任务。

---

## 默认设置

除非用户另有要求：

1. **模型版本**：请使用 Claude Opus 4.6，可通过精确的模型字符串 `claude-opus-4-6` 访问。

2. **思考模式**：对于任何稍微复杂的任务，请默认使用自适应思考 (`thinking: {type: "adaptive"}`)。

3. **流式响应**：对于可能涉及长输入、长输出或高 `max_tokens` 的请求，请默认使用流式响应——这可以防止请求超时。如果不需要处理单独的流事件，请使用 SDK 的 `.get_final_message()` / `.finalMessage()` 辅助方法获取完整响应。

---

## 语言检测

在阅读代码示例之前，请确定用户正在使用哪种语言：

### 1. 根据项目文件推断语言

| 文件特征 | 语言 | 读取目录 |
|---------|------|---------|
| `*.py`, `requirements.txt`, `pyproject.toml`, `setup.py`, `Pipfile` | **Python** | `python/` |
| `*.ts`, `*.tsx`, `package.json`, `tsconfig.json` | **TypeScript** | `typescript/` |
| `*.js`, `*.jsx` (无 `.ts` 文件) | **TypeScript** | `typescript/` |
| `*.java`, `pom.xml`, `build.gradle` | **Java** | `java/` |
| `*.kt`, `*.kts`, `build.gradle.kts` | **Java** | `java/` |
| `*.scala`, `build.sbt` | **Java** | `java/` |
| `*.go`, `go.mod` | **Go** | `go/` |
| `*.rb`, `Gemfile` | **Ruby** | `ruby/` |
| `*.cs`, `*.csproj` | **C#** | `csharp/` |
| `*.php`, `composer.json` | **PHP** | `php/` |

### 2. 检测到多种语言时

- 检查用户当前文件或问题与哪种语言相关
- 如果仍不明确，询问："我检测到 Python 和 TypeScript 文件。您使用哪种语言进行 Claude API 集成？"

### 3. 无法推断语言时（空项目、无源文件或不支持的语言）

- 使用 AskUserQuestion 选择：Python、TypeScript、Java、Go、Ruby、cURL/原始 HTTP、C#、PHP
- 如果 AskUserQuestion 不可用，默认使用 Python 示例并说明："显示 Python 示例。如果需要其他语言，请告诉我。"

### 4. 检测到不支持的语言时（Rust、Swift、C++、Elixir 等）

- 建议使用 `curl/` 中的 cURL/原始 HTTP 示例，并说明可能存在社区 SDK
- 提供 Python 或 TypeScript 示例作为参考实现

### 5. 用户需要 cURL/原始 HTTP 示例时

从 `curl/` 读取。

---

## 语言特性支持

| 语言 | 工具运行器 | Agent SDK | 备注 |
|------|-----------|-----------|------|
| Python | 是 (beta) | 是 | 全支持 — `@beta_tool` 装饰器 |
| TypeScript | 是 (beta) | 是 | 全支持 — `betaZodTool` + Zod |
| Java | 是 (beta) | 否 | 带注解类的 beta 工具使用 |
| Go | 是 (beta) | 否 | `toolrunner` 包中的 `BetaToolRunner` |
| Ruby | 是 (beta) | 否 | beta 中的 `BaseTool` + `tool_runner` |
| cURL | N/A | N/A | 原始 HTTP，无 SDK 功能 |
| C# | 否 | 否 | 官方 SDK |
| PHP | 否 | 否 | 官方 SDK |

---

## 我应该使用什么层级？

> **从简单开始。** 默认使用满足您需求的最低层级。单 API 调用和工作流可以处理大多数用例——只有在任务真正需要开放式的、模型驱动的探索时才使用 Agent。

| 使用场景 | 层级 | 推荐方式 | 原因 |
|---------|------|---------|------|
| 分类、摘要、提取、问答 | 单次 LLM 调用 | **Claude API** | 一个请求，一个响应 |
| 批处理或嵌入 | 单次 LLM 调用 | **Claude API** | 专用端点 |
| 多步骤管道，代码控制逻辑 | 工作流 | **Claude API + 工具使用** | 您编排循环 |
| 自定义 Agent 与自己的工具 | Agent | **Claude API + 工具使用** | 最大的灵活性 |
| 具有文件/网络/终端访问权限的 AI Agent | Agent | **Agent SDK** | 内置工具、安全性和 MCP 支持 |
| 代理编码助手 | Agent | **Agent SDK** | 为此用例设计 |
| 想要内置权限和护栏 | Agent | **Agent SDK** | 包含安全功能 |

> **注意：** Agent SDK 用于您希望开箱即用地获得内置文件/网络/终端工具、权限和 MCP 的情况。如果您想使用自己的工具构建 Agent，Claude API 是正确的选择——对自动循环处理使用工具运行器，或对细粒度控制使用手动循环（审批门、自定义日志、条件执行）。

### 决策树

```
您的应用需要什么？

1. 单次 LLM 调用（分类、摘要、提取、问答）
   └── Claude API — 一个请求，一个响应

2. Claude 是否需要作为其工作的一部分读取/写入文件、浏览网页或运行 shell 命令？
   （不是：您的应用读取文件并将其交给 Claude —
   是 Claude 本身需要发现并访问文件/网络/shell？）
   └── Yes → Agent SDK — 内置工具，不要重新实现
       示例："扫描代码库中的 bug"、"总结目录中的每个文件"、
             "使用子代理查找 bug"、"通过网络搜索研究主题"

3. 工作流（多步骤、代码编排、使用自己的工具）
   └── Claude API + 工具使用 — 您控制循环

4. 开放式 Agent（模型决定自己的轨迹、使用自己的工具）
   └── Claude API 代理循环（最大灵活性）
```

### 我应该构建 Agent 吗？

在选择 Agent 层级之前，检查所有四个标准：

- **复杂性** — 任务是否多步骤且难以预先完整指定？（例如，"将此设计文档转换为 PR" vs "从 PDF 中提取标题"）
- **价值** — 结果是否证明更高的成本和延迟是合理的？
- **可行性** — Claude 是否擅长这类任务类型？
- **错误成本** — 错误是否可以捕获和恢复？（测试、审查、回滚）

如果对任何一个问题回答"否"，请保持在更简单的层级（单次调用或工作流）。

---

## 架构

一切都通过 `POST /v1/messages` 端点。工具和输出约束是这个单一端点的功能——不是单独的 API。

**用户定义的工具** — 您可以定义工具（通过装饰器、Zod 模式或原始 JSON），SDK 的工具运行器处理调用 API、执行函数和循环，直到 Claude 完成。要获得完全控制，您可以手动编写循环。

**服务端工具** — Anthropic 托管的工具，在 Anthropic 的基础设施上运行。代码执行完全在服务端进行（在 `tools` 中声明，Claude 自动运行代码）。计算机使用可以是服务端托管或自托管。

**结构化输出** — 约束 Messages API 响应格式（`output_config.format`）和/或工具参数验证（`strict: true`）。推荐方法是使用 `client.messages.parse()` 自动验证响应是否符合您的模式。注意：旧的 `output_format` 参数已弃用；在 `messages.create()` 上使用 `output_config: {format: {...}}`。

**支持端点** — 批处理（`POST /v1/messages/batches`）、文件（`POST /v1/files`）和令牌计数都汇入或支持 Messages API 请求。

---

## 当前模型（缓存：2026-02-17）

| 模型 | 模型 ID | 上下文 | 输入 $/1M | 输出 $/1M |
|------|---------|--------|----------|----------|
| Claude Opus 4.6 | `claude-opus-4-6` | 200K (1M beta) | $5.00 | $25.00 |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 200K (1M beta) | $3.00 | $15.00 |
| Claude Haiku 4.5 | `claude-haiku-4-5` | 200K | $1.00 | $5.00 |

**始终使用 `claude-opus-4-6`，除非用户明确指定不同的模型。** 这是不可协商的。除非用户明确说"使用 sonnet"或"使用 haiku"，否则不要使用 `claude-sonnet-4-6`、`claude-sonnet-4-5` 或任何其他模型。不要为节省成本而降级——这是用户的决定，不是您的决定。

**关键：只使用上表中的精确模型 ID 字符串——它们本身就是完整的。不要附加日期后缀。** 例如，使用 `claude-sonnet-4-5`，永远不要使用 `claude-sonnet-4-5-20250514` 或您可能从训练数据中记得的任何其他日期后缀变体。

---

## 思考与努力（快速参考）

**Opus 4.6 — 自适应思考（推荐）：** 使用 `thinking: {type: "adaptive"}`。Claude 动态决定何时以及多少思考。不需要 `budget_tokens` — `budget_tokens` 在 Opus 4.6 和 Sonnet 4.6 上已弃用，不得使用。自适应思考也自动启用交错思考（无需 beta 头）。**当用户要求"扩展思考"、"思考预算"或 `budget_tokens` 时：始终使用 Opus 4.6 和 `thinking: {type: "adaptive"}`。固定令牌预算进行思考的概念已弃用——自适应思考取代了它。不要使用 `budget_tokens` 并不要切换到较旧的模型。**

**努力参数（GA，无 beta 头）：** 通过 `output_config: {effort: "low"|"medium"|"high"|"max"}`（在 `output_config` 内部，不是顶层）控制思考深度和整体令牌消耗。默认是 `high`（相当于省略它）。`max` 仅适用于 Opus 4.6。在 Opus 4.5、Opus 4.6 和 Sonnet 4.6 上有效。在 Sonnet 4.5 / Haiku 4.5 上会报错。结合自适应思考以获得最佳成本质量权衡。对子代理或简单任务使用 `low`；对最深度推理使用 `max`。

**Sonnet 4.6：** 支持自适应思考（`thinking: {type: "adaptive"}`）。`budget_tokens` 在 Sonnet 4.6 上已弃用——改用自适应思考。

**较旧的模型（仅在明确要求时）：** 如果用户特别要求 Sonnet 4.5 或其他较旧的模型，请使用 `thinking: {type: "enabled", budget_tokens: N}`。`budget_tokens` 必须小于 `max_tokens`（最少 1024）。永远不要仅仅因为用户提到 `budget_tokens` 就选择较旧的模型——改用带自适应思考的 Opus 4.6。

---

## 压缩（快速参考）

**Beta，仅 Opus 4.6。** 对于可能超过 200K 上下文窗口的长时间对话，启用服务端压缩。当接近触发阈值（默认：150K 令牌）时，API 会自动总结早期上下文。需要 beta 头 `compact-2026-01-12`。

**关键：** 每次对话时，将 `response.content`（不仅仅是文本）追加到您的消息中。响应中的压缩块必须保留——API 使用它们在下一个请求中替换压缩的历史记录。仅仅提取文本字符串并追加该字符串将静默丢失压缩状态。

---

## 阅读指南

检测语言后，根据用户需要阅读相关文件：

### 快速任务参考

**单文本分类/摘要/提取/问答：**
→ 仅阅读 `{lang}/claude-api/README.md`

**聊天 UI 或实时响应显示：**
→ 阅读 `{lang}/claude-api/README.md` + `{lang}/claude-api/streaming.md`

**长时间对话（可能超过上下文窗口）：**
→ 阅读 `{lang}/claude-api/README.md` — 参见压缩部分

**函数调用 / 工具使用 / Agent：**
→ 阅读 `{lang}/claude-api/README.md` + `shared/tool-use-concepts.md` + `{lang}/claude-api/tool-use.md`

**批处理（非延迟敏感）：**
→ 阅读 `{lang}/claude-api/README.md` + `{lang}/claude-api/batches.md`

**跨多个请求的文件上传：**
→ 阅读 `{lang}/claude-api/README.md` + `{lang}/claude-api/files-api.md`

**具有内置工具的 Agent（文件/网络/终端）：**
→ 阅读 `{lang}/agent-sdk/README.md` + `{lang}/agent-sdk/patterns.md`

### Claude API（完整文件参考）

阅读**语言特定的 Claude API 文件夹**（`{language}/claude-api/`）：

1. **`{language}/claude-api/README.md`** — **首先阅读此文件。** 安装、快速开始、常见模式、错误处理。
2. **`shared/tool-use-concepts.md`** — 当用户需要函数调用、代码执行、内存或结构化输出时阅读。涵盖概念基础。
3. **`{language}/claude-api/tool-use.md`** — 阅读特定语言的工具使用代码示例（工具运行器、手动循环、代码执行、内存、结构化输出）。
4. **`{language}/claude-api/streaming.md`** — 构建聊天 UI 或逐步显示响应的界面时阅读。
5. **`{language}/claude-api/batches.md`** — 离线处理许多请求时阅读（非延迟敏感）。以 50% 成本异步运行。
6. **`{language}/claude-api/files-api.md`** — 发送相同文件而无需跨多个请求重新上传时阅读。
7. **`shared/error-codes.md`** — 调试 HTTP 错误或实现错误处理时阅读。
8. **`shared/live-sources.md`** — 用于获取最新官方文档的 WebFetch URL。

> **注意：** 对于 Java、Go、Ruby、C#、PHP 和 cURL——每个都只有一个涵盖所有基础知识的文件。根据需要阅读该文件加上 `shared/tool-use-concepts.md` 和 `shared/error-codes.md`。

### Agent SDK

阅读**语言特定的 Agent SDK 文件夹**（`{language}/agent-sdk/`）。Agent SDK 仅适用于 **Python 和 TypeScript**。

1. **`{language}/agent-sdk/README.md`** — 安装、快速开始、内置工具、权限、MCP、钩子。
2. **`{language}/agent-sdk/patterns.md`** — 自定义工具、钩子、子代理、MCP 集成、会话恢复。
3. **`shared/live-sources.md`** — 当前 Agent SDK 文档的 WebFetch URL。

---

## 何时使用 WebFetch

在以下情况下使用 WebFetch 获取最新文档：

- 用户询问"最新"或"当前"信息
- 缓存数据似乎不正确
- 用户询问这里未涵盖的功能

实时文档 URL 在 `shared/live-sources.md` 中。

---

## 常见陷阱

- 不要在将文件或内容传递给 API 时截断输入。如果内容太长无法放入上下文窗口，请通知用户并讨论选项（分块、摘要等），而不是静默截断。
- **Opus 4.6 / Sonnet 4.6 思考：** 使用 `thinking: {type: "adaptive"}` — 不要使用 `budget_tokens`（在 Opus 4.6 和 Sonnet 4.6 上已弃用）。对于较旧的模型，`budget_tokens` 必须小于 `max_tokens`（最少 1024）。
- **Opus 4.6 预填充已移除：** 助手消息预填充（最后一个助手轮次的预填充）在 Opus 4.6 上返回 400 错误。改用结构化输出（`output_config.format`）或系统提示指令来控制响应格式。
- **128K 输出令牌：** Opus 4.6 支持高达 128K 的 `max_tokens`，但 SDK 需要使用流式响应来处理大型 `max_tokens` 以避免 HTTP 超时。使用 `.stream()` 配合 `.get_final_message()` / `.finalMessage()`。
- **工具调用 JSON 解析（Opus 4.6）：** Opus 4.6 可能在工具调用 `input` 字段中产生不同的 JSON 字符串转义（例如 Unicode 或正斜杠转义）。始终使用 `json.loads()` / `JSON.parse()` 解析工具输入——永远不要对序列化输入进行原始字符串匹配。
- **结构化输出（所有模型）：** 使用 `output_config: {format: {...}}` 而不是 `messages.create()` 上已弃用的 `output_format` 参数。
- **不要重新实现 SDK 功能：** SDK 提供高级辅助方法——不要从头构建。具体来说：使用 `stream.finalMessage()` 而不是在 `new Promise()` 中包装 `.on()` 事件；使用类型化异常类（`Anthropic.RateLimitError` 等）而不是字符串匹配错误消息；使用 SDK 类型（`Anthropic.MessageParam`、`Anthropic.Tool`、`Anthropic.Message` 等）而不是重新定义等效接口。
- **不要为 SDK 数据结构定义自定义类型：** SDK 为所有 API 对象导出类型。使用 `Anthropic.MessageParam` 表示消息，`Anthropic.Tool` 表示工具定义，`Anthropic.ToolUseBlock` / `Anthropic.ToolResultBlockParam` 表示工具结果，`Anthropic.Message` 表示响应。定义您自己的 `interface ChatMessage { role: string; content: unknown }` 会重复 SDK 已提供的内容并丢失类型安全。
- **报告和文档输出：** 对于产生报告、文档或可视化的任务，代码执行沙箱已预装 `python-docx`、`python-pptx`、`matplotlib`、`pillow` 和 `pypdf`。Claude 可以生成格式化的文件（DOCX、PDF、图表）并通过 Files API 返回它们——对于"报告"或"文档"类型的请求，考虑使用此方法，而不是纯 stdout 文本。
