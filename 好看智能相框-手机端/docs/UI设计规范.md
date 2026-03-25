# 好看智能相框 — UI 设计规范（Design System）

> 本规范适用于 iOS、Android 原生开发及 HTML 原型设计。  
> 所有新增页面和组件**必须**引用本规范中的设计令牌，不允许自行定义颜色、字号等硬编码值。

**设计基准**：iPhone 14 Pro（逻辑宽度 390pt），Android 按密度等比适配。

---

# 第一部分：设计令牌（Design Tokens）

设计令牌是整套规范的原子单元，所有组件样式都从这里引用。

---

## 1. 颜色（Color）

### 1.1 品牌色与功能色

| Token 名 | 色值 | 用途 |
|---|---|---|
| `color-primary` | `#007AFF` | 主操作按钮、链接、指示条、可点击文字 |
| `color-primary-dark` | `#0A84FF` | 暗色背景上的主色（通知栏、后台模拟） |
| `color-primary-light` | `#5AC8FA` | 渐变终止色 |
| `color-primary-bg` | `rgba(0,122,255,0.08)` | 主色浅背景（如胶囊按钮、指示条底色） |
| `color-primary-bg-pressed` | `rgba(0,122,255,0.15)` | 主色按下态背景 |
| `color-success` | `#34C759` | 成功状态 |
| `color-success-end` | `#30D158` | 成功渐变终止色 |
| `color-warning` | `#FF9500` | 警告/部分失败 |
| `color-warning-end` | `#FF9F0A` | 警告渐变终止色 |
| `color-error` | `#FF3B30` | 错误/全部失败 |
| `color-error-end` | `#FF453A` | 错误渐变终止色 |
| `color-select` | `#3B82F6` | 选中态（照片选中边框、圆圈） |
| `color-select-bg` | `rgba(59,130,246,0.08)` | 选中态蒙层 |

### 1.2 渐变（统一 135° 方向）

| Token 名 | 值 | 用途 |
|---|---|---|
| `gradient-primary` | `#007AFF → #5AC8FA` | 发送中图标 |
| `gradient-success` | `#34C759 → #30D158` | 成功图标 |
| `gradient-warning` | `#FF9500 → #FF9F0A` | 部分失败图标 |
| `gradient-error` | `#FF3B30 → #FF453A` | 全部失败图标 |

### 1.3 文字色

| Token 名 | 色值 | 用途 |
|---|---|---|
| `color-text-title` | `#000000` | 一级标题 |
| `color-text-primary` | `#1D1D1F` | 正文 |
| `color-text-secondary` | `rgba(60,60,67,0.6)` | 弹窗描述、二级信息 |
| `color-text-tertiary` | `rgba(60,60,67,0.45)` | 三级信息 |
| `color-text-placeholder` | `rgba(60,60,67,0.4)` | 占位文字、关闭按钮 |
| `color-text-disabled` | `#9CA3AF` | 不可用/未选中 Tab |
| `color-text-on-dark` | `#FFFFFF` | 深色背景上的文字 |
| `color-text-on-dark-60` | `rgba(255,255,255,0.6)` | 深色背景上的二级文字 |
| `color-text-on-dark-30` | `rgba(255,255,255,0.3)` | 深色背景上的三级文字 |

### 1.4 背景色

| Token 名 | 色值 | 用途 |
|---|---|---|
| `color-bg-primary` | `#FFFFFF` | 页面背景 |
| `color-bg-secondary` | `#F3F4F6` (gray-100) | 分组切换器背景、占位图 |
| `color-bg-tertiary` | `#F9FAFB` (gray-50) | 极浅背景、分隔线 |
| `color-bg-skeleton` | `#F0F0F0 / #E0E0E0` | 骨架屏 |
| `color-bg-overlay-light` | `rgba(0,0,0,0.3)` | 弹窗遮罩 |
| `color-bg-overlay-medium` | `rgba(0,0,0,0.4)` | Action Sheet 遮罩 |
| `color-bg-overlay-heavy` | `rgba(0,0,0,0.88)` | 后台模拟遮罩 |
| `color-bg-toast` | `rgba(0,0,0,0.85)` | Toast 背景 |
| `color-bg-banner` | `rgba(30,30,34,0.95)` | Banner 通知背景 |
| `color-bg-notif` | `rgba(50,50,54,0.95)` | 后台通知栏背景 |
| `color-bg-bar-dark` | `rgba(28,28,30,0.9)` | 底部选择栏背景 |

