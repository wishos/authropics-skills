# Skill Creator - 创建和改进 Skills 的完整指南

## 触发条件

当用户想要从头创建技能、编辑或优化现有技能、运行评估测试技能性能、通过方差分析进行基准测试技能性能，或优化技能的描述以提高触发准确性时使用此 Skill。

---

## 概述

一个用于创建新 Skills 并迭代改进它们的 Skill。

从高层次来看，创建 Skill 的过程如下：

- 决定您希望 Skill 做什么以及它应该如何大致完成
- 起草 Skill
- 创建一些测试提示，并在带有 Skill 访问权限的 Claude 上运行它们
- 帮助用户定性和定量地评估结果
  - 当运行在后台发生时，起草一些定量评估（如果没有的话，如果有，您可以根据认为需要更改的内容使用或修改它们）。然后向用户解释它们（如果已经存在，解释已经存在的）
  - 使用 `eval-viewer/generate_review.py` 脚本向用户展示结果供他们查看，同时让他们查看定量指标
- 根据用户对结果的评估反馈重写 Skill（以及如果定量基准显示任何明显缺陷）
- 重复直到满意
- 扩大测试集并再次尝试更大规模

您在使用此 Skill 时的工作是找出用户在这个过程的哪个阶段，然后加入并帮助他们推进这些阶段。例如，也许他们像"我想为 X 制作一个 Skill"。您可以帮助缩小他们的意思，起草测试用例，弄清楚他们想要如何评估，运行所有提示，并重复。

另一方面，也许他们已经有了 Skill 的草稿。在这种情况下，您可以直接进入评估/迭代循环部分。

当然，您应该始终保持灵活，如果用户像"我不需要运行一堆评估，只是和我一起感受"，您可以这样做。

然后，在 Skill 完成后（但顺序again是灵活的），您还可以运行 Skill 描述优化器，我们有一个单独的脚本，用于优化 Skill 的触发准确性。

---

## 与用户沟通

Skill Creator 可能被各种熟悉程度的用户使用。所以请注意上下文线索以了解如何措辞您的交流！默认情况下，只是给您一些概念：

- "evaluation" 和 "benchmark" 是边缘情况，但可以
- 对于 "JSON" 和 "assertion"，在未解释使用它们之前，您希望看到用户确实知道它们是什么的认真线索

如果您有疑问，可以简短地解释术语，如果您不确定用户是否会理解，也可以用简短定义来澄清。

---

## 创建 Skill

### 捕获意图

首先了解用户的意图。当前对话可能已经包含他们想要捕获的工作流程（例如，他们说"将其转换为 Skill"）。如果是这样，首先从对话历史中提取答案——使用的工具、步骤顺序、用户所做的更正、观察到的输入/输出格式。用户可能需要填补空白，并应在继续下一步之前确认。

1. 这个 Skill 应该让 Claude 能够做什么？
2. 这个 Skill 应该在什么时候触发？（什么用户短语/上下文）
3. 预期的输出格式是什么？
4. 我们是否应该设置测试用例来验证 Skill 有效？具有客观可验证输出的 Skills（文件转换、数据提取、代码生成、固定工作流程步骤）受益于测试用例。具有主观输出的 Skills（写作风格、艺术）通常不需要它们。根据 Skill 类型建议适当的默认值，但让用户决定。

### 面试和研究

主动询问边缘情况、输入/输出格式、示例文件、成功标准和依赖项。在您准备好这部分之前，不要编写测试提示。

检查可用的 MCP——如果对研究有用（搜索文档、查找类似 Skills、查找最佳实践），如果可用，通过子代理并行研究，否则内联。准备好背景知识以减少用户的负担。

### 编写 SKILL.md

根据用户面试，填写这些组件：

- **name**：Skill 标识符
- **description**：何时触发，它做什么。这是主要的触发机制——包括 Skill 做什么以及什么时候使用的具体上下文。所有"何时使用"信息都放在这里，而不是正文。注意：目前 Claude 有"undertrigger"Skills 的倾向——在它们有用时不使用它们。为了解决这个问题，请将 Skill 描述写得稍微"激进"一点。例如，不要写"How to build a simple fast dashboard to display internal Anthropic data."，您可以写"How to build a simple fast dashboard to display internal Anthropic data. Make sure to use this skill whenever the user mentions dashboards, data visualization, internal metrics, or wants to display any kind of company data, even if they don't explicitly ask for a 'dashboard.'"
- **compatibility**：必需工具、依赖项（可选，很少需要）
- **Skill 的其余部分 :)**

