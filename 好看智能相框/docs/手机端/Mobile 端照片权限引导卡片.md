# Mobile 端照片权限引导卡片

## 一、需求背景

### 1.1 当前问题

Android 14+ 引入了**部分照片访问权限**机制：
- 用户可以选择"允许全部"或"选择部分照片"
- 如果用户只授予"部分照片"权限，App 只能访问用户手动选择的那些照片
- **自动识别功能无法完整工作**：无法扫描和整理用户的全部照片

### 1.2 目标

当用户**未授予"全部照片"权限**时，在**本地照片区域第一个位置**显示引导卡片：
- 明确告知用户为什么需要"全部照片"权限
- 说明授权后的好处（自动识别、智能整理）
- 一键引导用户去设置中更改权限

---

## 二、交互设计

### 2.1 引导卡片位置

**放在"本地照片"横向月份卡片的第一个位置**

```
                              ← 横向滚动 →

┌────────────┐  ┌──────────────────┐  ┌──────────────────┐
│            │  │           待发送  │  │           待发送  │
│   📷       │  │                  │  │                  │
│            │  │   ╔══════════╗   │  │   ╔══════════╗   │
│  允许访问   │  │   ║  封面图  ║   │  │   ║  封面图  ║   │
│  全部照片   │  │   ║          ║   │  │   ║          ║   │
│            │  │   ╚══════════╝   │  │   ╚══════════╝   │
│  授权后可   │  │                  │  │                  │
│ 自动筛选... │  │ 2026年1月        │  │ 2025年12月       │
│            │  │ 28张照片         │  │ 35张照片         │
│ [去设置]   │  │                  │  │                  │
└────────────┘  └──────────────────┘  └──────────────────┘
   🎯引导卡片          普通月份              普通月份
   (宽度较小)
```

### 2.2 显示条件

| Android 版本 | 显示条件 |
|-------------|---------|
| **Android 14+** | 未授予"全部照片"权限（只有部分或无权限） |
| **Android 13** | 未授予 READ_MEDIA_IMAGES 权限 |
| **Android 12 及以下** | 未授予 READ_EXTERNAL_STORAGE 权限 |

---

## 三、UI 设计（Apple 风格）

### 3.1 卡片布局

采用 Apple Human Interface Guidelines 风格设计：

```
┌────────────────┐
│                │
│    [ 弹性 ]    │
│                │
│      📷       │  ← 图标（40dp，Apple 蓝色）
│                │
│  允许访问全部照片 │  ← 标题（15sp，粗体，深色）
│                │
│ 授权后可自动筛选  │  ← 描述（12sp，灰色）
│ 出优质照片，确认  │
│ 后即可发送        │
│                │
│    [ 弹性 ]    │
│                │
│   (去设置)     │  ← 胶囊按钮（Apple 蓝色）
│                │
└────────────────┘
```

### 3.2 设计规范

| 元素 | 规格 |
|-----|------|
| **卡片宽度** | 屏幕宽度 × 45%（比月份卡片窄） |
| **卡片高度** | 与月份卡片一致（屏幕宽度 × 58%） |
| **卡片圆角** | 12dp |
| **背景** | 柔和渐变（#F5F7FA → #EDF1F7） |
| **图标** | 40dp，Apple 蓝色 #007AFF |
| **标题** | 15sp，粗体，深色 #1C1C1E |
| **描述** | 12sp，灰色 #8E8E93 |
| **按钮** | 胶囊型，32dp 高，Apple 蓝色 #007AFF |

### 3.3 文案

| 元素 | 内容 |
|-----|------|
| **标题** | 允许访问全部照片 |
| **描述** | 授权后可自动筛选出优质照片，确认后即可发送 |
| **按钮** | 去设置 |

---

## 四、技术实现

### 4.1 权限检查（PhotoPermissionHelper.kt）

