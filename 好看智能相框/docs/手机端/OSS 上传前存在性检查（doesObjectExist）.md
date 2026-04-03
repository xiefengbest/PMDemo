# OSS 上传前存在性检查（doesObjectExist）

## 概述

Android 端在上传图片到阿里云 OSS 之前，会先通过 `doesObjectExist` 接口检查目标 objectKey 是否已存在于 OSS 上。如果已存在则直接跳过上传，返回上传成功。此逻辑可有效避免重复上传，节省带宽和上传时间。

**请 iOS 端对齐此逻辑。**

---

## 阿里云 OSS 官方文档

### 通用参考

| 文档 | 链接 |
|------|------|
| OSS HeadObject API 说明 | https://www.alibabacloud.com/help/en/oss/developer-reference/headobject |
| OSS SDK 总览（各语言） | https://help.aliyun.com/zh/oss/developer-reference/sdk-code-samples/ |

### Android SDK

| 文档 | 链接 |
|------|------|
| Android SDK 概述（v2.9.21） | https://www.alibabacloud.com/help/zh/oss/developer-reference/introduction/ |
| Android SDK 快速入门 | https://help.aliyun.com/zh/oss/developer-reference/getting-started-with-oss-sdk-for-android |
| Android SDK GitHub 源码 | https://github.com/aliyun/aliyun-oss-android-sdk |

Android SDK 中判断对象是否存在的方法：
```java
// com.alibaba.sdk.android.oss.OSS 接口
boolean doesObjectExist(String bucketName, String objectKey) throws ClientException, ServiceException;
```

### iOS SDK（Objective-C，v2.10.20）

| 文档 | 链接 |
|------|------|
| iOS SDK 概述 | https://help.aliyun.com/zh/oss/developer-reference/preface-5/ |
| iOS SDK 安装 | https://help.aliyun.com/zh/oss/developer-reference/installation-8 |
| iOS SDK 初始化 | https://help.aliyun.com/zh/oss/developer-reference/initialization-8 |
| iOS SDK 配置访问凭证 | https://help.aliyun.com/zh/oss/developer-reference/ios-configure-access-credentials |
| iOS SDK GitHub 源码 | https://github.com/aliyun/aliyun-oss-ios-sdk |

iOS SDK 中判断对象是否存在的方法（`OSSClient+Utilities`）：
```objectivec
// OSSClient.h - Utilities 分类
// 返回值：YES = 对象存在，NO && *error == nil = 对象不存在，NO && *error != nil = 发生错误
- (BOOL)doesObjectExistInBucket:(NSString *)bucketName
                      objectKey:(NSString *)objectKey
                          error:(const NSError **)error;
```

### iOS Swift SDK（预览版）

| 文档 | 链接 |
|------|------|
| Swift SDK 判断文件是否存在 | https://www.alibabacloud.com/help/en/oss/developer-reference/swift-determine-whether-an-object-exists |
| Swift SDK GitHub 源码 | https://github.com/aliyun/alibabacloud-oss-swift-sdk-v2 |
| Swift SDK 完整示例 | https://github.com/aliyun/alibabacloud-oss-swift-sdk-v2/blob/master/Samples/Sources/IsObjectExist/Program.swift |

Swift SDK 中判断对象是否存在的方法：
```swift
let result = try await client.isObjectExist(bucket, key)
```

### 权限要求

判断文件是否存在需要具有 **`oss:GetObject`** 权限。该接口底层为 HTTP HEAD 请求，不会产生数据传输费用。

---

## 核心流程

```
用户选择图片发布
    │
    ▼
计算图片文件 MD5 → 生成 objectKey（格式：release/image/{md5}.{ext}）
    │
    ▼
调用 doesObjectExist(bucketName, objectKey) 检查 OSS 上是否已存在该文件
    │
    ├── 已存在 → 直接返回上传成功（跳过实际上传）
    │
    └── 不存在 / 检查异常 → 继续执行正常的分片上传流程
```

---

## 详细实现说明

### 1. objectKey 的生成规则

objectKey 由文件内容的 **MD5 哈希值** + **文件扩展名** 组成，保证相同内容的文件会映射到同一个 key。

```
objectKey = "release/image/" + md5(fileContent) + "." + fileExtension
```

