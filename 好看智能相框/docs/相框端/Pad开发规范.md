# Pad开发规范

## 一、网络请求开发流程

本项目采用 **Retrofit + Coroutines + Flow + Model 解耦** 的现代化架构。以下是开发一个完整网络请求功能的标准步骤。

### 1. 定义接口层模型 (DTO)
在 `com.hk.phone_album.pad.model` 包下创建与后端 JSON 结构一致的数据类。
- **命名规范**：统一以 **`Resp`** 结尾。
- **职责**：仅用于解析原始网络数据，字段建议设为可空。

### 2. 定义 UI/业务层模型 (Domain Model)
定义 UI 界面真正使用的、字段友好的数据类。
- **职责**：提供给 UI 使用，保证字段非空且类型安全，屏蔽后端原始字段名。

### 3. 编写转换器 (Converter)
在 `com.hk.phone_album.pad.model.Convertor.kt` 中添加扩展函数。
- **职责**：将 DTO 映射为 UI Model，处理判空（如 `?: 0`）、字段重命名及逻辑转换（如 `String` 转 `Boolean`）。
- **核心价值**：实现 UI 层与接口层的解耦，隔离后端接口变动带来的影响。

### 4. 定义网络请求接口 (Service)
在 `com.hk.phone_album.pad.api.HkService.kt` 接口中添加请求方法。
- **规范**：请求参数统一使用 `@Body body: Map<String, @JvmSuppressWildcards Any>`。

### 5. 在 Repository 中封装业务方法
在 `com.hk.phone_album.pad.Repository.kt` 中实现并返回 `Flow` 对象。
- **标准写法**：
    ```kotlin
    fun getYourBusinessData(id: Int) = flow {
        // 1. 组装参数 (复用公共参数)
        val params = commonParams().apply { put("id", id) }
        // 2. 调用接口并立即使用扩展函数转换
        val result = service.getYourApiData(params).asDomainModel()
        // 3. 发射结果
        emit(result)
    }.flowOn(Dispatchers.IO) // 4. 强制在 IO 线程执行
    ```

### 6. 在 UI 层/启动步骤中调用
利用 `core:framework` 提供的 `.asResult()` 扩展函数来处理状态机。
- **标准写法**：
    ```kotlin
    scope.launch {
        Repository.getYourBusinessData(id)
            .asResult()
            .collectLatest { result ->
                when (result) {
                    is Result.Loading -> { /* 显示加载中 */ }
                    is Result.Success -> { /* 处理解耦后的干净数据 result.data */ }
                    is Result.Error -> { /* 处理异常情况 */ }
                }
            }
    }
    ```

---

## 二、Activity 开发规范

开发新页面时，应遵循统一的 `Activity` 编写规范，以保证代码风格和健壮性。

### 1. 基类继承与 ViewBinding
- **继承**: Activity 必须继承 `PadBaseActivity`，以自动获得屏幕常亮、沉浸式状态栏等通用处理。
- **ViewBinding**:
    1. 在 `onCreate` 方法中，通过 `ActivityXxxBinding.inflate(layoutInflater)` 初始化 `binding` 对象。
    2. 调用 `setContentView(binding.root)` 设置内容视图。
    3. 所有视图元素的访问都必须通过 `binding` 对象，严禁使用 `findViewById`。

### 2. 数据加载与 UI 刷新
- **协程作用域**: 网络请求等耗时操作必须在 `lifecycleScope.launch` 中执行，以实现生命周期安全。
- **数据流处理**: 严格遵守 `Repository` + `.asResult()` 的数据流处理模式。
