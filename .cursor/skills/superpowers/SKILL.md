---
name: superpowers
description: Boosts execution for multi-step engineering tasks by enforcing a “plan-first, then implement” workflow, tight scope control, and fast verification loops. Use when the user asks for “superpowers/超能力/提效/加速/一键/自动化”, when work spans 3+ steps or multiple files, or when tasks involve Android (Gradle/Manifest/签名/渠道/Deep Link) or web download pages and Apple AASA/Universal Links.
---

# Superpowers

本技能用于把复杂任务做得更快、更稳：先对齐目标与约束，再最小化改动落地，最后快速验证闭环。

## 何时启用

- 用户明确提到：`superpowers` / “超能力” / “提效” / “加速” / “一键” / “自动化”
- 任务天然是多步骤（3+）或跨多个文件/模块
- 涉及 Android：Gradle、Manifest、签名、渠道、Deep Link、包名/版本号
- 涉及 Web 下载页与深链：HTML 下载页、`.well-known/apple-app-site-association`、Universal Links

## 总原则（强约束）

- 先出方案，明确验收，再执行落地。
- 只改与目标直接相关的文件与代码；避免“顺手重构”。
- 优先复用项目已有模式与命名；不引入不必要依赖。
- 任何改动都要有可验证的结果（能跑则跑，能校验则校验）。
- 默认使用简体中文沟通。

## 标准工作流

### 0. 快速澄清（不反问式）

在不打断用户的前提下，基于上下文做合理默认并记录：

- 目标是什么、成功标准是什么
- 约束是什么（目录/平台/兼容性/上线时间/隐私合规）
- 影响范围大概在哪些文件/模块

### 1. 先出方案（短而可执行）

输出一段简短方案，必须包含：

- **目标**：一句话
- **约束**：最多 3 条
- **执行步骤**：3–7 条，按依赖顺序
- **风险点**：最多 3 条（如回滚、兼容性、SEO/深链影响）
- **验证方式**：怎么确认成功（命令、手工步骤或可观察现象）

### 2. 执行落地（工具优先、批量并行）

- 先读文件再改，改动保持聚焦。
- 能并行获取的信息并行做（例如同时搜索/读取多个关键文件）。
- 改完立即做最小验证：
  - Android：构建/单测/运行关键流程（条件允许时）
  - Web：本地打开/简单静态检查/关键链接与深链匹配检查

### 3. 收口交付（让用户可直接用）

交付时用简短总结：

- 做了什么（1–3 条）
- 怎么验证（1–3 条）
- 如有风险/回滚点，明确说明

## 项目相关默认知识（本仓库）

- 项目主路径：`PMDemo`
- 产品文档：`好看智能相框/docs/`（含 `手机端/`、`相框端/`）

## 输出模板

### 方案模板

- 目标：…
- 约束：…
- 步骤：
  - …
  - …
  - …
- 风险：…
- 验证：…