**示例：**
```
release/image/a1b2c3d4e5f6g7h8i9j0.jpg
release/image/f4e5d6c7b8a9012345ef.png
```

**关键点：**
- MD5 是基于**文件内容**计算的（非文件名），保证相同图片内容生成相同的 objectKey
- 支持从文件路径（`File`）或 `content:// URI`（通过 `ContentResolver.openInputStream`）计算 MD5
- 文件扩展名从原始文件路径或 MIME 类型获取，默认为 `jpg`

### 2. doesObjectExist 检查逻辑

在每张图片实际上传之前，调用 OSS SDK 的 `doesObjectExist` 方法：

```
doesObjectExist(bucketName, objectKey) → Boolean
```

**参数：**
| 参数 | 说明 | 示例值 |
|------|------|--------|
| `bucketName` | OSS Bucket 名称 | `hksocial`（公开）/ `hksocial-private`（私有） |
| `objectKey` | 文件在 OSS 上的路径 | `release/image/a1b2c3d4e5f6.jpg` |

**行为：**
- 返回 `true` → 文件已存在，**跳过上传**，直接回调上传成功
- 返回 `false` → 文件不存在，继续执行正常上传
- 抛出异常 → **不影响上传流程**，忽略异常并继续上传（容错处理）

### 3. Android 端参考代码

以下是 Android 端的核心实现（Kotlin）：

```kotlin
// AliOssUpload.kt - ossUploadImageSync 方法中

// 获取 Bucket 名称
val bucketName = if (isPrivate) "hksocial-private" else "hksocial"

// 检查对象是否已存在
try {
    if (ossClient.doesObjectExist(bucketName, objectKey) == true) {
        // 已存在，直接返回上传成功
        callback?.uploadSuccess(objectKey, source)
        Log.d(TAG, "objectKey is Exist, skip upload: $objectKey")
        return
    }
} catch (e: Exception) {
    // 如果检查失败，继续上传流程（不影响正常上传）
    Log.w(TAG, "doesObjectExist failed, continue upload: ${e.message}")
}

// 不存在，执行正常的分片上传...
```

### 4. objectKey 生成参考代码

```kotlin
// UploadImageTask.kt - uploadSingleImage 方法中

// 1. 计算文件内容的 MD5
val fileMd5 = calculateMd5(file)  // 或从 content:// URI 的 InputStream 计算

// 2. 获取文件扩展名
val fileExtension = getFileExtension(uri, filePath)  // 如 "jpg", "png"

// 3. 生成 objectKey
val objectKey = "release/image/${fileMd5}.${fileExtension}"
```

---

## iOS 端需要实现的内容

### 需要实现的核心功能

1. **文件 MD5 计算**
   - 基于文件内容（非文件名）计算 MD5
   - 支持从 `PHAsset` / 文件路径 / `Data` 计算 MD5
   - MD5 计算失败时使用时间戳作为后备值（但此时无法命中存在性检查缓存）

2. **objectKey 生成**
   - 格式：`release/image/{md5}.{ext}`
   - 与 Android 端保持一致，确保同一张图片在两端生成相同的 objectKey

3. **doesObjectExist 检查**
   - 在每张图片上传前调用
   - 使用阿里云 OSS iOS SDK 的 `doesObjectExistInBucket:objectKey:error:` 方法
   - 如果对象已存在则跳过上传，直接回调成功
   - 检查失败时降级为正常上传（不能阻塞上传流程）

### iOS 阿里云 OSS SDK 参考代码

#### 方式1：使用 `doesObjectExistInBucket` （推荐，SDK 原生支持）

```objectivec
// Objective-C - 使用 OSSClient 的 Utilities 分类方法
// 需要 pod 'AliyunOSSiOS' (>= 2.10.x)

NSError *error = nil;
BOOL exist = [ossClient doesObjectExistInBucket:bucketName
                                      objectKey:objectKey
                                          error:&error];

if (exist) {
    // 对象已存在，跳过上传，直接回调成功
    NSLog(@"objectKey already exists, skip upload: %@", objectKey);
    if (callback) {
        callback(objectKey, YES, nil);
    }
    return;
}

if (error) {
    // 检查出错，降级为正常上传（不阻塞上传流程）
    NSLog(@"doesObjectExist failed, continue upload: %@", error.localizedDescription);
}

// 对象不存在，继续正常上传流程...
```

