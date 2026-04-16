# 好看云壁纸 - 项目规则

## 项目信息
- **名称**: 好看云壁纸
- **路径**: `/Users/haokan/Documents/code/AndroidStudioProjects/PMDemo/好看云壁纸`
- **主要文件**: `index.html`（单文件应用）
- **技术栈**: HTML + CSS + JavaScript（无框架，使用 Tailwind CSS）

## 开发规范
1. **单文件应用**: 所有代码在 `index.html` 中
2. **Tailwind CSS**: 使用 Tailwind 类名，避免内联样式
3. **模态框**: 通过 `.show` 类控制显示/隐藏
4. **二级页面**: 使用 `openSecondaryPage(pageId, title)` 打开
5. **平台检测**: 使用 `getPhotoPermissionPlatform()` 检测 iOS/Android
6. **Git 提交**: 永远不要自动提交到 git，只有在用户明确要求时才执行

## 核心数据结构
- `CAT_PRODUCTS`: 产品数据数组
- `CREATORS`: 创作者数据数组
- `subscribedProductIds`: 已订阅产品 ID 集合
- `localStorage.myRotationWallpapers`: 轮播壁纸数据

## 重要页面层级
- 创作者页: 210
- 二级页面: 400
- 详情页: 450
- 全屏壁纸预览: 500
- 锁屏引导: 550
- 壁纸模式选择: 600

## 特殊产品
- **画境集 (id=102)**: 组图壁纸，使用订阅逻辑但文案不同
  - 按钮: "设为壁纸" → "正在使用" → "停止使用"
  - 隐藏订阅权益模块
  - 不显示壁纸更新提醒

## 平台差异
- **iOS**: 不显示轮播壁纸入口
- **Android**: 显示轮播壁纸入口

## 新任务开始
1. 先读取 `PROJECT_CONTEXT.md` 了解项目背景
2. 查看 `git log` 了解最近变更
3. 确认用户需求后再开始开发
