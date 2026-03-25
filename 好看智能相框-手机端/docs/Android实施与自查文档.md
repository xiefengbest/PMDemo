# 好看智能相框 App — Android 实施与自查文档

> 面向 Android 原生开发：按页面结构整理的切图说明、UI 规格与自查清单，便于复刻时逐项核对。  
> 设计基准：逻辑宽度 390dp（与 iPhone 14 Pro 一致），请按密度等比适配。

**关联文档**：本目录（`docs/`）下 `UI设计规范.md`、`UI视觉规格文档.md` 为完整 Design System 与逐条视觉规格，本文档为其「按页面 + 切图 + 自查」的 Android 实施摘要。

---

## 一、页面结构（信息架构）

### 1.1 根结构

```
App
├── 欢迎/绑定引导页（未绑定相框时全屏）
├── 主容器（max-width 约 448dp 居中，带阴影）
│   ├── 顶部导航栏（Header）
│   ├── 主内容区（Tab 切换）
│   │   ├── Tab1：照片页（localView / cloudView）
│   │   ├── Tab2：相框页（deviceView）
│   │   ├── Tab3：商城页（占位）
│   │   └── Tab4：我的页（占位）
│   ├── 任务指示条（Tab 栏上方，有任务时展开）
│   └── 底部 Tab 栏（4 个 Tab）
└── 全局浮层
    ├── 图片查看器（imageViewer）
    ├── 相册详情（albumModal，右滑全屏）
    ├── 设备详情（deviceModal，右滑全屏）
    ├── 相框选择器（framePickerSheet，底部弹出）
    ├── 删除确认弹窗（deleteDialogSheet，底部弹出）
    ├── 任务面板（taskPanel，底部 Sheet）
    ├── iOS 风格结果 Alert（iosAlertBackdrop）
    ├── 重命名弹窗、解绑弹窗等
    └── Toast / Banner 通知
```

### 1.2 主要页面与入口

| 页面/模块 | 入口 | 说明 |
|-----------|------|------|
| 照片页 | Tab「照片」 | 精选 + 人物 + 地点 + 分组照片网格 |
| 相册详情 | 点击精选/人物/地点卡片 | 右滑全屏，头部「全选」+ 9 宫格照片 |
| 相框页 | Tab「相框」 | 设备列表卡片 |
| 设备详情 | 点击设备卡片 | 右滑全屏：订阅壁纸、人物、全部照片 |
| 图片查看器 | 点击任意照片 | 全屏黑底，顶部返回/计数，底部删除/发送 |
| 相框选择器 | 选择栏/相册/查看器「发送到相框」 | 底部 Sheet，列表选相框 + 确认发送 |
| 底部选择栏 | 照片页/相册/设备详情中勾选照片 | 取消 + 删除 + 发送到相框（N） |
| 删除弹窗 | 选择栏删除 / 查看器删除 | 底部弹出，多按钮（如从全部相框删除 / 仅当前相框） |
| 任务指示条 | 发送任务进行中 | Tab 栏上方一条，可点击展开任务面板 |
| 任务面板 | 点击任务指示条 | 底部 Sheet，任务列表 + 进度 |
| 欢迎页 | 未绑定相框时 | 全屏：图标 + 标题 + 描述 + 插图 + 扫码绑定 + 跳过 |

### 1.3 页面层级（z-index 参考）

| 层级 | 用途 |
|------|------|
| 10 | 顶部导航栏 sticky |
| 15–16 | 时间滚动条、气泡 |
| 20 | 底部 Tab 栏 |
| 40 | 设备详情 Modal |
| 45 | 相册详情 Modal |
| 55 | 图片查看器 |
| 58–59 | 相框选择器遮罩 + Sheet |
| 60 | 删除弹窗 |
| 200–201 | 任务面板遮罩 + 面板 |
| 210 | iOS Alert |
| 260+ | 后台模拟、Banner/Toast |

---

## 二、切图与资源清单

### 2.1 图标类（已导出至 `assets/icons/`，建议转 Vector Drawable）

