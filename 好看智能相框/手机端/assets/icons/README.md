# 图标切图（Icons）

本目录包含从 HTML 原型导出的线性图标，统一为 **viewBox="0 0 24 24"**，便于 Android/iOS 按设计稿尺寸缩放使用。

## 使用说明

- **描色**：除特殊说明外，均使用 `stroke="currentColor"`，便于在原生侧通过 tint 切换颜色。
- **Android**：可将 SVG 转为 Vector Drawable，或使用 Android Studio 的 Vector Asset 导入。
- **尺寸**：设计稿中常见尺寸见下表，导出时以 24×24 为基准，在布局中指定宽高即可（如 22dp、18dp）。

## 文件清单

| 文件名 | 用途 | 设计稿常见尺寸 |
|--------|------|----------------|
| `icon-scan.svg` | 顶部导航「扫码」 | 22×22 |
| `icon-scan-frame.svg` | 添加设备「扫码」、空态入口 | 22×22、16×16 |
| `icon-chevron-left.svg` | 返回（相册/设备详情/查看器） | 24×24、20×20 |
| `icon-chevron-right.svg` | 查看器下一页 | 20×20 |
| `icon-chevron-down.svg` | 任务指示条展开、下拉 | 14×14 |
| `icon-send.svg` | 发送到相框（选择栏/查看器/相框选择器） | 18×18、16×16、12×12 |
| `icon-close.svg` | 取消、关闭面板 | 18×18、20×20 |
| `icon-trash.svg` | 删除（选择栏、删除弹窗） | 18×18 |
| `icon-image.svg` | Tab「照片」、相册、权限空态插图 | 22×22、36×36、18×18 |
| `icon-settings.svg` | 前往设置、人物操作「设置」 | 16×16、18×18 |
| `icon-alert.svg` | 受限访问提示（警告） | 16×16，色 #FF9500 |
| `icon-plus.svg` | 添加设备、订阅更多 | 14×14 |
| `icon-bag.svg` | Tab「商城」、空态「商城」 | 22×22、36×36 |
| `icon-user.svg` | Tab「我的」、人物 | 22×22、36×36、56×56 容器 |
| `icon-more.svg` | 相册更多（三点） | 20×20 |
| `icon-frame.svg` | Tab「相框」、任务面板相框、欢迎页 Logo | 22×22、28×28、40×40 |
| `icon-frame-placeholder.svg` | 相框列表无照片时的封面占位符 | 32×32 |
| `icon-edit.svg` | 重命名、编辑 | 16×16 |
| `icon-users.svg` | 人物列表、多用户 | 16×16 |
| `icon-eye-off.svg` | 隐藏/不可见 | 16×16 |
| `icon-check.svg` | 相框选择器选中、结果成功 | 14×14、16×16、28×28 |
| `icon-map-pin.svg` | 地点/位置 | 10×10 |
| `icon-partial-fail.svg` | 发送结果部分失败 | 20×20 |

## 欢迎页专用

欢迎页使用的相框 Logo、插图和扫码按钮图标在 **`../welcome/`** 目录：

- `welcome-icon-frame.svg` — 顶部 Logo 图标
- `welcome-illustration.svg` — 相框+手机+连接线插图
- `welcome-btn-scan.svg` — 主按钮「扫码绑定」内图标

## 颜色与字重

- 选中/主操作：`#2563EB` (blue-600)
- 未选中/次要：`#9CA3AF` (gray-400)
- 危险/删除：`#F87171` (red-400) 或 `#FF3B30`
- 警告：`#FF9500`
- 描边粗细：多数 2–2.5，返回箭头 3，按设计稿为准。