### 1.5 边框与分隔线

| Token 名 | 色值 | 用途 |
|---|---|---|
| `color-border-light` | `rgba(60,60,67,0.12)` | 标准分隔线（0.5px） |
| `color-border-divider` | `rgba(60,60,67,0.18)` | Alert 按钮分隔线 |
| `color-border-card` | `#F3F4F6` (gray-100) | 卡片边框 |
| `color-border-indicator` | `rgba(0,122,255,0.12)` | 任务指示条顶部线 |

---

## 2. 字体排版（Typography）

### 2.1 字体族

| 平台 | 字体 |
|---|---|
| iOS | SF Pro（系统默认） |
| Android | Noto Sans SC / 系统默认 |

### 2.2 字号阶梯（Font Scale）

| Token 名 | 大小 | 字重 | 典型用途 |
|---|---|---|---|
| `font-headline-xl` | 20px | Bold | 页面大标题（"精选"、"我的相框"） |
| `font-headline-lg` | 18px | Bold | 页面标题（导航栏、区域标题） |
| `font-headline-md` | 17px | SemiBold | Alert 标题、面板标题 |
| `font-headline-sm` | 16px | Bold | 子页面标题（相框选择器） |
| `font-body-lg` | 14px | Medium/Bold | 列表项主标题、正文 |
| `font-body-md` | 13px | Medium/SemiBold | Banner 标题、指示条文案 |
| `font-body-sm` | 12px | Regular/SemiBold | Banner 描述、百分比、按钮文字 |
| `font-caption-lg` | 11px | Medium/Bold | 副标题、信息行、胶囊按钮 |
| `font-caption-md` | 10px | Bold/Medium | 分组操作按钮、设备人物姓名 |
| `font-caption-sm` | 9px | Bold | Tab 标签、角标文字、张数 |
| `font-caption-xs` | 8px | Bold | 订阅状态标签、时间刻度 |

### 2.3 字重映射

| 语义 | CSS weight | iOS | Android |
|---|---|---|---|
| Regular | 400 | Regular | Regular |
| Medium | 500 | Medium | Medium |
| SemiBold | 600 | Semibold | SemiBold |
| Bold | 700 | Bold | Bold |
| Heavy | 800 | Heavy | ExtraBold |
| Black | 900 | Black | Black |

### 2.4 特殊排版

| 属性 | 值 | 用途 |
|---|---|---|
| `font-variant-numeric: tabular-nums` | 等宽数字 | 百分比显示 |
| `letter-spacing: -0.2px` | 微紧 | Alert 标题 |
| `letter-spacing: -0.1px` | 微紧 | Banner 标题、任务行标题 |

---

## 3. 间距系统（Spacing）

基于 4px 为基础单元的间距阶梯：

| Token 名 | 值 | 典型用途 |
|---|---|---|
| `space-1` | 2px | 网格间距、照片预览条间距 |
| `space-2` | 4px | 微间距（图标与标签、行间距补偿） |
| `space-3` | 6px | 角标 offset、选中圆圈 offset |
| `space-4` | 8px | 按钮间距、地点卡片间距、Tab 上内边距 |
| `space-5` | 10px | 指示条元素间距、选择栏内边距 |
| `space-6` | 12px | 列表行上下内边距、人物/精选卡片间距、按钮间距 |
| `space-8` | 16px | 页面水平内边距（全局标准值）、面板头部上内边距 |
| `space-10` | 20px | 面板内容水平内边距、Alert body 上内边距 |
| `space-12` | 24px | Tab 栏底部内边距（含安全区）、底部按钮区 |
| `space-16` | 32px | 分组间距 |

**核心规则**：页面水平内边距统一使用 `space-8`（16px），面板内容水平内边距使用 `space-10`（20px）。

---

## 4. 圆角系统（Border Radius）

| Token 名 | 值 | 用途 |
|---|---|---|
| `radius-xs` | 1.5px | 进度条 |
| `radius-sm` | 6px | 分组切换器按钮 |
| `radius-md` | 8px | 分组切换器容器、骨架屏矩形 |
| `radius-lg` | 9px | 任务行图标 |
| `radius-xl` | 10px | Banner 应用图标 |
| `radius-2xl` | 12px | 标准按钮、照片选中圆角、订阅卡片 |
| `radius-3xl` | 14px | iOS Alert |
| `radius-4xl` | 16px | 卡片、面板顶部、选择栏、Action Sheet |
| `radius-5xl` | 20px | Banner 通知卡片 |
| `radius-full` | 9999px | 胶囊按钮、头像、Toast、角标 |