```kotlin
/**
 * 检查是否拥有全部照片访问权限
 */
fun hasFullPhotoAccess(context: Context): Boolean {
    return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.UPSIDE_DOWN_CAKE) {
        // Android 14+：需要检查是否是完全访问而非部分访问
        val hasReadImages = ContextCompat.checkSelfPermission(
            context,
            Manifest.permission.READ_MEDIA_IMAGES
        ) == PackageManager.PERMISSION_GRANTED
        
        val hasPartialAccess = ContextCompat.checkSelfPermission(
            context,
            Manifest.permission.READ_MEDIA_VISUAL_USER_SELECTED
        ) == PackageManager.PERMISSION_GRANTED
        
        // 有完整权限：READ_MEDIA_IMAGES 已授予，且不是通过部分选择获得的
        hasReadImages && !hasPartialAccess
    } else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
        // Android 13
        ContextCompat.checkSelfPermission(
            context,
            Manifest.permission.READ_MEDIA_IMAGES
        ) == PackageManager.PERMISSION_GRANTED
    } else {
        // Android 12 及以下
        ContextCompat.checkSelfPermission(
            context,
            Manifest.permission.READ_EXTERNAL_STORAGE
        ) == PackageManager.PERMISSION_GRANTED
    }
}

/**
 * 是否需要显示权限引导卡片
 */
fun shouldShowPermissionGuide(context: Context): Boolean {
    return !hasFullPhotoAccess(context)
}
```

### 4.2 Adapter 实现（MonthAdapter.kt）

```kotlin
class MonthAdapter(...) : RecyclerView.Adapter<RecyclerView.ViewHolder>() {

    companion object {
        private const val VIEW_TYPE_GUIDE = 0
        private const val VIEW_TYPE_MONTH = 1
        private const val VIEW_TYPE_LOADING = 2
    }

    private var showGuideCard = false

    /**
     * 设置引导卡片显示状态
     */
    fun setShowGuideCard(show: Boolean) {
        if (this.showGuideCard != show) {
            this.showGuideCard = show
            notifyDataSetChanged()
        }
    }

    override fun getItemViewType(position: Int): Int {
        if (position == 0 && showGuideCard) {
            return VIEW_TYPE_GUIDE
        }
        // ... 其他类型判断
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        val screenSize = parent.context.getScreenSize()
        val itemWidth = (screenSize.width * 0.58).toInt()
        
        return when (viewType) {
            VIEW_TYPE_GUIDE -> {
                val view = LayoutInflater.from(parent.context)
                    .inflate(R.layout.item_month_guide, parent, false)
                // 引导卡片宽度较小（45%）
                val guideWidth = (screenSize.width * 0.45).toInt()
                val layoutParams = RecyclerView.LayoutParams(guideWidth, itemWidth)
                view.layoutParams = layoutParams
                GuideViewHolder(view)
            }
            // ... 其他类型
        }
    }

    /**
     * 引导卡片 ViewHolder
     */
    class GuideViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        private val btnGoSettings: MaterialButton = itemView.findViewById(R.id.btnGoSettings)
        
        fun bind(position: Int, itemCount: Int) {
            btnGoSettings.setOnClickListener {
                openAppSettings()
            }
        }
        
        private fun openAppSettings() {
            val context = itemView.context
            val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS).apply {
                data = Uri.fromParts("package", context.packageName, null)
            }
            context.startActivity(intent)
        }
    }
}
```

### 4.3 布局文件（item_month_guide.xml）

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:cardCornerRadius="@dimen/dp_12"
    app:cardElevation="0dp"
    app:cardBackgroundColor="@android:color/transparent">

    <!-- 渐变背景 -->
    <View
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@drawable/bg_guide_card_gradient" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center_horizontal"
        android:orientation="vertical"
        android:paddingHorizontal="@dimen/dp_16"
        android:paddingVertical="@dimen/dp_16">

        <!-- 顶部弹性间距 -->
        <Space
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1" />

        <!-- 图标 -->
        <ImageView
            android:id="@+id/ivIcon"
            android:layout_width="@dimen/dp_40"
            android:layout_height="@dimen/dp_40"
            android:src="@drawable/ic_guide_photo" />

        <Space
            android:layout_width="match_parent"
            android:layout_height="@dimen/dp_12" />

        <!-- 标题 -->
        <TextView
            android:id="@+id/textViewTitle"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/guide_card_title"
            android:textColor="#1C1C1E"
            android:textSize="@dimen/sp_15"
            android:textStyle="bold"
            android:gravity="center" />

        <Space
            android:layout_width="match_parent"
            android:layout_height="@dimen/dp_4" />

        <!-- 描述 -->
        <TextView
            android:id="@+id/textViewDescription"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/guide_card_description"
            android:textColor="#8E8E93"
            android:textSize="@dimen/sp_12"
            android:gravity="center"
            android:lineSpacingMultiplier="1.2" />

        <!-- 底部弹性间距 -->
        <Space
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1" />

        <!-- 胶囊按钮 -->
        <com.google.android.material.button.MaterialButton
            android:id="@+id/btnGoSettings"
            android:layout_width="wrap_content"
            android:layout_height="@dimen/dp_32"
            android:paddingHorizontal="@dimen/dp_20"
            android:text="@string/guide_card_button"
            android:textColor="@color/white"
            android:textSize="@dimen/sp_13"
            android:insetTop="0dp"
            android:insetBottom="0dp"
            android:minWidth="0dp"
            android:minHeight="0dp"
            app:backgroundTint="#007AFF"
            app:cornerRadius="@dimen/dp_16"
            app:elevation="0dp" />

    </LinearLayout>

