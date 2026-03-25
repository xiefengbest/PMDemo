# 背景与圆角 Drawable 资源

本目录为 Android 可直接使用的 `drawable` XML，复制到 `res/drawable/` 即可。部分场景需在代码中叠加**毛玻璃（blur）**，见下表说明。

## 一、文件清单与用途

| 文件名 | 用途 | 圆角 | 备注 |
|--------|------|------|------|
| **背景** | | | |
| `bg_bar_dark.xml` | 底部选择栏容器 | 16dp | 需叠加 blur 20dp，边框 1dp 白 10% |
| `bg_button_primary.xml` | 主操作按钮（发送到相框、确认发送） | 12dp | 阴影 shadow-lg blue-900/40 |
| `bg_button_capsule_gray.xml` | 全选、取消全选、重试 | 全圆角 | 无 |
| `bg_button_capsule_blue.xml` | 发送至相框（分组栏）、添加设备 | 全圆角 | 无 |
| `bg_button_capsule_red.xml` | 解绑、移除订阅 | 全圆角 | 无 |
| `bg_card.xml` | 设备卡片、订阅卡片 | 16dp | 边框 1dp #F3F4F6 |
| `bg_photo_placeholder.xml` | 照片占位、骨架屏矩形 | 8dp | 无 |
| `bg_group_switcher.xml` | 年/月/日分组切换器容器 | 8dp | 无 |
| `bg_group_switcher_selected.xml` | 分组切换器选中项 | 6dp | 白底 + shadow-sm |
| `bg_overlay_light.xml` | 弹窗遮罩、相框选择器遮罩 | 无 | 30% 黑 |
| `bg_overlay_medium.xml` | Action Sheet、删除弹窗遮罩 | 无 | 40% 黑 |
| `bg_task_indicator.xml` | 任务指示条 | 无 | 顶边 0.5dp 线 #1F007AFF |
| `bg_toast.xml` | Toast | 全圆角 | 85% 黑 |
| `bg_selection_cancel.xml` | 选择栏取消按钮 | 12dp | 白 10%，需 blur |
| `bg_selection_delete.xml` | 选择栏删除按钮 | 12dp | 白 5%，需 blur |
| `bg_progress_track.xml` | 进度条轨道 | 1.5dp | 高 3dp |
| `bg_progress_fill.xml` | 进度条填充 | 1.5dp | 失败态用 #FF9500 |
| `bg_photo_selection_overlay.xml` | 照片选中蒙层 | 12dp | 8% 蓝 #3B82F6 |
| `bg_check_circle_unselected.xml` | 照片未选中圆圈 | 圆形 | 需叠加 blur 4dp |
| `bg_check_circle_selected.xml` | 照片选中圆圈 | 圆形 | 白边 2dp，阴影 0 2px 8px rgba(59,130,246,0.4) |
| `bg_gradient_primary.xml` | 欢迎页 Logo、发送中图标 | 按容器 | 135° 渐变 |
| `bg_gradient_success.xml` | 成功状态图标 | 按容器 | 135° 渐变 |
| `bg_badge_unsent.xml` | 角标「待发送」 | 全圆角 | 红 #EF4444 |
| `bg_bubble_dark.xml` | 时间气泡、卡片角按钮 | 10dp | 需叠加 blur 10dp |

## 二、圆角系统（统一数值）

以下圆角在布局中与 drawable 配合使用，建议放入 `dimens.xml`：

| Token | 值(dp) | 用途 |
|-------|--------|------|
| radius_xs | 1.5 | 进度条 |
| radius_sm | 6 | 分组切换器内按钮 |
| radius_md | 8 | 分组切换器容器、骨架屏、占位 |
| radius_lg | 9 | 任务行图标容器 |
| radius_xl | 10 | Banner 图标、气泡 |
| radius_2xl | 12 | 标准按钮、照片选中、订阅卡片、选择栏按钮 |
| radius_3xl | 14 | iOS Alert |
| radius_4xl | 16 | 卡片、面板顶部、选择栏容器、Action Sheet |
| radius_5xl | 20 | Banner 通知卡片 |
| radius_full | 9999 | 胶囊按钮、头像、Toast、角标 |

## 三、毛玻璃（Blur）说明

以下背景在原型中使用了毛玻璃，Android 需在代码中实现（如 `RenderEffect.createBlurEffect` 或第三方库）：

| 场景 | blur | 背景色/透明度 |
|------|------|----------------|
| 导航栏 / Tab 栏 | 12dp | 白 80%–90% |
| 底部选择栏 | 20dp | rgba(28,28,30,0.9) |
| 任务面板 / iOS Alert | 40dp + saturate(180%) | 白 96%–97% |
| Banner 通知 | 40dp + saturate(180%) | rgba(30,30,34,0.95) |
| 未选中圆圈 | 4dp | rgba(0,0,0,0.15) |
| 选择栏取消/删除按钮 | 与栏一致 | 白 10% / 5% |

## 四、边框与分隔线

| 用途 | 色值 | 线宽 |
|------|------|------|
| 卡片边框 | #F3F4F6 | 1dp |
| 标准分隔线 | rgba(60,60,67,0.12) | 0.5dp |
| Alert 按钮分隔线 | rgba(60,60,67,0.18) | 0.5dp |
| 任务指示条顶线 | rgba(0,122,255,0.12) | 0.5dp |
| 选择栏容器边框 | 白 10% | 1dp |
| 照片选中边框 | #3B82F6 | 3dp |

## 五、阴影（elevation / shadow）

Android 可用 `elevation` 或 `MaterialShapeDrawable` 实现，关键数值：

| 场景 | 建议 |
|------|------|
| 设备卡片、头像 | elevation 2dp 或 shadow-sm |
| 精选封面 | elevation 4dp 或 shadow-md |
| 主操作按钮、发送按钮 | elevation 8dp + 蓝色阴影 rgba(30,64,175,0.4) |
| 选择栏、面板、Action Sheet | elevation 24dp 或 shadow-2xl |
| 选中圆圈 | 0 2px 8px rgba(59,130,246,0.4) |

以上色值、圆角、blur 与《UI设计规范》《UI视觉规格文档》一致，可直接用于 Android 复刻。