---

## 5. 阴影系统（Shadow）

| Token 名 | 值 | 用途 |
|---|---|---|
| `shadow-sm` | 系统 shadow-sm | 头像、设备卡片、订阅卡片 |
| `shadow-md` | 系统 shadow-md | 精选封面 |
| `shadow-lg` | 系统 shadow-lg | 主操作按钮 |
| `shadow-xl` | 系统 shadow-xl | Toast |
| `shadow-2xl` | 系统 shadow-2xl | 面板、Action Sheet、选择栏 |
| `shadow-button-primary` | `shadow-lg` + `rgba(30,64,175,0.4)` | 蓝色发送按钮 |
| `shadow-button-confirm` | `shadow-lg` + `rgba(37,99,235,0.3)` | 相框选择器确认按钮 |
| `shadow-panel` | `0 -4px 30px rgba(0,0,0,0.12)` | 任务面板 |
| `shadow-banner` | `0 8px 30px rgba(0,0,0,0.28)` | 前台 Banner |
| `shadow-notif` | `0 8px 30px rgba(0,0,0,0.35)` | 后台通知 |
| `shadow-alert` | `0 0 0 0.5px rgba(0,0,0,0.06), 0 8px 40px rgba(0,0,0,0.18)` | iOS Alert |
| `shadow-select` | `0 2px 8px rgba(59,130,246,0.4)` | 照片选中圆圈 |

---

## 6. 毛玻璃效果（Blur）

| Token 名 | blur | saturate | 典型背景色 | 用途 |
|---|---|---|---|---|
| `blur-nav` | 12px | — | 白色 80%–90% | 导航栏、Tab 栏 |
| `blur-bar` | 20px | — | `color-bg-bar-dark` | 底部选择栏 |
| `blur-panel` | 40px | 180% | 白色 96%–97% | 任务面板、iOS Alert |
| `blur-banner` | 40px | 180% | `color-bg-banner` | Banner 通知 |
| `blur-overlay` | 24px | — | `color-bg-overlay-heavy` | 后台模拟遮罩 |
| `blur-subtle` | 4px | — | — | 照片未选中圆圈 |
| `blur-bubble` | 10px | — | `rgba(0,0,0,0.78)` | 时间气泡 |

---

## 7. 动画系统（Motion）

### 7.1 缓动曲线（Easing）

| Token 名 | cubic-bezier | 性格 | 用途 |
|---|---|---|---|
| `ease-standard` | (0.4, 0, 0.2, 1) | 平滑标准 | Tab 切换、面板弹出、指示条展开 |
| `ease-decelerate` | (0.25, 0.46, 0.45, 0.94) | 减速 | 照片选中缩放 |
| `ease-spring` | (0.175, 0.885, 0.32, 1.275) | 弹性 | 选中圆圈弹出 |
| `ease-bounce` | (0.34, 1.56, 0.64, 1) | 弹性回弹 | Banner/Push 通知弹出 |
| `ease-alert` | (0.23, 1, 0.32, 1) | 快速弹出 | iOS Alert |
| `ease-default` | ease | 默认 | 进度条、通用过渡 |

### 7.2 时长（Duration）

| Token 名 | 值 | 用途 |
|---|---|---|
| `duration-fast` | 0.15s | 刻度点过渡、极快反馈 |
| `duration-normal` | 0.2s–0.3s | 按钮反馈、面板滑入、进度条、通用过渡 |
| `duration-slow` | 0.35s | 指示条展开、任务面板弹出 |
| `duration-banner` | 0.45s | Banner 弹出 |
| `duration-push` | 0.5s | Push 通知弹出 |

### 7.3 预定义动画

| 动画名 | 描述 | 参数 |
|---|---|---|
| `anim-pop` | 选中弹出 | scale 0.5→1, opacity 0→1, 0.3s `ease-spring` |
| `anim-pulse-ring` | 脉冲环 | scale 0.8→2.2, opacity 1→0, 循环 |
| `anim-shimmer` | 骨架屏闪烁 | background-position -200%→200%, 1.5s ease-in-out infinite |
| `anim-check-draw` | 勾选描边 | stroke-dashoffset 24→0, 0.4s delay 0.2s ease |
| `anim-alert-in` | Alert 弹出 | scale 1.08→1, opacity 0→1, 0.25s `ease-alert` |
| `anim-ai-enter` | AI 内容入场 | opacity 0→1, scale 0.92→1, 0.45s `ease-bounce` |