### Skill 写作指南

#### Skill 的剖析

```
skill-name/
├── SKILL.md (必需)
│   ├── YAML 头部 (name, description 必需)
│   └── Markdown 指令
└── 捆绑资源 (可选)
    ├── scripts/    - 用于确定性/重复任务的可执行代码
    ├── references/ - 根据需要加载到上下文的文档
    └── assets/     - 输出中使用的文件（模板、图标、字体）
```

#### 渐进式披露

Skills 使用三级加载系统：
1. **元数据**（name + description）- 始终在上下文中（~100 字）
2. **SKILL.md 正文** - 只要 Skill 触发就在上下文中（理想 <500 行）
3. **捆绑资源** - 根据需要（无限制，脚本可以执行而无需加载）

这些字数是近似值，如果您需要，可以随意更长。

**关键模式：**
- 将 SKILL.md 保持在 500 行以下；如果您接近这个限制，添加额外的层次结构层次，并清楚指出使用 Skill 的模型应该去哪里跟进
- 从 SKILL.md 清楚引用文件，并在何时阅读它们时提供指导
- 对于大型参考文件（>300 行），包含目录

**领域组织**：当 Skill 支持多个域/框架时，按变体组织：
```
cloud-deploy/
├── SKILL.md (工作流 + 选择)
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```
Claude 只读取相关的参考文件。

#### 不可惊喜原则

不言自明，Skills 不得包含恶意软件、漏洞利用代码或可能损害系统安全的内容。如果描述的内容，Skill 的内容不应该让用户感到意外。不要配合创建误导性 Skills 或旨在促进未经授权访问、数据外泄或其他恶意活动的请求。像"扮演 XYZ"的事情是可以的。

#### 写作模式

在指令中更喜欢使用祈使形式。

**定义输出格式** - 您可以这样写：
```markdown
## 报告结构
始终使用这个精确模板：
# [标题]
## 执行摘要
## 关键发现
## 建议
```

**示例模式** - 包括示例很有用。您可以这样格式化（但如果示例中有"输入"和"输出"，您可能想要稍微偏离）：
```markdown
## 提交消息格式
**示例 1：**
输入：Added user authentication with JWT tokens
输出：feat(auth): implement JWT-based authentication
```

### 写作风格

尽量解释模型为什么事情重要，而不是用沉重的 MUST。使用心智理论，努力使 Skill 通用，而不是针对特定示例非常狭窄。先起草，然后以新的眼光看待它并改进它。

### 测试用例

起草 Skill 草稿后，提出 2-3 个现实的测试提示——真正的用户实际会说的那种。与用户分享它们："这里有我想尝试的几个测试用例。这些看起来对吗，或者您想添加更多？"然后运行它们。

将测试用例保存到 `evals/evals.json`。此时不编写断言——只是提示。您将在下一步（运行正在进行时）起草断言。

```json
{
  "skill_name": "example-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "用户的任务提示",
      "expected_output": "预期结果描述",
      "files": []
    }
  ]
}
```

参见 `references/schemas.md` 获取完整模式（包括您稍后添加的 `assertions` 字段）。

---

## 运行和评估测试用例

本节是一个连续序列——不要中途停止。不要使用 `/skill-test` 或任何其他测试技能。

将结果放在 `<skill-name>-workspace/` 中，作为技能目录的兄弟。在工作空间内，按迭代组织结果（`iteration-1/`、`iteration-2/` 等），在其中，每个测试用例有一个目录（`eval-0/`、`eval-1/` 等）。不要预先创建所有内容——随着您的进行创建目录。

### 第 1 步：在同一轮中生成所有运行（with-skill 和 baseline）

对于每个测试用例，在同一轮中生成两个子代理——一个带 Skill，一个不带。这很重要：先生成 with-skill 运行，然后为 baselines 回来。立即启动一切，这样一切都大约同时完成。