图标已从原型导出为独立 SVG，见 **`assets/icons/`** 目录及该目录下 `README.md`。以下为速查表。

| 名称 | 尺寸(dp) | 用途 | 说明 |
|------|----------|------|------|
| 扫码 | 22×22 | 顶部导航右侧 | stroke 2.2，蓝色 #2563EB |
| 返回(chevron-left) | 24×24 | 相册/设备详情/查看器 | stroke 3，蓝/白 |
| 发送(纸飞机) | 12×12 / 18×18 | 胶囊按钮/选择栏/查看器 | stroke 2.5 |
| 删除(垃圾桶) | 18×18 | 选择栏删除按钮 | 红色 #F87171 |
| 关闭(×) | 18×18 | 选择栏取消、面板关闭 | 白色 70% |
| 勾选(check) | 14×14 | 相框选择器选中圆内 | 白色 stroke 3 |
| 设置(齿轮) | 16×16 | 权限空态「前往设置」 | 当前色 |
| 相册(图片) | 36×36 | 权限空态插图 | 灰色 stroke 1.5 |
| 警告(感叹号) | 16×16 | 受限访问 Banner | #FF9500 |
| 添加(+) | 14×14 | 添加设备 | 蓝色 |
| Tab 图标×4 | 22×22 | 照片/相框/商城/我的 | 选中 #2563EB，未选 #9CA3AF，stroke 2.2 |
| 翻页(chevron) | 20×20 | 查看器左右 | 白色，圆形背景 |
| 更多(三点) | 20×20 | 相册人物更多 | gray-500 |

**说明**：所有线性图标统一风格（圆角线帽、统一 stroke），SVG 使用 `currentColor` 便于 Android tint。详见 `assets/icons/README.md`。

### 2.2 欢迎页资源（已有 SVG）

| 文件 | 用途 |
|------|------|
| `assets/welcome/welcome-icon-frame.svg` | 欢迎页顶部相框 Logo（88×88 容器内 40×40 图标） |
| `assets/welcome/welcome-illustration.svg` | 相框+手机+连接线插图（约 200×160） |
| `assets/welcome/welcome-btn-scan.svg` | 主按钮「扫码绑定」内图标 20×20 |

### 2.3 背景与圆角（已导出至 `assets/drawables/`）

- **背景 / 圆角**：已导出为 Android 可直接使用的 drawable XML，见 **`assets/drawables/`** 目录及该目录下 `README.md`。
- **完整清单**：所有背景、圆角、边框、阴影的数值与使用场景见 **`docs/背景与圆角规格.md`**。

| 文件 | 用途 |
|------|------|
| `assets/welcome/welcome-icon-frame.svg` | 欢迎页顶部相框 Logo（88×88 容器内 40×40 图标） |
| `assets/welcome/welcome-illustration.svg` | 相框+手机+连接线插图（约 200×160） |
| `assets/welcome/welcome-btn-scan.svg` | 主按钮「扫码绑定」内图标 20×20 |

### 2.4 无需切图的 UI 元素（用代码绘制）

- 照片选中圆圈：24×24 圆，未选中半透明黑+blur，选中蓝色 #3B82F6 + 白边 + 阴影
- 进度条：高度 3dp，圆角 1.5dp，轨道/填充色见 Token
- 分组切换器：圆角 8dp 背景 gray-100，内按钮 6dp 圆角
- 角标「待发送」：全圆角胶囊，red-500 背景，白字 9sp Bold
- 骨架屏：矩形/圆角矩形，shimmer 渐变动画

### 2.5 图片与占位

- 照片网格：1:1 正方形，object-fit: cover，占位背景 #F3F4F6
- 设备封面：80×80，圆角 12dp
- 订阅/人物卡片：按规格文档尺寸，封面比例 4:3 等

---

## 三、Design Token 速查表（Android 可直接用）

以下数值可直接放入 `colors.xml`、`dimens.xml`、`integers` 或 Theme 中引用。

### 3.1 颜色（colors.xml）

