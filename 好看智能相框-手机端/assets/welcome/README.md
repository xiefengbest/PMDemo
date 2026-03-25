# 欢迎页（绑定引导页）资源

本目录包含「绑定你的智能相框」欢迎页的矢量资源，供 Android / iOS 原生开发使用。

| 文件 | 说明 | 建议用法 |
|------|------|----------|
| `welcome-icon-frame.svg` | 顶部相框 Logo（40×40 逻辑尺寸） | 置于渐变圆角矩形容器内，描边设为白色 |
| `welcome-illustration.svg` | 中部插图 200×160（相框+手机+连接线） | 可直接用或按设计稿缩放 |
| `welcome-btn-scan.svg` | 主按钮内扫码图标（20×20） | 描边使用 currentColor，按钮文字为白色时设为白色 |

- **Android**：可转为 Vector Drawable（或使用 Android 的 Vector 导入）。
- **iOS**：可导入为 Asset 或转为 PDF/SFSymbol 使用。
- 尺寸均为 @1x；双端按需提供 @2x/@3x 或在原生侧按比例缩放。

详细视觉规格见项目根目录《UI视觉规格文档》**二十七、欢迎页（绑定引导页）**。