**With-skill 运行：**

```
执行这个任务：
- Skill 路径：<path-to-skill>
- 任务：<eval prompt>
- 输入文件：<eval files if any, or "none">
- 保存输出到：<workspace>/iteration-<N>/eval-<ID>/with_skill/outputs/
- 保存的输出：<用户关心的东西——例如 ".docx 文件"、"最终 CSV">
```

**Baseline 运行**（相同的提示，但 baseline 取决于上下文）：
- **创建新 Skill**：根本没有 Skill。相同的提示，没有 Skill 路径，保存到 `without_skill/outputs`。
- **改进现有 Skill**：旧版本。在编辑之前，快照 Skill（`cp -r <skill-path> <workspace>/skill-snapshot/`），然后将 baseline 子代理指向快照。保存到 `old_skill/outputs`。

为每个测试用例编写 `eval_metadata.json`（此时断言可以为空）。给每个 eval 一个基于其测试内容的描述性名称——不仅仅是"eval-0"。也为此目录使用此名称。如果此迭代使用新的或修改的 eval 提示，为每个新的 eval 目录创建这些文件——不要假设它们从上一次迭代继承。

```json
{
  "eval_id": 0,
  "eval_name": "descriptive-name-here",
  "prompt": "用户的任务提示",
  "assertions": []
}
```

### 第 2 步：运行进行时，起草断言

不要只是等待运行完成——您可以有效地利用这段时间。为每个测试用例起草定量断言，并向用户解释它们。如果 `evals/evals.json` 中已经存在断言，审查它们并解释它们检查什么。

好的断言是客观可验证的并且有描述性名称——它们应该在基准查看器中清晰阅读，这样有人浏览结果立即理解每个检查什么。具有主观输出的 Skills（写作风格、设计质量）更好地定性评估——不要将断言强加于需要人类判断的事情上。

更新 `eval_metadata.json` 文件和 `evals/evals.json`，一旦起草了断言。还要向用户解释他们在查看器中会看到什么——定性和定量基准。

### 第 3 步：运行时，捕获计时数据