```xml
<!-- 品牌与功能 -->
<color name="color_primary">#007AFF</color>
<color name="color_primary_dark">#0A84FF</color>
<color name="color_primary_light">#5AC8FA</color>
<color name="color_primary_bg">#14007AFF</color> <!-- 8% -->
<color name="color_success">#34C759</color>
<color name="color_warning">#FF9500</color>
<color name="color_error">#FF3B30</color>
<color name="color_select">#3B82F6</color>
<color name="color_select_bg">#143B82F6</color> <!-- 8% -->

<!-- 文字 -->
<color name="text_title">#000000</color>
<color name="text_primary">#1D1D1F</color>
<color name="text_secondary">#993C3C43</color> <!-- 60,60,67 0.6 -->
<color name="text_tertiary">#733C3C43</color>
<color name="text_placeholder">#663C3C43</color>
<color name="text_disabled">#9CA3AF</color>

<!-- 背景 -->
<color name="bg_primary">#FFFFFF</color>
<color name="bg_secondary">#F3F4F6</color>
<color name="bg_tertiary">#F9FAFB</color>
<color name="bg_bar_dark">#E61C1C1E</color> <!-- 28,28,30 0.9 -->
<color name="bg_overlay_light">#4D000000</color>
<color name="bg_overlay_medium">#66000000</color>
<color name="bg_toast">#D9000000</color>
<color name="bg_banner">#F21E1E22</color>

<!-- 边框与线 -->
<color name="border_light">#1F3C3C43</color>
<color name="border_card">#F3F4F6</color>
<color name="border_indicator">#1F007AFF</color>

<!-- 组件 -->
<color name="blue_600">#2563EB</color>
<color name="blue_50">#EFF6FF</color>
<color name="gray_50">#F9FAFB</color>
<color name="gray_100">#F3F4F6</color>
<color name="gray_300">#D1D5DB</color>
<color name="gray_400">#9CA3AF</color>
<color name="gray_500">#6B7280</color>
<color name="gray_800">#1F2937</color>
<color name="gray_900">#111827</color>
<color name="red_400">#F87171</color>
<color name="red_50">#FEF2F2</color>
<color name="red_500">#EF4444</color>
```

### 3.2 间距与圆角（dimens.xml）

```xml
<dimen name="space_1">2dp</dimen>
<dimen name="space_2">4dp</dimen>
<dimen name="space_3">6dp</dimen>
<dimen name="space_4">8dp</dimen>
<dimen name="space_5">10dp</dimen>
<dimen name="space_6">12dp</dimen>
<dimen name="space_8">16dp</dimen>
<dimen name="space_10">20dp</dimen>
<dimen name="space_12">24dp</dimen>
<dimen name="space_16">32dp</dimen>

<dimen name="radius_xs">1.5dp</dimen>
<dimen name="radius_sm">6dp</dimen>
<dimen name="radius_md">8dp</dimen>
<dimen name="radius_lg">9dp</dimen>
<dimen name="radius_xl">10dp</dimen>
<dimen name="radius_2xl">12dp</dimen>
<dimen name="radius_3xl">14dp</dimen>
<dimen name="radius_4xl">16dp</dimen>
<dimen name="radius_5xl">20dp</dimen>
```

### 3.3 字号与字重（styles / typography）

| 用途 | 字号(sp) | 字重 |
|------|----------|------|
| 页面大标题 | 20 | Bold |
| 页面/导航标题 | 18 | Bold |
| Alert/面板标题 | 17 | SemiBold |
| 子页标题 | 16 | Bold |
| 列表主标题/正文 | 14 | Medium / Bold |
| Banner/指示条 | 13 | Medium |
| 描述/按钮/百分比 | 12 | Regular / SemiBold |
| 副标题/信息行/胶囊 | 11 | Medium / Bold |
| 分组/设备人物 | 10 | Bold |
| Tab/角标/张数 | 9 | Bold |
| 刻度/标签 | 8 | Bold |

### 3.4 毛玻璃（Blur）

| 场景 | blur(px) | 背景 |
|------|----------|------|
| 导航/Tab 栏 | 12 | 白 80%–90% |
| 底部选择栏 | 20 | rgba(28,28,30,0.9) |
| 任务面板/Alert | 40 + saturate(180%) | 白 96%–97% |
| Banner | 40 + saturate(180%) | color_banner |
| 未选中圆圈 | 4 | — |