```swift
// Swift 等效写法（使用 Objective-C SDK）
do {
    var error: NSError?
    let exist = ossClient.doesObjectExist(inBucket: bucketName,
                                          objectKey: objectKey,
                                          error: &error)
    if exist {
        // 对象已存在，跳过上传
        print("objectKey already exists, skip upload: \(objectKey)")
        callback?(objectKey, true, nil)
        return
    }
    if let error = error {
        // 检查出错，降级为正常上传
        print("doesObjectExist failed, continue upload: \(error.localizedDescription)")
    }
} catch {
    // 异常降级，继续上传
    print("doesObjectExist exception, continue upload: \(error)")
}
// 对象不存在，继续正常上传流程...
```

#### 方式2：使用 `headObject` 请求（底层等价实现）

```objectivec
// Objective-C - 如果 SDK 版本不支持 doesObjectExistInBucket，可用 headObject 替代
OSSHeadObjectRequest *head = [OSSHeadObjectRequest new];
head.bucketName = bucketName;
head.objectKey = objectKey;

OSSTask *task = [ossClient headObject:head];
[task waitUntilFinished];

if (!task.error) {
    // headObject 成功 → 对象存在，跳过上传
    NSLog(@"objectKey already exists, skip upload: %@", objectKey);
    if (callback) {
        callback(objectKey, YES, nil);
    }
    return;
}

// headObject 失败
if ([task.error.domain isEqualToString:@"com.aliyun.oss.serverError"] 
    && task.error.code == -404) {
    // 404 表示对象不存在，继续正常上传
    NSLog(@"objectKey not found, proceed with upload");
} else {
    // 其他错误，降级为正常上传
    NSLog(@"headObject error, continue upload: %@", task.error);
}

// 继续正常上传流程...
```

#### 方式3：使用 Swift SDK（预览版）

```swift
// Swift SDK (alibabacloud-oss-swift-sdk-v2) - 如果项目使用了新版 Swift SDK
import AlibabaCloudOSS

do {
    let exist = try await client.isObjectExist(bucketName, objectKey)
    if exist {
        // 对象已存在，跳过上传
        print("objectKey already exists, skip upload: \(objectKey)")
        callback?(objectKey, true, nil)
        return
    }
} catch {
    // 检查失败，降级为正常上传
    print("isObjectExist failed, continue upload: \(error)")
}

// 对象不存在，继续正常上传流程...
```

### 注意事项

1. **容错优先**：`doesObjectExist` 检查失败时必须降级为正常上传，不能因为检查异常而导致上传失败
2. **性能考虑**：`doesObjectExist` 底层是一次 HTTP HEAD 请求（参见 [HeadObject API](https://www.alibabacloud.com/help/en/oss/developer-reference/headobject)），网络延迟通常在 50-200ms，相比完整上传（可能数秒）是值得的，且不产生数据传输费用
3. **STS 凭证**：检查和上传使用同一套 STS 临时凭证，凭证过期时需要刷新
4. **Bucket 选择**：根据 `isPrivate` 参数选择对应的 Bucket
   - 公开：`hksocial`
   - 私有：`hksocial-private`
5. **OSS 端点**：`https://oss-accelerate.aliyuncs.com`（全球加速）
6. **权限要求**：需要 `oss:GetObject` 权限

---

## OSS 配置信息

| 配置项 | 值 |
|--------|------|
| Endpoint | `https://oss-accelerate.aliyuncs.com` |
| Bucket（公开） | `hksocial` |
| Bucket（私有） | `hksocial-private` |
| 图片 CDN Host | `https://hksocial-cdn.levect.com/` |
| objectKey 前缀（发布图片） | `release/image/` |
| objectKey 前缀（壁纸） | `release/wallpaper/image/` |
| objectKey 前缀（头像） | `headerimg/` |
| objectKey 前缀（视频） | `release/video/` |
| objectKey 前缀（视频封面） | `release/video/cover/` |

---

## 预期收益

- **减少重复上传**：用户重复发布同一张图片时，跳过实际上传，节省流量和时间
- **加速上传流程**：已存在的文件直接返回成功，大幅缩短批量上传时间
- **降低 OSS 存储成本**：相同内容不重复存储（因为 objectKey 相同会覆盖，但检查后可避免无意义的覆盖上传）