---

## 8. 图标系统（Iconography）

统一使用 Lucide 风格线性图标。

| Token 名 | 尺寸 | stroke-width | 用途 |
|---|---|---|---|
| `icon-xs` | 10×10 | 2.5 | 信息行辅助图标 |
| `icon-sm` | 12×12 | 2.5 | 胶囊按钮内图标 |
| `icon-md` | 14×14 | 2.5 | 指示条图标、任务行内图标 |
| `icon-lg` | 16×16 | 2–2.5 | Banner 内图标、面板按钮图标 |
| `icon-xl` | 18×18 | 2.2–2.5 | 选择栏按钮图标、箭头 |
| `icon-2xl` | 20×20 | 2.5 | 面板关闭按钮、翻页按钮 |
| `icon-3xl` | 22×22 | 2.2 | Tab 栏图标、导航栏按钮 |
| `icon-4xl` | 24×24 | 2.5–3 | 查看器返回/结果图标 |
| `icon-5xl` | 28×28 | 2 | 后台遮罩大图标 |

---

## 9. 尺寸令牌（Size）

### 可点击元素最小尺寸

| Token 名 | 值 | 说明 |
|---|---|---|
| `tap-target-min` | 44pt | Apple HIG 最小点击区域 |

### 固定高度

| Token 名 | 值 | 用途 |
|---|---|---|
| `height-button-lg` | 48px | 查看器底部按钮 |
| `height-button-md` | 44px | 标准按钮（Alert 按钮、确认按钮、选择栏按钮） |
| `height-tab-bar` | ~56px（含安全区更大） | 底部 Tab 栏 |

### 图标容器

| Token 名 | 尺寸 | 圆角 | 用途 |
|---|---|---|---|
| `icon-container-sm` | 34×34 | 9px | 任务行图标 |
| `icon-container-md` | 36×36 | 10px | Banner/通知应用图标 |
| `icon-container-lg` | 48×48 | 圆形 | Alert 结果图标 |
| `icon-container-xl` | 64×64 | 16px | 后台遮罩大图标 |

---

# 第二部分：通用组件规范（Components）

以下组件在整个 App 中复用，任何页面使用时必须遵循统一样式。

---

## C1. 主操作按钮（Primary Button）

出现场景：选择栏"发送到相框"、相框选择器"确认发送"、查看器"发送到相框"、相册"一键发送到相框"

### 大尺寸（选择栏/查看器/确认按钮）