### 3.5 动画曲线（可转为 Interpolator）

| 名称 | cubic-bezier | 用途 |
|------|----------------|------|
| ease_standard | (0.4, 0, 0.2, 1) | Tab、面板、指示条 |
| ease_decelerate | (0.25, 0.46, 0.45, 0.94) | 照片选中缩放 |
| ease_spring | (0.175, 0.885, 0.32, 1.275) | 选中圆圈弹出 |
| ease_bounce | (0.34, 1.56, 0.64, 1) | Banner/通知弹出 |
| ease_alert | (0.23, 1, 0.32, 1) | Alert 弹出 |

---

## 四、按页面/组件的 UI 实施要点

### 4.1 底部 Tab 栏

- 背景：白 80% + blur 12dp，顶边 1dp #F3F4F6
- 内边距：水平 16dp，上 8dp，下 24dp（含 safe-area-inset-bottom）
- 4 个 Tab 等分，单 Tab 宽约 64dp；图标 22×22，标签 9sp Bold
- 选中 #2563EB，未选 #9CA3AF；点击反馈 opacity 0.7

### 4.2 顶部导航栏

- 背景：白 90% + blur；底边 1dp #F9FAFB；sticky，z 高于内容
- 标题 18sp Bold 居中；右侧扫码 22×22 蓝色

### 4.3 照片网格（照片页/相册/设备详情）

- 列数：年 4 列，月/日 3 列；间距 2dp；单元格 1:1
- 占位背景 #F3F4F6；图片 cover
- 选中态：图片 scale(0.88)，圆角 12dp，边框 3dp #3B82F6，蒙层 8% 蓝
- 选中圆圈：24×24，右上角 offset 6dp；未选半透明黑+blur 4dp，选中蓝+白边+阴影；序号 11sp 800 白色
- 点击热区：右上 44×44dp，z 高于圆圈

### 4.4 分组标题栏（年/月/日分组上）

- 左侧标题 16sp Black #1F2937，计数 11sp #9CA3AF，间距 8dp
- 右侧「发送至相框」：10sp Bold 蓝字，背景 blue_50，内边距 10dp/4dp，全圆角
- 右侧「全选」：10sp Bold gray_500，背景 gray_100，内边距 10dp/4dp，全圆角
- Sticky：距顶约 40dp，背景白，上下 6dp 内边距

### 4.5 底部选择栏（三按钮）

- 外层 padding 12dp；容器圆角 16dp，内边距 10dp；背景 rgba(28,28,30,0.9) + blur 20dp；边框 1dp 白 10%
- 取消：固定 44×44，背景白 10%，关闭图标白 70%
- 删除：flex 1，背景白 5%，图标 red_400，文字 12sp 白
- 发送到相框：flex 2.5，背景 #2563EB，文字 12sp Black 白，阴影 shadow-lg blue-900/40
- 按钮高 44dp，圆角 12dp，间距 8dp；点击 scale(0.95)

### 4.6 相框选择器（Bottom Sheet）

- 遮罩 rgba(0,0,0,0.4)；面板白底，顶圆角 16dp，阴影
- 标题区：水平 20dp，上 20dp 下 12dp；标题 16sp Bold；关闭 20×20 gray_400
- 列表区：水平 20dp，最大高度 256dp，可滚动
- 确认按钮：全宽，高 44dp，圆角 12dp，背景 #2563EB，文字 14sp Bold 白，阴影 blue-600/30
- 列表项：左 48×48 封面 12dp 圆角，中 14sp Bold 名称 + 11sp gray 信息，右 24×24 选中圈（未选 2dp gray_300 边框，选中蓝底+白勾）

### 4.7 相册详情页（右滑全屏）

