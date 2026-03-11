# Web Artifacts 构建器 (Web Artifacts Builder) 指南

## 触发条件

当需要使用现代前端 Web 技术（React、Tailwind CSS、shadcn/ui）创建复杂的多组件 claude.ai HTML artifacts 时使用此技能。用于需要状态管理、路由或 shadcn/ui 组件的复杂 artifacts——不适用于简单的单文件 HTML/JSX artifacts。

---

## 概述

要构建强大的前端 claude.ai artifacts，请按照以下步骤：

1. 使用 `scripts/init-artifact.sh` 初始化前端仓库
2. 通过编辑生成的代码开发您的 artifact
3. 使用 `scripts/bundle-artifact.sh` 将所有代码捆绑成单个 HTML 文件
4. 向用户展示 artifact
5. （可选）测试 artifact

**技术栈**：React 18 + TypeScript + Vite + Parcel（捆绑）+ Tailwind CSS + shadcn/ui

---

## 设计和样式指南

**非常重要**：为了避免通常所说的"AI slop"，避免使用过度的居中布局、紫色渐变、统一的圆角和 Inter 字体。

---

## 快速开始

### 步骤 1：初始化项目

运行初始化脚本创建新的 React 项目：
```bash
bash scripts/init-artifact.sh <project-name>
cd <project-name>
```

这将创建一个完全配置的项目，包含：
- ✅ React + TypeScript（通过 Vite）
- ✅ Tailwind CSS 3.4 带 shadcn/ui 主题系统
- ✅ 路径别名（`@/`）已配置
- ✅ 40+ shadcn/ui 组件预安装
- ✅ 所有 Radix UI 依赖项已包含
- ✅ 为捆绑配置了 Parcel（通过 .parcelrc）
- ✅ Node 18+ 兼容性（自动检测并固定 Vite 版本）

### 步骤 2：开发您的 Artifact

要构建 artifact，编辑生成的文件。请参阅下面的**常见开发任务**获取指导。

### 步骤 3：捆绑成单个 HTML 文件

将 React 应用捆绑成单个 HTML artifact：
```bash
bash scripts/bundle-artifact.sh
```

这将创建 `bundle.html` - 一个自包含的 artifact，所有 JavaScript、CSS 和依赖项都内联。这个文件可以直接在 Claude 对话中作为 artifact 共享。

**要求**：您的项目必须在根目录中有 `index.html`。

**脚本做了什么**：
- 安装捆绑依赖项（parcel、@parcel/config-default、parcel-resolver-tspaths、html-inline）
- 创建带路径别名支持的 `.parcelrc` 配置
- 使用 Parcel 构建（无 source maps）
- 使用 html-inline 将所有资源内联到单个 HTML

### 步骤 4：与用户共享 Artifact

最后，在对话中与用户共享捆绑的 HTML 文件，以便他们可以将其作为 artifact 查看。

### 步骤 5：测试/可视化 Artifact（可选）

注意：这是一个完全可选的步骤。仅在必要时或应要求执行。

要测试/可视化 artifact，使用可用工具（包括其他 Skills 或 Playwright 或 Puppeteer 等内置工具）。一般来说，避免提前测试 artifact，因为会增加请求和看到最终 artifact 之间的延迟。稍后，在展示 artifact 之后，如果请求或出现问题，再进行测试。

---

## shadcn/ui 组件

请参阅：https://ui.shadcn.com/docs/components
