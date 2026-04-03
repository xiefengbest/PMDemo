# 好看智能相框 App — iOS 复刻指南

> **目标**：完美复刻 **`手机端/index.html`**（相对产品目录 `好看智能相框/`）的 UI 与交互，供 iOS 原生实现时作为唯一复刻依据。  
> **设计基准**：iPhone 14 Pro，逻辑宽度 390pt，使用 Safe Area 与系统字体。  
> 本文档已汇总 `UI设计规范`、`UI视觉规格文档`、`背景与圆角规格`、`发送任务逻辑说明` 等，复刻时以本文档 + **`手机端/index.html`** + **`手机端/assets/`** 为准即可。

---

## 一、参考与资源路径

| 类型 | 路径 | 说明 |
|------|------|------|
| **可交互原型** | `手机端/index.html` | 所有页面、动效、交互的参考实现，请本地打开逐屏对照 |
| **图标** | `手机端/assets/icons/` | SVG，可导入 Asset Catalog 或转为 SF Symbol 使用；详见 `手机端/assets/icons/README.md` |
| **背景/圆角** | `手机端/assets/drawables/` | 规格与色值同 Android；iOS 用 UIColor + 圆角/阴影代码或 Asset 实现，见 `手机端/assets/drawables/README.md`、`docs/手机端/背景与圆角规格.md` |
| **欢迎页** | `手机端/assets/welcome/` | 欢迎页 Logo、插图、扫码按钮 SVG |

**详细规格**（需查某模块精确数值时）：`docs/手机端/UI视觉规格文档.md`、`docs/手机端/背景与圆角规格.md`。

---

## 二、页面与路由

### 2.1 根结构

与 Android 一致：

```
App
├── 欢迎页（未绑定相框时全屏）
├── 主容器（max-width ≈ 448pt 居中）
│   ├── 顶部导航栏
│   ├── 主内容区（Tab 切换）
│   │   ├── Tab1 照片页
│   │   ├── Tab2 相框页
│   │   ├── Tab3 商城（占位）
│   │   └── Tab4 我的（占位）
│   ├── 任务指示条（Tab 栏上方）
│   └── 底部 Tab 栏（4 Tab）
└── 全局浮层（查看器、相册、设备详情、相框选择器、删除弹窗、任务面板、Alert、Toast/Banner）
```

### 2.2 入口与层级

与 Android 复刻指南「二、页面与路由」一致；层级关系同（导航 10 → Tab 20 → 各 Modal/Sheet/Alert 递增）。  
唯一差异：iOS 使用 **Safe Area**（`safeAreaInsets`）处理刘海与 Home Indicator，底部 Tab 与选择栏需预留 `safeAreaInsets.bottom`。

---

## 三、Design Token（iOS 实现）

### 3.1 颜色

与 Android 色值一致，在 iOS 中建议定义为 `UIColor` 扩展或 Asset Catalog：

- **品牌/功能**：`#007AFF`、`#0A84FF`、`#5AC8FA`、`#3B82F6`、`#34C759`、`#FF9500`、`#FF3B30`。
- **文字**：`#000000`、`#1D1D1F`、`rgba(60,60,67,0.6)`、`#9CA3AF`。
- **背景**：白、`#F3F4F6`、`#F9FAFB`、`rgba(28,28,30,0.9)`、遮罩与 Toast 同前。

### 3.2 间距与圆角（pt）

- **间距**：2/4/6/8/10/12/16/20/24/32 pt（与 Android dp 数值一致）。
- **圆角**：1.5pt 进度条、6pt 分组选中、8pt 分组容器、12pt 按钮/选中图、14pt Alert、16pt 卡片/面板、20pt Banner、全圆角（如 9999 或 `CGFloat.greatestFiniteMagnitude`）胶囊/Toast/角标。

### 3.3 字体与字重

- **字体**：SF Pro（系统默认），中文可用系统简体中文。
- **字重**：Regular(400)、Medium(500)、Semibold(600)、Bold(700)、Heavy(800)、Black(900)。
- **字号**：与 Android 一致（20pt 大标题、18pt 导航、17pt 面板/Alert、16pt 子页、14pt 列表、13pt 指示条、12pt 描述/按钮、11pt 副标题/胶囊、10pt 分组、9pt Tab/角标、8pt 刻度）。

### 3.4 毛玻璃（Blur）

使用 `UIVisualEffectView` 与 `UIBlurEffect`：

- **导航/Tab 栏**：`.regular` 或 12pt 等效 blur，背景白 80%–90%。
- **底部选择栏**：等效 20pt blur，背景 `rgba(28,28,30,0.9)`（可用 `.dark` + 自定义 tint）。
- **任务面板 / Alert / Banner**：等效 40pt + 高饱和度，背景白 96%–97% 或深色；可用 `.systemUltraThinMaterial` 等并调透明度。

### 3.5 动效曲线

与 Android 一致，在 iOS 用 `CubicTimingParameters` 或 `UIView.animate(withDuration:delay:options:)` 的 curve：

- 标准：(0.4, 0, 0.2, 1)；选中缩放：(0.25, 0.46, 0.45, 0.94)；圆圈弹出：(0.175, 0.885, 0.32, 1.275)；Banner：(0.34, 1.56, 0.64, 1)；Alert：(0.23, 1, 0.32, 1)。