- 进入动画：translateX(100%) → 0，0.3s；背景白；z 高于 Tab
- 头部：白 90% + blur；返回 24×24 蓝；标题 18sp Bold 单行截断；副标题 11sp gray_400
- 头部右侧「全选」：10sp Bold gray_500，背景 gray_100，内边距 10dp/4dp，全圆角（无图标）
- 内容区：3 列网格，2dp 间距，与照片网格单元格一致

### 4.8 设备详情页

- 同相册为右滑全屏；头部含「解绑」11sp Bold red_500 背景 red_50
- 信息栏 11sp gray_400；订阅/人物横向滚动；全部照片下为分组切换器 + 照片网格
- 删除逻辑：弹窗需提供「从全部相框删除」「仅从当前相框删除」两选项

### 4.9 图片查看器

- 全屏黑底；顶部渐变+返回+计数 12sp Bold 白 70%
- 底部渐变+删除（flex 1）+ 发送（flex 2）；按钮高 48dp，圆角 16dp；删除白 10%+blur，发送 #2563EB+阴影
- 左右翻页：40×40 圆，白 10%+blur，图标 20×20 白

### 4.10 删除弹窗（底部弹出）

- 遮罩 40% 黑；卡片 max-width 约 448dp，底边距 8dp，圆角 16dp
- 标题 16sp Bold，描述 12sp gray_500；操作按钮纵向排列，14sp，危险项 red_500/red_50，分隔线 gray_100

### 4.11 任务指示条

- 位置：Tab 栏正上方；收起 max-height 0，展开约 64dp，0.35s ease_standard
- 背景 rgba(0,122,255,0.06)，顶边 0.5dp rgba(0,122,255,0.12)
- 内容：发送图标 14×14 蓝 + 文案 13sp Medium 蓝 + 进度条(flex 1) + 百分比 12sp SemiBold 蓝 + 箭头 14×14
- 进度条高 3dp，轨道 rgba(120,120,128,0.12)，填充 #007AFF（有失败时 #FF9500）

### 4.12 任务面板（Bottom Sheet）

- 遮罩 30% 黑；面板白 97% + blur 40dp saturate 180%，顶圆角 16dp，max-height 60vh
- 头部：16dp/20dp/12dp，底边 0.5dp；标题 17sp SemiBold；关闭 20×20
- 任务行：12dp 上下 20dp 左右；左侧 34×34 圆角 9dp 渐变图标；标题 14sp Medium；副标题 11sp；进度条+重试按钮

### 4.13 Toast

- 位置：顶 40dp 水平居中；背景 rgba(0,0,0,0.85)；内边距 10dp/20dp；全圆角；12sp Medium 白

### 4.14 iOS 风格结果 Alert

- 宽度 270dp；圆角 14dp；背景白 96% + blur 40dp
- Body：上 20dp 左右 16dp 下 16dp；标题 17sp SemiBold；描述 13sp secondary
- 按钮高 44dp，17sp #007AFF，分隔线 0.5dp；弹出 scale 1.08→1, opacity 0→1, 0.25s

### 4.15 欢迎页

- 全屏白底；图标区 88×88 圆角 22dp 渐变 #007AFF→#5AC8FA
- 标题 24sp 800；描述 15sp secondary，max-width 280dp
- 插图区 200×160；主按钮「扫码绑定」高 50dp 圆角 14dp 蓝底 17sp 600；次要「跳过」15sp 500 secondary

---

## 五、Android 自查清单

请按模块逐项核对，在复刻完成后自查。

### 5.1 全局与基础

- [ ] 设计基准 390dp 逻辑宽度，密度适配正确
- [ ] 底部安全区（NavigationBar/手势区）已预留
- [ ] 字体：Noto Sans SC 或系统默认中文字体，字重与规范一致
- [ ] 主色 #007AFF、选中蓝 #3B82F6、错误 #FF3B30 等 Token 全局统一
- [ ] 毛玻璃（Blur）在导航栏、Tab 栏、选择栏、面板、Alert 处正确实现

### 5.2 底部 Tab 栏

- [ ] 4 个 Tab：照片、相框、商城、我的；图标 22×22，标签 9sp Bold
- [ ] 选中 #2563EB，未选 #9CA3AF；背景白 80% + blur
- [ ] 内边距与安全区正确；点击反馈明显

