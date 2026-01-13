---
name: web-artifacts-builder
description: 使用现代前端 Web 技术（React、Tailwind CSS、shadcn/ui）创建精细的多组件 claude.ai HTML 工件的工具套件。用于需要状态管理、路由或 shadcn/ui 组件的复杂工件——不适用于简单的单文件 HTML/JSX 工件。
license: 完整条款见 LICENSE.txt
---

# Web 工件构建器

要构建强大的前端 claude.ai 工件，请遵循以下步骤：
1. 使用 `scripts/init-artifact.sh` 初始化前端仓库
2. 通过编辑生成的代码开发你的工件
3. 使用 `scripts/bundle-artifact.sh` 将所有代码打包成单个 HTML 文件
4. 向用户展示工件
5. （可选）测试工件

**技术栈**：React 18 + TypeScript + Vite + Parcel（打包）+ Tailwind CSS + shadcn/ui

## 设计与样式指南

非常重要：为避免通常所说的"AI 废话"，避免使用过度的居中布局、紫色渐变、统一的圆角和 Inter 字体。

## 快速开始

### 步骤 1：初始化项目

运行初始化脚本创建新的 React 项目：
```bash
bash scripts/init-artifact.sh <project-name>
cd <project-name>
```

这将创建一个完全配置的项目，包含：
- ✅ React + TypeScript（通过 Vite）
- ✅ Tailwind CSS 3.4.1 配合 shadcn/ui 主题系统
- ✅ 路径别名（`@/`）已配置
- ✅ 40+ shadcn/ui 组件预安装
- ✅ 所有 Radix UI 依赖项已包含
- ✅ Parcel 配置用于打包（通过 .parcelrc）
- ✅ Node 18+ 兼容性（自动检测并锁定 Vite 版本）

### 步骤 2：开发你的工件

要构建工件，编辑生成的文件。参见下面的**常见开发任务**获取指导。

### 步骤 3：打包为单个 HTML 文件

要将 React 应用打包成单个 HTML 工件：
```bash
bash scripts/bundle-artifact.sh
```

这将创建 `bundle.html` - 一个自包含的工件，所有 JavaScript、CSS 和依赖项都已内联。此文件可以直接作为工件在 Claude 对话中共享。

**要求**：你的项目必须在根目录有一个 `index.html`。

**脚本做什么**：
- 安装打包依赖项（parcel、@parcel/config-default、parcel-resolver-tspaths、html-inline）
- 创建带有路径别名支持的 `.parcelrc` 配置
- 使用 Parcel 构建（无源映射）
- 使用 html-inline 将所有资源内联到单个 HTML 中

### 步骤 4：与用户共享工件

最后，在对话中与用户共享打包的 HTML 文件，以便他们可以将其作为工件查看。

### 步骤 5：测试/可视化工件（可选）

注意：这是一个完全可选的步骤。仅在必要或被请求时执行。

要测试/可视化工件，使用可用的工具（包括其他技能或内置工具如 Playwright 或 Puppeteer）。一般来说，避免预先测试工件，因为这会增加请求和看到完成工件之间的延迟。如果请求或出现问题，在展示工件后再测试。

## 参考

- **shadcn/ui 组件**：https://ui.shadcn.com/docs/components
