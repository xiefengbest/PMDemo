# 好看云壁纸 - 项目上下文

## 项目信息
- **名称**: 好看云壁纸
- **路径**: `/Users/haokan/Documents/code/AndroidStudioProjects/PMDemo/好看云壁纸`
- **主要文件**: `index.html`（单文件应用）
- **技术栈**: HTML + CSS + JavaScript（无框架，使用 Tailwind CSS）

## 页面架构

### 静态 HTML 页面（通过 CSS 类控制显示/隐藏）
| 页面 | ID | z-index | 说明 |
|------|-----|---------|------|
| 创作者页 | `#creator-modal` | 210 | 创作者信息 + 作品列表 |
| 详情页 | `#detail-modal` | 450 | 壁纸产品详情 + 订阅 |
| 全屏壁纸预览 | `#wallpaper-fullscreen-preview` | 500 | 壁纸预览 + 操作按钮 |
| 二级页面容器 | `#secondary-page` | 400 | 包含 4 个子页面 |
| 锁屏引导 | `#lock-screen-guide-modal` | 550 | 设置壁纸引导 |
| 壁纸模式选择 | `#wallpaper-mode-overlay` | 600 | 单独设为壁纸/加入轮播 |

### 二级页面子页面（静态 HTML）
| 页面 | ID | 说明 |
|------|-----|------|
| 账号管理 | `#secondary-account-page` | 绑定手机、修改密码、账号注销 |
| 关于我们 | `#secondary-about-page` | 公司信息、版本 |
| 我的合辑 | `#secondary-subscriptions-page` | 已订阅壁纸列表 |
| 轮播壁纸 | `#secondary-rotation-page` | 已加入轮播的壁纸列表 |

### 动态渲染页面
| 页面 | 函数 | 说明 |
|------|------|------|
| 锁屏全屏预览 | `showLockScreenPreview()` | 个人照片预览，动态创建 DOM |

## 核心数据

### 产品数据
- **变量**: `CAT_PRODUCTS` 数组
- **画境集**: id=102，组图壁纸，使用订阅逻辑但文案不同
- **创作者数据**: `CREATORS` 数组

### 订阅状态
- **变量**: `subscribedProductIds` (Set)
- **评价**: `userReviewsByProduct` (Object)

### 轮播壁纸
- **存储**: `localStorage` 的 `myRotationWallpapers`
- **函数**: `getRotationWallpapers()`, `saveRotationWallpapers()`, `addToRotation()`, `removeFromRotation()`

## 核心功能

### 壁纸设置逻辑
- **普通壁纸**: 订阅 → 预加载 → 锁屏引导 → 评价
- **组图壁纸（画境集）**: 设为壁纸 → 预加载 → 锁屏引导 → 评价
- **文案差异**: 
  - 普通: "立即订阅"/"已订阅"/"取消订阅"
  - 组图: "设为壁纸"/"正在使用"/"停止使用"

### 全屏预览
- **壁纸预览**: `#wallpaper-fullscreen-preview`，有操作按钮
- **照片预览**: `.ls-fullscreen`，点击屏幕退出
- **日期时间控件**: 两个页面样式一致（日期在上，时间在下）

### 平台检测
- **函数**: `getPhotoPermissionPlatform()`
- **返回**: 'ios' 或 'android'
- **iOS 限制**: 不显示轮播壁纸入口

## 代码规范
- 使用 Tailwind CSS 类名
- 模态框通过 `.show` 类控制显示
- 二级页面通过 `openSecondaryPage(pageId, title)` 打开
- 关闭二级页面: `closeSecondaryPage()`

## 重要函数
| 函数 | 说明 |
|------|------|
| `openDetail(id)` | 打开壁纸详情页 |
| `openCreatorPage()` | 打开创作者页 |
| `openSecondaryPage(pageId, title)` | 打开二级页面 |
| `closeSecondaryPage()` | 关闭二级页面 |
| `showLockScreenPreview(images, index)` | 锁屏全屏预览 |
| `openGenericFullscreenPreview(images, index, title, tushuo)` | 通用全屏预览 |
| `openRotationWallpaperPreview(index)` | 轮播壁纸全屏预览 |
| `setWallpaperFromFullscreen()` | 单独设为壁纸 |
| `addToRotationFromFullscreen()` | 加入轮播 |
| `showWallpaperModeModal()` | 显示壁纸模式选择弹窗 |

## 最近提交
1. `2ef6751` - 将二级页面改为静态 HTML
2. `c32291b` - 统一全屏预览日期时间控件样式
3. `cca7e22` - iOS 隐藏轮播壁纸入口
4. `50e04d7` - 修复全屏预览和轮播壁纸页面问题
5. `bb6fba7` - 优化组图壁纸交互

## 待优化项
- 锁屏全屏预览 (`.ls-fullscreen`) 可考虑改为静态 HTML
- 页面层级已优化，详情页 (450) > 二级页面 (400)