| 属性 | Token / 值 |
|---|---|
| 背景 | `color-select` (#3B82F6) 或 `color-primary` (#007AFF) |
| 文字 | `color-text-on-dark`, Black 或 Bold |
| 高度 | `height-button-md` (44px) 或 `height-button-lg` (48px) |
| 圆角 | `radius-2xl` (12px) 或 `radius-4xl` (16px) |
| 阴影 | `shadow-button-primary` |
| 图标 | 发送图标, `icon-xl` (18×18), 间距 `space-4` (8px) |
| 点击反馈 | scale(0.95) 或 scale(0.98) |

### 小尺寸/胶囊（"一键发送到相框"/"发送至相框"/"添加设备"）

| 属性 | Token / 值 |
|---|---|
| 背景 | `color-select` (实心蓝) 或 `color-primary-bg` (浅蓝底) |
| 文字 | 白色（实心）或 `color-primary`（浅底） |
| 字号 | `font-caption-lg` (11px) 或 `font-caption-md` (10px), Bold |
| 内边距 | 水平 14px/10px, 垂直 6px/4px |
| 圆角 | `radius-full` |
| 图标 | `icon-sm` (12×12), 间距 `space-2` (4px) |
| 点击反馈 | opacity → 0.6 |

---

## C2. 次级按钮（Secondary Button）

出现场景：选择栏"删除"、查看器"删除"

| 属性 | Token / 值 |
|---|---|
| 背景 | `rgba(255,255,255,0.05)` 或 `rgba(255,255,255,0.1)` + blur |
| 文字 | 白色 |
| 图标颜色 | red-400 (`#F87171`) |
| 高度 | `height-button-md` (44px) 或 `height-button-lg` (48px) |
| 圆角 | `radius-2xl` (12px) 或 `radius-4xl` (16px) |
| 点击反馈 | scale(0.95) |

---

## C3. 文字按钮（Text Button）

出现场景："全选"、"重试"

| 变体 | 字号 | 字重 | 颜色 | 背景 |
|---|---|---|---|---|
| 胶囊灰 | 10px | Bold | gray-500 | gray-100, `radius-full` |
| 纯文字蓝 | 12px | SemiBold | `color-primary` | 无 |
| 危险文字 | 11px–14px | Bold | `color-error` | red-50 或无 |

---

## C4. 底部栏（Bottom Bar）

出现场景：选择栏（照片页、相册、设备详情通用）

| 属性 | Token / 值 |
|---|---|
| 外层 padding | `space-6` (12px) |
| 容器背景 | `color-bg-bar-dark` + `blur-bar` |
| 容器圆角 | `radius-4xl` (16px) |
| 容器内边距 | `space-5` (10px) |
| 容器边框 | 1px `rgba(255,255,255,0.1)` |
| 按钮间距 | `space-4` (8px) |
| 结构 | 取消(44×44) + 删除(flex:1) + 主操作(flex:2.5) |

**三个场景（照片页/相册/设备详情）复用同一套底部栏组件，仅主按钮文案不同。**

---

## C5. Bottom Sheet（底部弹出面板）

出现场景：任务面板、相框选择器

| 属性 | Token / 值 |
|---|---|
| 遮罩 | `color-bg-overlay-light` 或 `color-bg-overlay-medium` |
| 面板背景 | 白色 或 白色 97% + `blur-panel` |
| 顶部圆角 | `radius-4xl` (16px) |
| 弹出动画 | translateY(100%)→0, `duration-slow`, `ease-standard` |
| 阴影 | `shadow-2xl` 或 `shadow-panel` |
| 头部 padding | 上 `space-8`(16px), 左右 `space-10`(20px), 下 `space-6`(12px) |
| 头部底部线 | 0.5px `color-border-light` |

---

## C6. iOS Alert 弹窗

出现场景：发送结果弹窗

| 属性 | Token / 值 |
|---|---|
| 宽度 | 270px |
| 圆角 | `radius-3xl` (14px) |
| 背景 | 白色 96% + `blur-panel` |
| 阴影 | `shadow-alert` |
| 弹出动画 | `anim-alert-in` |
| body padding | 上 20px, 左右 16px, 下 16px |
| 标题 | `font-headline-md`, `color-text-title` |
| 描述 | 13px, `color-text-secondary` |
| 按钮高度 | `height-button-md` (44px) |
| 按钮文字 | 17px, `color-primary` |
| 按钮分隔线 | 0.5px `color-border-divider` |

---

## C7. Action Sheet（操作菜单）

出现场景：人物操作、删除确认、解绑确认

| 属性 | Token / 值 |
|---|---|
| 遮罩 | `color-bg-overlay-medium` |
| 主卡片背景 | 白色, `radius-4xl` |
| 菜单项 | padding-y 14px, 14px Medium, `color-primary` |
| 危险项 | `color-error` |
| 分隔线 | 1px `color-border-card` |
| 取消按钮 | 独立卡片, margin-top 8px, 14px Bold, `color-primary` |
| 弹出动画 | translateY(100%)→0, 0.3s |

---

## C8. Banner 通知（顶部横幅）

出现场景：发送开始、发送完成

| 属性 | Token / 值 |
|---|---|
| 弹出 | top -80px → 8px, `duration-banner`, `ease-bounce` |
| 宽度 | 100% - 16px, 最大 400px, 居中 |
| 卡片背景 | `color-bg-banner` + `blur-banner` |
| 卡片圆角 | `radius-5xl` (20px) |
| 卡片 padding | 14px 16px |
| 卡片阴影 | `shadow-banner` |
| 应用图标 | `icon-container-md` (36×36, `radius-xl`) |
| 标题 | `font-body-md` (13px SemiBold), 白色 |
| 描述 | `font-body-sm` (12px), `color-text-on-dark-60` |

---

## C9. Toast 提示

| 属性 | Token / 值 |
|---|---|
| 位置 | 顶部 40px, 居中 |
| 背景 | `color-bg-toast` |
| 圆角 | `radius-full` |
| 内边距 | 10px / 20px |
| 文字 | `font-body-sm` (12px Medium), 白色 |
| 阴影 | `shadow-xl` |
| 动画 | opacity 过渡 0.3s |

---

## C10. 卡片（Card）

出现场景：设备卡片

| 属性 | Token / 值 |
|---|---|
| 背景 | 白色 |
| 圆角 | `radius-4xl` (16px) |
| 边框 | 1px `color-border-card` |
| 阴影 | `shadow-sm` |
| 点击反馈 | scale(0.97), opacity 0.85, 0.2s ease |

---

## C11. 输入弹窗（Input Dialog）

出现场景：重命名弹窗

| 属性 | Token / 值 |
|---|---|
| 宽度 | 288px |
| 圆角 | `radius-4xl` (16px) |
| 标题 | `font-headline-sm` (16px Bold) |
| 输入框 | 边框 1px gray-200, `radius-2xl` (12px), 14px, 居中 |
| 焦点态 | 边框 `color-primary`, ring-2 blue-100 |
| 底部按钮 | 两栏等分, 分隔线 1px `color-border-card` |
| 取消 | 14px Medium, gray-500 |
| 确定 | 14px Bold, `color-primary` |

---

## C12. 进度条（Progress Bar）

出现场景：任务指示条、任务行、后台通知栏

| 属性 | Token / 值 |
|---|---|
| 高度 | 3px（前台）/ 2.5px（后台通知） |
| 轨道颜色 | `rgba(120,120,128,0.12)` 或 `rgba(255,255,255,0.12)` |
| 填充颜色 | `color-primary` / `color-warning`（有失败时） |
| 圆角 | `radius-xs` (1.5px) |
| 过渡 | width 0.3s `ease-default` |

---

## C13. 横向滚动区域（Horizontal Scroll Section）

出现场景：精选、人物、地点、订阅壁纸

| 属性 | Token / 值 |
|---|---|
| 滚动条 | 隐藏 |
| 水平内边距 | `space-8` (16px) |
| 项间距 | 8px–16px（按场景） |
| 区域标题 | `font-headline-xl`(20px) 或 `font-headline-lg`(18px) Bold |
| 加载提示 | `font-caption-lg`(11px), gray-400, 带旋转加载图标 |
| 入场动画 | `anim-ai-enter` |

---

## C14. 照片网格（Photo Grid）

| 属性 | Token / 值 |
|---|---|
| 列数 | 3（月/日模式）或 4（年模式） |
| 间距 | `space-1` (2px) |
| 单元格比例 | 1:1 正方形 |
| 占位背景 | `color-bg-secondary` |
| 图片填充 | cover |
| 选中缩放 | scale(0.88), `radius-2xl` |
| 选中边框 | 3px `color-select` |
| 选中圆圈 | 24×24, 右上角 offset `space-3` (6px) |
| 点击热区 | 44×44 右上角 |

---

# 第三部分：使用指南

## HTML 原型中的应用

在 HTML 中，可以将 Design Tokens 定义为 CSS 自定义属性：

```css
:root {
    --color-primary: #007AFF;
    --color-success: #34C759;
    --color-warning: #FF9500;
    --color-error: #FF3B30;
    --color-select: #3B82F6;
    --radius-2xl: 12px;
    --radius-4xl: 16px;
    --space-8: 16px;
    /* ... */
}
```

后续所有样式引用变量，不再写死数值。

## iOS 原生中的应用

在 Swift 中定义 Design Token 枚举或扩展：

```swift
extension UIColor {
    static let primary = UIColor(hex: "#007AFF")
    static let success = UIColor(hex: "#34C759")
    // ...
}

struct Spacing {
    static let s8: CGFloat = 16
    static let s10: CGFloat = 20
    // ...
}
```

## Android 原生中的应用

在 `values/tokens.xml` 中定义：

```xml
<color name="color_primary">#007AFF</color>
<dimen name="space_8">16dp</dimen>
<dimen name="radius_4xl">16dp</dimen>
```

---

## 规范维护原则

1. **新增颜色**必须先在 Token 表中注册，禁止在组件中使用未定义的色值
2. **新增字号**必须归入字号阶梯，不允许出现阶梯外的大小
3. **新增间距**应尽量使用已有阶梯值，确需新增须补充到 Token 表
4. **新增组件**须标注引用了哪些 Token，并评估能否复用已有通用组件
5. **修改 Token 值**须同步更新所有引用该 Token 的组件和代码