当每个子代理任务完成时，您会收到包含 `total_tokens` 和 `duration_ms` 的通知。立即将此数据保存到运行目录中的 `timing.json`：

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3
}
```

这是捕获此数据的唯一机会——它通过任务通知传递，不会其他地方保留。处理每个通知，因为它到达，而不是试图批量处理。

### 第 4 步：评分、聚合，并启动查看器

所有运行完成后：

1. **为每个运行评分** - 生成一个评分器子代理（或内联评分），读取 `agents/grader.md` 并针对输出评估每个断言。将结果保存到每个运行目录中的 `grading.json`。grading.json expectations 数组必须使用字段 `text`、`passed` 和 `evidence`（不是 `name`/`met`/`details` 或其他变体）——查看器依赖这些精确字段名称。对于可以编程检查的断言，编写并运行脚本而不是凭眼检查——脚本更快、更可靠，并且可以跨迭代重用。

2. **聚合成基准** - 从 skill-creator 目录运行聚合脚本：
   ```bash
   python -m scripts.aggregate_benchmark <workspace>/iteration-N --skill-name <name>
   ```
   这将产生 `benchmark.json` 和 `benchmark.md`，包含每个配置的 pass_rate、时间和 tokens，带有 mean ± stddev 和 delta。如果手动生成 benchmark.json，请参见 `references/schemas.md` 获取查看器期望的确切模式。
   将每个 with_skill 版本放在其 baseline 对应物之前。

3. **做分析师检查** - 阅读基准数据并浮出聚合统计可能隐藏的模式。参见 `agents/analyzer.md`（"分析基准结果"部分）了解要寻找什么——像总是通过的断言（非歧视）、高方差评估（可能是 flaky）和时间/tokens 权衡。

4. **启动查看器**，同时显示定性输出和定量数据：
   ```bash
   nohup python <skill-creator-path>/eval-viewer/generate_review.py \
     <workspace>/iteration-N \
     --skill-name "my-skill" \
     --benchmark <workspace>/iteration-N/benchmark.json \
     > /dev/null 2>&1 &
   VIEWER_PID=$!
   ```
   对于迭代 2+，还要传递 `--previous-workspace <workspace>/iteration-<N-1>`。

   **同事/无头环境**：如果 `webbrowser.open()` 不可用或环境没有显示，使用 `--static <output_path>` 写入独立 HTML 文件而不是启动服务器。反馈将作为 `feedback.json` 文件下载，当用户点击"Submit All Reviews"。下载后，将 `feedback_json` 复制到工作空间目录，以便下一次迭代获取。

请使用 generate_review.py 创建查看器；无需编写自定义 HTML。

5. **告诉用户**类似："我在浏览器中打开了结果。有两个选项卡——'Outputs' 允许您点击每个测试用例并留下反馈，'Benchmark' 显示定量比较。完成后，回到这里让我知道。"

### 用户在查看器中看到的内容

"Outputs"选项卡一次显示一个测试用例：
- **Prompt**：给出的任务
- **Output**：Skill 产生的文件，在可能的情况下内联渲染
- **Previous Output**（迭代 2+）：显示上次迭代输出的折叠部分
- **Formal Grades**（如果运行了评分）：显示断言通过/失败的折叠部分
- **Feedback**：一个自动保存的文本框，因为他们输入
- **Previous Feedback**（迭代 2+）：他们上次的评论，显示在文本框下方

"Benchmark"选项卡显示统计摘要：每个配置的 pass rates、时间和 token 使用，带有 per-eval 分解和分析师观察。

导航通过上一个/下一个按钮或箭头键。完成后，他们点击"Submit All Reviews"将所有反馈保存到 `feedback.json`。

### 第 5 步：阅读反馈

当用户告诉您他们完成时，阅读 `feedback.json`：

```json
{
  "reviews": [
    {"run_id": "eval-0-with_skill", "feedback": "chart is missing axis labels", "timestamp": "..."},
    {"run_id": "eval-1-with_skill", "feedback": "", "timestamp": "..."},
    {"run_id": "eval-2-with_skill", "feedback": "perfect, love this", "timestamp": "..."}
  ],
  "status": "complete"
}
```

空反馈意味着用户认为它没问题。将改进重点放在用户有具体投诉的测试用例上。

完成后停止查看器服务器：

```bash
kill $VIEWER_PID 2>/dev/null
```

---

## 改进 Skill

这是循环的核心。您已经运行了测试用例，用户已经查看了结果，现在您需要根据他们的反馈使 Skill 变得更好。

### 如何思考改进

1. **从反馈中归纳。** 这里发生的宏观事情是，我们正试图创建可以跨许多不同提示使用一百万次的 Skills（可能literally，也许更多）。您和用户只是在这里迭代几个例子，因为它有助于移动得更快。用户知道这些例子并深入了解，快速评估新输出。但是，如果您和用户共同开发的 Skill 只对这些例子有效，那它就没有用了。不要放入棘手的过度拟合更改，或压迫性限制性的 MUST，如果有顽固的问题，您可以尝试分支并使用不同的比喻，或推荐不同的工作模式。相对便宜可以尝试，也许您会找到很棒的东西。

2. **保持提示精简。** 去掉没有发挥作用的确保阅读成绩单，而不仅仅是最终输出——如果看起来 Skill 让模型浪费大量时间做无生产力的事情，您可以尝试删除让模型那样做的 Skill 部分，看看会发生什么。

3. **解释为什么。** 努力解释您要求模型做的一切的**为什么**。今天的 LLMs 很聪明。他们有很好的心智理论，当给予良好的框架时，可以超越 rote 指令，真正发挥作用。即使反馈来自用户是简洁或沮丧的，尝试真正理解任务以及用户为什么写他们写的内容，他们实际写的是什么，然后将这种理解传递到指令中。如果您发现自己在全大写中使用 ALWAYS 或 NEVER，或使用超级 rigid 结构，这是一个黄旗——如果可能，重新框并解释原因，以便模型理解您要求它的事情为什么重要。这是更人性化、更强大和更有效的方法。

4. **寻找跨测试用例的重复工作。** 读取测试运行的成绩单，注意子代理是否都独立编写了类似的辅助脚本或对某些事情采取相同的多步骤方法。如果所有 3 个测试用例导致子代理编写 `create_docx.py` 或 `build_chart.py`，那就是一个强烈的信号，Skill 应该捆绑该脚本。写一次，放在 `scripts/` 中，并告诉 Skill 使用它。这节省了未来每次调用的重新发明轮子。

这个任务相当重要（我们正试图在这里创造每年数十亿美元的经济价值！），您的时间不是障碍；慢慢来，真正深思熟虑。我建议起草一个修订版，然后新的眼光看待它并做出改进。真的尽最大努力进入用户的头脑，理解他们想要和需要什么。

### 迭代循环

改进 Skill 后：

1. 将您的改进应用到 Skill
2. 将所有测试用例重新运行到新的 `iteration-<N+1>/` 目录，包括 baseline 运行。如果是创建新 Skill，baseline 始终是 `without_skill`（无 Skill）——它在迭代中保持不变。如果是改进现有 Skill，使用您的判断什么作为 baseline 有意义：用户带来的原始版本，或上一次迭代。
3. 使用 `--previous-workspace` 指向上一次迭代启动审查器
4. 等待用户查看并告诉他们完成
5. 读取新反馈，再次改进，重复

继续直到：
- 用户说他们满意
- 反馈都是空的（一切看起来不错）
- 您没有取得有意义的进展

---

## 高级：盲比较

对于想要更严格地比较两个版本 Skill 的情况（例如，用户问"新版本实际上更好吗？"），有一个盲比较系统。 `agents/comparator.md` 和 `读取agents/analyzer.md` 了解详情。基本思想是：将两个输出提供给独立代理，不告诉它哪个是哪个，让它判断质量。然后分析为什么赢家赢了。

这是可选的，需要子代理，大多数用户不需要。人类审查循环通常就足够了。

---

## 描述优化

SKILL.md 头部中的描述字段是决定 Claude 是否调用 Skill 的主要机制。创建或改进 Skill 后，提供优化描述以获得更好的触发准确性。

### 第 1 步：生成触发评估查询

创建 20 个评估查询——应该是触发和不触发的混合。保存为 JSON：

```json
[
  {"query": "the user prompt", "should_trigger": true},
  {"query": "another prompt", "should_trigger": false}
]
```

查询必须是现实的，是 Claude Code 或 Claude.ai 用户实际会输入的。不是抽象请求，而是具体详细的请求。例如，文件路径、用户工作或个人情况的背景、列名和值、公司名称、URL。一些可能是小写或包含缩写或拼写错误或随意语音。使用不同长度的混合，专注于边缘情况，而不是使它们显而易见——用户将有机会签署它们。

坏："Format this data"、"Extract text from PDF"、"Create a chart"

好："ok so my boss just sent me this xlsx file (its in my downloads, called something like 'Q4 sales final FINAL v2.xlsx') and she wants me to add a column that shows the profit margin as a percentage. The revenue is in column C and costs are in column D i think"

对于**应该触发**的查询（8-10），考虑覆盖率。您希望同一意图的不同措辞——一些正式，一些随意。包括用户未明确命名 Skill 或文件类型但明确需要它的案例。扔进一些不常见用例，以及这个 Skill 与另一个竞争但应该赢的案例。

对于**不应该触发**的查询（8-10），最有价值的是near-misses——共享关键词或概念与 Skill 但实际需要不同东西的查询。考虑相邻域、模糊措辞，其中朴素关键词匹配会触发但不应该，以及触及 Skill 做的事情但在另一个工具更合适的上下文中查询。

关键要避免：不要使不应该触发的查询明显无关。"Write a fibonacci function" 作为 PDF Skill 的负面测试太容易了——它不测试任何东西。负面案例应该真正棘手。

### 第 2 步：与用户审查

使用 HTML 模板向用户展示评估集：

1. 从 `assets/eval_review.html` 读取模板
2. 替换占位符：
   - `__EVAL_DATA_PLACEHOLDER__` → 评估项目 JSON 数组（周围没有引号——它是一个 JS 变量赋值）
   - `__SKILL_NAME_PLACEHOLDER__` → Skill 的名称
   - `__SKILL_DESCRIPTION_PLACEHOLDER__` → Skill 的当前描述
3. 写入临时文件（例如 `/tmp/eval_review_<skill-name>.html`）并打开：`open /tmp/eval_review_<skill-name>.html`
4. 用户可以编辑查询，切换 should-trigger，添加/删除条目，然后点击"Export Eval Set"
5. 文件下载到 `~/Downloads/eval_set.json`——检查下载文件夹中的最新版本（如果有多个，例如 `eval_set (1).json`）

这一步很重要——坏的评估查询导致坏的描述。

### 第 3 步：运行优化循环

告诉用户："这需要一些时间——我会在后台运行优化循环并定期检查它。"

将评估集保存到工作空间，然后后台运行：

```bash
python -m scripts.run_loop \
  --eval-set <path-to-trigger-eval.json> \
  --skill-path <path-to-skill> \
  --model <model-id-powering-this-session> \
  --max-iterations 5 \
  --verbose