### 5.3 照片页

- [ ] 精选/人物/地点横向滚动，隐藏滚动条；卡片尺寸与间距符合规格
- [ ] 分组切换器（年/月/日）gray_100 背景，选中白底 shadow
- [ ] 照片网格 3/4 列、2dp 间距、1:1 单元格；占位色 #F3F4F6
- [ ] 选中态：缩放 0.88、圆角 12dp、3dp 蓝框、蒙层、右上 24×24 圆圈（未选/选中样式）
- [ ] 选中圆圈弹出动画 0.3s 弹性；勾选热区 44×44
- [ ] 分组栏「发送至相框」「全选」胶囊样式与字号正确

### 5.4 相册详情

- [ ] 右滑进入全屏；头部返回+标题+副标题+「全选」胶囊（无图标）
- [ ] 9 宫格与照片页单元格一致；全选/取消全选逻辑正确
- [ ] 底部选择栏与照片页共用规格（取消+删除+发送到相框 N）

### 5.5 相框页与设备详情

- [ ] 设备卡片圆角 16dp、边框、阴影；封面 80×80 圆角 12dp
- [ ] 设备详情右滑全屏；订阅/人物横向列表；全部照片分组+网格
- [ ] 删除照片弹窗具备「从全部相框删除」「仅从当前相框删除」两选项

### 5.6 图片查看器

- [ ] 全屏黑底；顶部返回+计数；底部删除+发送，比例与样式正确
- [ ] 左右翻页按钮 40×40 圆；滑动/点击翻页流畅

### 5.7 相框选择器

- [ ] 底部 Sheet 弹出；列表项左图+中名称/信息+右选中圈
- [ ] 确认按钮禁用态（未选相框）50% 透明度；文案「发送 N 张到{相框名}」

### 5.8 删除弹窗

- [ ] 底部弹出；标题+描述+多按钮；危险项红色；分隔线正确

### 5.9 任务与通知

- [ ] 任务指示条在 Tab 上方，可展开/收起；进度条与百分比、失败态橙色
- [ ] 任务面板 Sheet：毛玻璃、任务行布局、重试按钮
- [ ] Toast 位置与样式；Banner 通知（若有）毛玻璃与圆角
- [ ] 发送结果 iOS 风格 Alert：宽度 270dp、圆角 14dp、按钮 44dp

### 5.10 欢迎页

- [ ] 未绑定相框时全屏展示；图标+标题+描述+插图+主按钮+跳过
- 主按钮与插图资源与设计一致（可用 `assets/welcome` 下 SVG 转 Vector）

### 5.11 动效与细节

- [ ] 照片选中缩放 0.35s ease_decelerate；圆圈弹出 0.3s ease_spring
- [ ] 面板/Sheet 滑入 0.3–0.35s；Banner/Alert 使用规定曲线
- [ ] 按钮点击 scale 或 opacity 反馈一致
- [ ] 骨架屏 shimmer 动画（若使用）

---

## 六、参考文件一览

| 文件 | 说明 |
|------|------|
| `../index.html` | 可交互 HTML 原型，所有页面与交互的参考实现 |
| `UI设计规范.md` | Design Token、组件规范、使用指南 |
| `UI视觉规格文档.md` | 按模块的精确数值（字号、间距、颜色、动效） |
| `Android实施与自查文档.md` | 本文档：页面结构、切图、Token 表、自查清单 |
| `背景与圆角规格.md` | 背景/圆角/边框/阴影完整清单，便于查漏 |
| `../assets/icons/README.md` | 图标尺寸、用途、颜色说明 |
| `../assets/drawables/` | 背景与圆角 drawable XML（可复制到 res/drawable） |
| `../assets/drawables/README.md` | Drawable 清单、圆角系统、blur/阴影说明 |
| `../assets/welcome/*.svg` | 欢迎页图标与插图 |

**建议流程**：先按本文档完成页面结构与 Token 接入 → 用《UI视觉规格文档》逐模块对尺寸与样式 → 用本文档「Android 自查清单」做最终核对。