---

## 四、资源与切图

- **图标**：`手机端/assets/icons/` 下 SVG，可按尺寸导入 Asset Catalog（Single Scale 或 @1x），或映射为 SF Symbol 使用；颜色用 tint 或 `currentColor` 语义。
- **欢迎页**：`手机端/assets/welcome/` 下三份 SVG，可直接用于 `UIImageView` 或转 PDF 进 Asset。
- **背景/圆角**：无现成 iOS 资源包，按 `手机端/assets/drawables/README.md` 与 `docs/手机端/背景与圆角规格.md` 用 `UIColor` + 圆角/阴影/渐变代码实现；渐变 135° 可用 `CAGradientLayer`。

---

## 五、按模块的 UI 规格摘要

与 Android 复刻指南「五、按模块的 UI 规格摘要」**数值一致**，单位改为 **pt**：

- 底部 Tab 栏：4 Tab，图标 22×22pt，标签 9pt Bold，选中 #2563EB、未选 #9CA3AF，白 80% + blur，内边距 16/8/24 pt，**下内边距含 safeAreaInsets.bottom**，顶边 1pt #F3F4F6。
- 顶部导航：18pt Bold 标题居中，扫码 22×22pt，白 90% + blur，底边 1pt #F9FAFB，**顶部注意 safeAreaInsets.top**。
- 照片网格：年 4 列/月日 3 列，间距 2pt，1:1 单元格，选中 scale 0.88、圆角 12pt、边框 3pt #3B82F6、蒙层 8% 蓝，圆圈 24×24pt offset 6pt，勾选热区 44×44pt。
- 分组标题栏、底部选择栏、相框选择器、相册/设备详情、图片查看器、删除弹窗、任务指示条、任务面板、Toast、结果 Alert、欢迎页：规格与 Android 一致，单位 pt，**所有底部栏需加 safeAreaInsets.bottom**。

更细数值见 `docs/手机端/UI视觉规格文档.md`。

---

## 六、交互要点（与 HTML 一致）

与 Android 复刻指南「六、交互要点」**完全一致**：

- **发送流程**：选照片 → 选相框 → 确认 → 发送中标记 → 任务指示条/面板 → 完成/部分失败/全部失败 → Banner/Alert → 8 秒清理。
- **重试**：从失败照片新建任务，标记发送中，立即发送。
- **多选与全选**：勾选后底部选择栏；相册「全选」为全选/取消全选可见照片；删除弹窗在相框内照片需「从全部相框删除」「仅从当前相框删除」。
- **相册/设备详情**：右滑全屏，设备删除两选项必现。
- **图片查看器**：翻页、删除、发送到相框，删除时同两选项。

详见 `docs/手机端/发送任务逻辑说明文档.md`。

---

## 七、自查清单

复刻完成后请逐项核对（与 Android 对齐，单位 pt，并强调 Safe Area）：

- **全局**：390pt 基准、**Safe Area 全场景**、SF Pro 与字重、主色/选中/错误、毛玻璃实现正确。
- **Tab 栏**：4 Tab、图标 22×22pt、标签 9pt Bold、选中/未选色、blur、**底部 safeAreaInsets.bottom**、点击反馈。
- **照片页**：精选/人物/地点横向滚动；年/月/日切换器与网格；选中态与圆圈；勾选热区 44×44pt；分组栏胶囊。
- **相册详情**：右滑全屏、全选胶囊、9 宫格与选择栏一致。
- **相框页与设备详情**：设备卡片、详情右滑、订阅/人物/全部照片、删除两选项。
- **图片查看器**：全屏黑底、返回/计数、删除/发送、翻页按钮。
- **相框选择器**：底部 Sheet、列表项与选中圈、确认按钮禁用态与文案。
- **删除弹窗**：底部弹出、多按钮、危险项；**底部避免与 Home Indicator 重叠**。
- **任务与通知**：指示条、任务面板、Toast、结果 Alert；**Banner/Alert 考虑 safeAreaInsets.top**。
- **欢迎页**：全屏、Logo/标题/描述/插图/主按钮/跳过，资源来自 `手机端/assets/welcome`。
- **动效**：选中缩放与圆圈弹出、面板/Sheet 滑入、按钮反馈、骨架屏（若用）。

---

## 八、参考文档（需细查时）

| 文档 | 用途 |
|------|------|
| `docs/手机端/UI视觉规格文档.md` | 各模块精确数值 |
| `docs/手机端/UI设计规范.md` | 完整 Design Token 与组件规范 |
| `docs/手机端/背景与圆角规格.md` | 背景/圆角/边框/阴影完整清单 |
| `docs/手机端/发送任务逻辑说明文档.md` | 发送任务状态与四层反馈 UI |
| `docs/手机端/照片权限交互说明文档.md` | 权限与空态交互 |

**复刻流程建议**：用本文档 + `手机端/index.html` + `手机端/assets/` 搭页面与 Token → 按「五、按模块的 UI 规格」与《UI视觉规格文档》对细节 → 按「六、交互要点」实现逻辑 → 用「七、自查清单」收尾核对。