```

使用系统提示中的模型 ID（为当前会话供电的那个），这样触发测试与用户实际体验相匹配。

当它运行时，定期 tail 输出以向用户更新它在哪个迭代以及分数是什么样的。

这自动处理完整优化循环。它将评估集拆分为 60% 训练和 40% 保留测试，评估当前描述（运行每个查询 3 次以获得可靠的触发率），然后调用 Claude 根据失败的查询提出改进。它在训练和测试上重新评估每个新描述，迭代最多 5 次。完成后，它在浏览器中打开显示每个迭代结果的 HTML 报告，并返回带有 `best_description` 的 JSON——根据测试分数而不是训练分数选择，以避免过拟合。

### Skill 触发如何工作

了解触发机制有助于设计更好的评估查询。Skills 出现在 Claude 的 `available_skills` 列表中，其名称 + 描述，Claude 决定是否根据该描述咨询 Skill。需要知道的重要事情是，Claude 只为它不能轻易自行处理的任务咨询 Skills——简单、一步式查询如"读取这个 PDF"可能不会触发 Skill，即使描述完全匹配，因为 Claude 可以用基本工具直接处理它们。复杂、多步或专业化查询可靠地触发 Skills，当描述匹配时。

这意味着您的评估查询应该足够实质，Claude 实际上会受益于咨询 Skill。像"读取文件 X"这样的简单查询是糟糕的测试用例——无论描述质量如何，它们都不会触发 Skills。

### 第 4 步：应用结果

从 JSON 输出中获取 `best_description` 并更新 Skill 的 SKILL.md 头部。向用户展示之前/之后并报告分数。

---

## 打包和展示（仅当 `present_files` 工具可用）

检查您是否有权访问 `present_files` 工具。如果没有，跳过这一步。如果有，打包 Skill 并向用户展示 .skill 文件：

```bash
python -m scripts.package_skill <path/to/skill-folder>
```

打包后，将生成的 .skill 文件路径指向用户，以便他们可以安装它。

---

## Claude.ai 特定指令

在 Claude.ai 中，核心工作流相同（起草 → 测试 → 审查 → 改进 → 重复），但因为 Claude.ai 没有子代理，一些机制改变。这是需要调整的内容：

**运行测试用例**：没有子代理意味着没有并行执行。对于每个测试用例，读取 Skill 的 SKILL.md，然后按照其指令自己完成测试提示。逐一进行。这不如独立子代理严格（您编写 Skill 并且也运行它，所以您有完整上下文），但这是一个有用的健全性检查——人类审查步骤补偿。跳过 baseline 运行——只需使用 Skill 完成请求的任务。

**审查结果**：如果您无法打开浏览器（例如 Claude.ai 的 VM 没有显示，或者您在远程服务器上），完全跳过浏览器审查器。相反，直接在对话中呈现结果。对于每个测试用例，显示提示和输出。如果输出是用户需要看到的文件（如 .docx 或 .xlsx），将其保存到文件系统并告诉他们在哪里，以便他们可以下载和检查。Inline 询问反馈："这看起来怎么样？有什么您想改变的吗？"

**基准测试**：跳过定量基准测试——它依赖于没有子代理就没有意义的 baseline 比较。将重点放在用户的定性反馈上。

**迭代循环**：与之前相同——改进 Skill，重新运行测试用例，询问反馈——只是中间没有浏览器审查器。您仍然可以将结果组织到文件系统上的迭代目录中（如果您有的话）。

**描述优化**：此部分需要 `claude` CLI 工具（具体是 `claude -p`），仅在 Claude Code 中可用。在 Claude.ai 上跳过它。

**盲比较**：需要子代理。跳过它。

**打包**：`package_skill.py` 脚本在任何有 Python 和文件系统的地方工作。在 Claude.ai 上，您可以运行它，用户可以下载生成的 .skill 文件。

**更新现有 Skill**：用户可能要求您更新现有 Skill，而不是创建新 Skill。在这种情况下：
- **保留原始名称。** 注意 Skill 的目录名称和 `name` 头部字段——保持不变地使用。例如，如果安装的 Skill 是 `research-helper`，输出 `research-helper.skill`（不是 `research-helper-v2`）。
- **在编辑之前复制到可写位置。** 安装的 Skill 路径可能是只读的。复制到 `/tmp/skill-name/`，在那里编辑，并从副本打包。
- **如果手动打包，先暂存到 `/tmp/`**，然后复制到输出目录——由于权限，直接写入可能会失败。

---

## 同事特定指令

如果您在同事环境中，主要需要知道的是：

- 您有子代理，所以主要工作流（并行生成测试用例、运行 baseline、评分等）都可以工作。（但是，如果您遇到严重的超时问题，按顺序运行测试提示是可以的。）
- 您没有浏览器或显示，所以在生成评估查看器时，使用 `--static <output_path>` 写入独立 HTML 文件，而不是启动服务器。然后提供一个链接，用户可以点击在他们的浏览器中打开 HTML。
- 无论出于什么原因，同事环境似乎倾向于在运行测试后不生成评估查看器，所以重申：无论是在同事还是在 Claude Code 中，运行测试后，您应该始终生成评估查看器，以便人类在您自己评估输入之前查看测试用例，使用 `generate_review.py`（不是编写自己的花哨 html 代码）。对不起，但我会用大写：确保在你自己评估输入之前生成评估查看器*。
- 反馈不同：因为没有运行服务器，查看器的"Submit All Reviews"按钮将将 `feedback.json` 下载为文件。然后您可以从那里读取它（您可能必须先请求访问）。
- 打包工作——`package_skill.py` 只需要 Python 和文件系统。
- 描述优化（`run_loop.py` / `run_eval.py`）应该在同事环境中正常工作，因为它使用 `claude -p` 通过子进程，而不是浏览器，但请保存它直到您完全完成制作 Skill 并且用户同意它处于良好状态。
- **更新现有 Skill**：用户可能要求您更新现有 Skill，而不是创建新 Skill。遵循上面 claude.ai 部分中的更新指南。

---

## 参考文件

agents/ 目录包含专业子代理的指令。在需要生成相关子代理时读取它们。

- `agents/grader.md` — 如何针对输出评估断言
- `agents/comparator.md` — 如何在两个输出之间进行盲 A/B 比较
- `agents/analyzer.md` — 如何分析为什么一个版本击败了另一个

references/ 目录有额外文档：
- `references/schemas.md` — evals.json、grading.json 等的 JSON 结构

再次强调核心循环：

- 找出 Skill 是关于什么的
- 起草或编辑 Skill
- 在测试提示上运行 claude-with-access-to-the-skill
- 与用户一起评估输出：
  - 创建 benchmark.json 并运行 `eval-viewer/generate_review.py` 以帮助用户查看它们
  - 运行定量评估
- 重复直到您和用户满意
- 打包最终 Skill 并将其返回给用户

如果您有这样的东西，请将步骤添加到您的 TodoList，以确保您不会忘记。如果您是在同事中，请具体将"创建 evals JSON 并运行 `eval-viewer/generate_review.py` 以便人类查看测试用例"放在您的 TodoList 中以确保它发生。

祝你好运！