</androidx.cardview.widget.CardView>
```

### 4.4 渐变背景（bg_guide_card_gradient.xml）

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <gradient
        android:angle="180"
        android:startColor="#F5F7FA"
        android:endColor="#EDF1F7"
        android:type="linear" />
    <corners android:radius="12dp" />
</shape>
```

### 4.5 图标（ic_guide_photo.xml）

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="40dp"
    android:height="40dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    
    <!-- 照片框 -->
    <path
        android:fillColor="#007AFF"
        android:pathData="M5,4C3.34,4 2,5.34 2,7v10c0,1.66 1.34,3 3,3h14c1.66,0 3,-1.34 3,-3V7c0,-1.66 -1.34,-3 -3,-3H5zM5,6h14c0.55,0 1,0.45 1,1v10c0,0.55 -0.45,1 -1,1H5c-0.55,0 -1,-0.45 -1,-1V7C4,6.45 4.45,6 5,6z" />
    
    <!-- 山峰 -->
    <path
        android:fillColor="#007AFF"
        android:pathData="M6,16l3,-4l2,2.5l4,-5l5,6.5H6z" />
    
    <!-- 太阳 -->
    <path
        android:fillColor="#007AFF"
        android:pathData="M8,9.5m-1.5,0a1.5,1.5 0,1 1,3 0a1.5,1.5 0,1 1,-3 0" />

</vector>
```

### 4.6 Fragment 权限检查（PhotoAnalysisFragment.kt）

```kotlin
override fun onResume() {
    super.onResume()
    // 每次 onResume 重新检查权限状态
    hasPermission = PhotoPermissionHelper.isAllPermissionGranted(requireContext())
    // 更新引导卡片显示状态
    updateGuideCardVisibility()
}

private fun updateGuideCardVisibility() {
    val shouldShowGuide = PhotoPermissionHelper.shouldShowPermissionGuide(requireContext())
    monthAdapter.setShowGuideCard(shouldShowGuide)
}

private fun updateLocalAlbumUI(
    monthGroups: List<MonthGroup>,
    isLoading: Boolean,
    hasMore: Boolean
) {
    val shouldShowGuide = PhotoPermissionHelper.shouldShowPermissionGuide(requireContext())
    
    // 无权限时，显示引导卡片
    if (!hasPermission) {
        binding.llLocalAlbumTitle.visibility = View.VISIBLE
        binding.recyclerViewMonths.visibility = View.VISIBLE
        monthAdapter.setShowGuideCard(shouldShowGuide)
        monthAdapter.updateData(emptyList(), false)
        return
    }
    
    // 有权限时的正常逻辑...
}
```

---

## 五、字符串资源

```xml
<!-- strings.xml -->
<string name="guide_card_title">允许访问全部照片</string>
<string name="guide_card_description">授权后可自动筛选出优质照片，确认后即可发送</string>
<string name="guide_card_button">去设置</string>
```

---

## 六、用户路径

```
用户打开本地照片页面
     │
     ▼
检测到未授予"全部照片"权限
     │
     ▼
横向月份卡片第一个位置显示引导卡片
     │
     ▼
点击"去设置"按钮
     │
     ▼
跳转到系统设置页面
     │
     ▼
用户选择"允许全部"
     │
     ▼
返回 App，引导卡片消失，正常显示月份列表
```

---

## 七、相关文件

| 文件 | 说明 |
|-----|------|
| `PhotoPermissionHelper.kt` | 权限检查工具类 |
| `PhotoAnalysisFragment.kt` | 页面逻辑，控制引导卡片显示 |
| `MonthAdapter.kt` | 适配器，支持引导卡片类型 |
| `item_month_guide.xml` | 引导卡片布局 |
| `bg_guide_card_gradient.xml` | 渐变背景 |
| `ic_guide_photo.xml` | 照片图标 |
| `strings.xml` | 文案资源 |
| `dimens.xml` | 尺寸资源 |
