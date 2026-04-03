# PhotoPreviewFragment 架构文档

## 一、页面架构

```
PhotoMainActivity
└── ViewPager2 (禁用手势滑动)
    ├── HomeFragment (纯容器，实现 OnNavigationListener)
    │   └── Fragment 容器
    │       ├── PhotoPreviewFragment ← 用户未曾"开始分析"时显示
    │       └── PhotoAnalysisFragment ← 点击"开始分析"后永久显示
    ├── ShopFragment
    └── ProfileFragment (个人中心)
```

## 二、页面流转

```
用户首次进入 → 检查 SharedPreferences 是否曾"开始分析"
    │
    ├── 未曾"开始分析" → PhotoPreviewFragment
    │       │
    │       ├── 点击"开始分析" → 保存标志 + 切换到 PhotoAnalysisFragment(autoStart=true)
    │       │
    │       └── 发送照片成功 → 切换到 PhotoAnalysisFragment(autoStart=false)
    │
    └── 曾经"开始分析" → 直接显示 PhotoAnalysisFragment
```

**关键：单向流转，无法从 PhotoAnalysisFragment 返回到 PhotoPreviewFragment**

## 三、PhotoPreviewFragment 状态

### 状态 1：未绑定相框

```
┌─────────────────────────────────────────────────┐
│  标题栏：智能相框               发送(置灰)      │
├─────────────────────────────────────────────────┤
│                                                 │
│                    📱                           │
│                                                 │
│              还没有绑定相框                     │
│         扫码绑定后即可发送照片到相框            │
│                                                 │
│            ┌─────────────────┐                  │
│            │   [📷] 扫码绑定 │                  │
│            └─────────────────┘                  │
│                                                 │
├─────────────────────────────────────────────────┤
│   💡 想让AI帮你整理照片？         开始分析 >   │
└─────────────────────────────────────────────────┘
```

### 状态 2：已绑定相框

```
┌─────────────────────────────────────────────────┐
│  标题栏：全部照片 ▼              发送(N)        │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐               │
│  │ ✓  │ │     │ │ ✓  │ │     │               │
│  │照片1│ │照片2│ │照片3│ │照片4│               │
│  └─────┘ └─────┘ └─────┘ └─────┘               │
│                                                 │
│  （4列宫格，最多选择10张...）                   │
│                                                 │
├─────────────────────────────────────────────────┤
│   💡 想让AI帮你整理照片？         开始分析 >   │
└─────────────────────────────────────────────────┘
```

## 四、PhotoAnalysisFragment 标题栏

```
┌─────────────────────────────────────────────────┐
│  [➕]           智能相框              [📷]      │
│  添加                                  扫码     │
└─────────────────────────────────────────────────┘
```

- **添加按钮**：检查相框列表 → 打开图片选择器 → 选择相框 → 上传
- **扫码按钮**：跳转扫码页面绑定新相框

## 五、核心逻辑

### 5.1 HomeFragment

- 纯容器，仅负责 Fragment 切换
- 实现 `PhotoPreviewFragment.OnNavigationListener` 接口
- 根据 `SharedPreferences("photo_analysis_prefs")` 的 `has_ever_started_analysis` 标志判断初始显示

### 5.2 PhotoPreviewFragment

- 进入时先检查相框绑定状态（API 请求）
- 未绑定：显示绑定引导页
- 已绑定：请求权限后加载本地照片宫格
- 发送成功后调用 `navigationListener.onNavigateToAnalysis(autoStart=false)`

### 5.3 PhotoAnalysisFragment

- 自带标题栏（添加 + 扫码）
- 分析状态管理：等待分析 / 分析中 / 分析完成
- 支持 `autoAnalysis` 参数控制是否自动开始分析
- 同时保存持久化标志 `has_ever_started_analysis`

## 六、状态持久化

使用 `SharedPreferences("photo_analysis_prefs")`：

| Key | 说明 |
|-----|------|
| `has_ever_started_analysis` | 用户是否曾经点击过"开始分析" |

**保存时机**：
1. `PhotoPreviewFragment` 点击"开始分析"时
2. `PhotoAnalysisFragment` 内部点击开始分析时

## 七、Fragment 通信

```kotlin
// PhotoPreviewFragment 定义接口
interface OnNavigationListener {
    fun onNavigateToAnalysis(autoStart: Boolean)
}

// HomeFragment 实现接口
class HomeFragment : Fragment(), PhotoPreviewFragment.OnNavigationListener {
    override fun onNavigateToAnalysis(autoStart: Boolean) {
        val fragment = PhotoAnalysisFragment.newInstance(autoAnalysis = autoStart)
        childFragmentManager.beginTransaction()
            .replace(R.id.container, fragment)
            .commit()
    }
}
```
