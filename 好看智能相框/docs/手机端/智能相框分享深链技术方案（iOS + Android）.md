# 智能相框分享深链技术方案（iOS + Android）

## 一、目标

通过分享深链（Deep Link）实现：

- 家人有 App：点击链接直接打开 App 并加入相框
- 家人无 App：跳转应用商店下载后自动继续绑定
- 支持 WhatsApp、短信、浏览器等分享渠道

该方案适用于 Android 与 iOS 双平台统一实现。

---

## 二、用户流程

```
Owner 点击【分享相框】
        ↓
生成邀请链接
        ↓
分享给家人
        ↓
家人点击链接
        ↓
有 App → 直接打开并加入相框
无 App → 留在当前 H5 下载页 → 安装后继续加入
```

---

## 三、深链结构设计

推荐使用 HTTPS 深链，结合我们的实际下载地址：

```
https://92-cn.levect.com/frames/download/index.html?token=abc123
```

不建议使用自定义 Scheme：

```
yourapp://invite?token=abc123
```

原因：

- HTTPS 支持 iOS Universal Links 和 Android App Links
- 可在浏览器、聊天软件中正常打开
- 完美兼容未安装 App 的用户（未安装时直接展示 H5 下载引导页）

---

## 四、后端设计

### 1. 邀请 Token 表

| 字段 | 类型 | 说明 |
|---|---|---|
| token | string | 邀请码 |
| device_id | string | 相框 ID |
| expire_time | datetime | 过期时间 |
| used_count | int | 已使用次数 |
| created_by | string | 创建者用户 ID |

---

### 2. 生成邀请链接 API

```
POST /device/invite
```

返回：

```json
{
  "invite_url": "https://92-cn.levect.com/frames/download/index.html?token=abc123"
}
```

---

## 五、iOS 实现（Universal Links）

### 1. apple-app-site-association

文件路径必须部署在我们的域名根目录下（需 HTTPS）：

```
https://92-cn.levect.com/.well-known/apple-app-site-association
```

示例内容：

```json
{
  "applinks": {
    "details": [
      {
        "appID": "TEAMID.com.yourapp.frame",
        "paths": ["/frames/download/index.html"]
      }
    ]
  }
}
```

### 2. App 内处理逻辑

```swift
func scene(_ scene: UIScene,
           continue userActivity: NSUserActivity) {

    let url = userActivity.webpageURL
    handleInvite(url)
}
```

---

## 六、Android 实现（App Links）

### 1. AndroidManifest.xml

```xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.BROWSABLE"/>
    <category android:name="android.intent.category.DEFAULT"/>

    <data
        android:scheme="https"
        android:host="92-cn.levect.com"
        android:path="/frames/download/index.html"/>
</intent-filter>
```

### 2. assetlinks.json

文件路径：

```
https://92-cn.levect.com/.well-known/assetlinks.json
```

示例：

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.yourapp.frame",
    "sha256_cert_fingerprints": ["XX:XX:XX"]
  }
}]
```

---

## 七、App 内邀请处理流程

当 App 接收到深链（或新用户安装后解析到分享口令/参数）：

```
https://92-cn.levect.com/frames/download/index.html?token=abc123
```

### 1. 基础逻辑流转

```
1. 解析 token
2. 检查登录状态
3. 未登录 → 强制跳转登录/注册页
4. 登录完成 / 已登录 → 请求后端校验 token 并获取相框信息
5. 弹窗引导用户确认加入（若已加入则提示“您已在此相框中”）
6. 用户点击“确认加入” → 执行加入相框
```

### 2. UI 交互要求（加入相框弹窗）

在真正绑定相框前，必须展示弹窗提示用户确认。

- **弹窗内容**：展示邀请人、相框名称等信息（需后端接口配合返回），无需展示当前成员列表。
- **权限说明**：弹窗中需明确告知用户加入后可以：1）向相框发送照片和视频；2）管理相框设置与所有照片。
- **已加入状态处理**：如果后端返回用户已是该相框成员，弹窗改为提示弹窗：“您已加入 [相框名称]”，点击按钮可直接进入相框详情或主页。
- **正常加入状态处理**：包含“取消”和“加入”按钮，点击“加入”调用加入相框接口。
- **加入后权限**：加入成功后，该用户拥有与 Owner 相同的相框管理权限，包括查看/删除所有照片、修改相框设置等。

### 3. 伪代码示例

```kotlin
fun handleInviteToken(token: String) {
    if (!isLogin()) {
        // 保存邀请状态，待登录后恢复流程
        savePendingInvite(token)
        goLogin()
    } else {
        // 验证 token 并获取相框信息
        val info = verifyToken(token)
        if (info.isAlreadyMember) {
            showAlreadyJoinedDialog(info.frameName)
        } else {
            showJoinConfirmDialog(info) {
                // 用户点击确认后执行加入
                joinDevice(token)
            }
        }
    }
}
```

---

## 八、未安装 App 的用户处理

用户点击链接后会打开我们的 H5 下载页：

```
https://92-cn.levect.com/frames/download/index.html?token=abc123
```

页面职责：

- 检测设备平台（iOS / Android）
- 尝试通过 Universal Links / App Links 直接唤起 App（通常由系统自动处理，H5 可增加显式打开按钮）
- 若系统未唤起，用户可点击页面上的下载按钮跳转对应应用商店或下载 APK
- 为了兼容部分无法直接使用 App Links 的场景（如微信内），H5 页面可引导用户“在浏览器中打开”
- 可通过剪贴板等方式（如使用 Sharetrace / OpenInstall 等第三方库，或自建剪贴板口令）在用户安装后还原邀请码，实现无缝绑定

---

## 九、国内厂商深链适配（Android 重点）

### 1. 问题背景

标准 Android App Links 的 `autoVerify` 机制依赖 Google Play Services 进行域名验证。在国内大多数 Android 设备上缺少 Google 服务，导致 App Links 验证失败，链接会直接在浏览器中打开而非唤起 App。

因此在国内市场，除了标准 App Links 外，还需要做**厂商深链适配**或**接入第三方深链服务**来保证用户体验。

> iOS 端无此问题，Universal Links 在所有 iOS 设备上表现一致。

### 2. 推荐方案对比

| 方案 | 优点 | 缺点 | 推荐场景 |
|---|---|---|---|
| 第三方深链服务（OpenInstall / Sharetrace） | 一次接入覆盖全厂商；支持延迟深链；维护成本低 | 有费用；数据经过第三方 | 快速上线、资源有限 |
| 逐一适配各厂商深链 | 可控性强；无第三方依赖 | 工作量大；各厂商标准不统一；需持续跟进 | 有充足研发资源且对数据安全要求高 |
| 自建剪贴板口令兜底 | 无外部依赖；全平台兼容 | 用户体验稍差（需手动粘贴） | 作为兜底方案补充 |

### 3. 方案一：接入第三方深链服务（推荐）

第三方深链服务已内部处理好各厂商差异、微信/QQ 等社交平台的限制以及延迟深链（Deferred Deep Link），是成本最低的方案。

#### 3.1 OpenInstall

- **官网**：<https://www.openinstall.io/>
- **接入指南（总览）**：<https://www.openinstall.io/doc/guide.html>
- **Android SDK 集成文档**：<https://www.openinstall.com/doc/android_sdk.html>
- **iOS SDK 集成文档**：<https://www.openinstall.io/doc/ios_sdk.html>
- **SDK 下载中心**：<https://www.openinstall.io/download.html>

核心能力：

- 一键拉起（已安装直接打开 App 并传参）
- 延迟深链（未安装 → 应用商店下载 → 首次打开自动还原参数）
- 渠道统计
- 免填邀请码

#### 3.2 Sharetrace

- **官网**：<https://www.sharetrace.com/>
- **Android SDK 集成文档**：<https://www.sharetrace.com/docs/guide/android.html>
- **iOS SDK 集成文档**：<https://www.sharetrace.com/docs/guide/ios.html>
- **SDK 下载中心**：<https://www.sharetrace.com/download.html>

核心能力与 OpenInstall 类似，支持免填邀请码安装与一键拉起。

#### 3.3 接入后的邀请流程调整

```
Owner 点击【分享相框】
        ↓
后端生成 invite token
        ↓
App 调用第三方 SDK 创建带参短链（参数含 token）
        ↓
分享短链给家人
        ↓
家人点击短链
        ↓
├─ 已安装 App → 第三方 SDK 唤起 App 并回传 token → 走加入流程
└─ 未安装 App → 跳转应用商店 → 安装后首次打开 → SDK 回调还原 token → 走加入流程
```

### 4. 方案二：逐一适配各厂商深链

如果选择自行适配，需要在以下厂商平台注册并配置深链服务：

#### 4.1 华为 — App Linking（AGC）

- **服务介绍**：<https://developer.huawei.com/consumer/cn/agconnect/App-linking/>
- **Android SDK 集成指南**：<https://developer.huawei.com/consumer/cn/doc/AppGallery-connect-Guides/agc-applinking-android-integrationsdk-0000001371557869>
- **Codelab 教程**：<https://developer.huawei.com/consumer/en/codelab/AppLinking/index.html>

配置要点：

- 在 AGC 控制台创建项目并启用 App Linking
- 下载 `agconnect-services.json` 配置文件
- 集成 SDK：`implementation 'com.huawei.agconnect:agconnect-applinking:x.x.x'`
- 支持延迟深链和链接数据统计

#### 4.2 小米 — 统一链接服务（OneLink）

- **OneLink 操作指南**：<https://dev.mi.com/xiaomihyperos/documentation/detail?pId=1965>
- **应用接力 DeepLink 开发指南**：<https://dev.mi.com/xiaomihyperos/documentation/detail?pId=1891>
- **API 文档**：<https://dev.mi.com/xiaomihyperos/documentation/detail?pId=1880>

配置要点：

- 登录小米开放平台开发者站，在分发菜单进入统一链接服务
- 配置链接名称、兜底页链接、跳转 deeplink 等参数
- 支持 DeepLink 和 AppLink 两种方式

#### 4.3 OPPO — 统一链接（OneLink）

- **开发者社区**：<https://open.oppomobile.com/>
- **应用商店 Deeplink 协议**：使用 `market://details?id={pkg}` 格式

配置要点：

- 在 OPPO 开放平台注册应用
- 配置统一链接，支持数据统计（访问量、下载量等）
- 需判断 OPPO 应用市场包名（`com.oppo.market` 或 `com.heytap.market`）
- 调用时必须使用 `startActivityForResult`，requestCode > 0

#### 4.4 vivo — DeepLink

- **开发者文档中心**：<https://developers.vivo.com/doc/d/23807c559e844cbeb06049ee69e71833>
- **开放平台**：<https://dev.vivo.com.cn/>

配置要点：

- 支持 App Link 格式：`applink://com.yourapp.id:port/path?params`
- 在 AndroidManifest 中配置对应 intent-filter
- 在目标 Activity 中解析并处理链接参数

### 5. H5 中间页适配策略

无论选择哪种方案，H5 中间页（`/frames/download/index.html`）都需要增强以下能力：

```
用户点击邀请链接
        ↓
H5 页面加载
        ↓
├─ 检测 UA 识别设备厂商和浏览器环境
│
├─ 微信/QQ 内打开 → 显示"请在浏览器中打开"蒙层引导
│
├─ 普通浏览器 →
│   ├─ 尝试 URL Scheme / App Links / 厂商深链 唤起 App
│   ├─ 超时未唤起 → 根据厂商跳转对应应用商店：
│   │   ├─ 华为设备 → AppGallery
│   │   ├─ 小米设备 → 小米应用商店
│   │   ├─ OPPO 设备 → OPPO 应用商店
│   │   ├─ vivo 设备 → vivo 应用商店
│   │   └─ 其他设备 → APK 下载 / 通用应用商店
│   └─ 已唤起 → 结束
│
└─ iOS → Universal Links 自动处理 / 跳转 App Store
```

### 6. 自建剪贴板口令（兜底方案）

作为所有方案的兜底补充，可实现类似"淘口令"的机制：

1. 分享时生成口令（如 `#HK相框邀请#复制此消息打开好看相册App加入我的相框 token:abc123`）
2. App 启动时读取剪贴板，匹配口令格式
3. 解析出 token 后走正常加入流程
4. 成功消费后清除剪贴板内容

> 注意：Android 10+ 和 iOS 14+ 对后台读取剪贴板有限制，需在前台且用户有主动粘贴行为时读取。

### 7. 适配工作清单

| 序号 | 事项 | 优先级 | 备注 |
|---|---|---|---|
| 1 | 确定深链方案（第三方服务 or 自行适配） | P0 | 影响后续所有工作 |
| 2 | 若选第三方：完成 OpenInstall / Sharetrace 接入 | P0 | Android + iOS 双端 |
| 3 | H5 中间页增加 UA 检测与厂商应用商店跳转 | P0 | 详见第 5 节 |
| 4 | H5 中间页增加微信/QQ 蒙层引导 | P0 | 检测 UA 含 MicroMessenger / QQ |
| 5 | 实现剪贴板口令兜底方案 | P1 | 覆盖所有无法深链唤起的场景 |
| 6 | 若选自行适配：逐一接入华为/小米/OPPO/vivo | P1 | 按用户设备占比排优先级 |
| 7 | 各厂商场景联调测试 | P1 | 覆盖主流机型 + 微信/QQ/短信/浏览器 |

---

## 十、安全设计

推荐策略：

- token 长度：32 位随机字符串
- 有效期：24 小时
- 最大加入人数：可配置（如 5 人）

服务器校验：

```
- token 是否存在
- 是否过期
- 是否超过使用次数
```

---

## 十一、完整链路时序

```
Owner App
   │
   │ POST /device/invite
   ▼
Cloud
   │
   │ 返回 invite_url (https://92-cn.levect.com/frames/download/index.html?token=xxx)
   ▼
Owner 分享给家人
   │
   ▼
家人点击链接
   │
   ├─ 有App → 触发 Universal Link / App Link → 直接打开App
   └─ 无App → 在浏览器打开 H5 页面 → 引导下载安装App
                │
                ▼
              打开App
                │
                ▼
      ┌─────────┴─────────┐
      │     检查登录状态    │
      └─────────┬─────────┘
                │
   未登录 ──────┼─────── 已登录
      │         │
   跳转登录页     │
      │         │
   登录成功 ──────┘
                │
                ▼
      ┌─────────┴─────────┐
      │   请求后端校验Token  │
      └─────────┬─────────┘
                │
      已加入相框 ──┴─── 未加入相框
         │             │
         ▼             ▼
  弹窗提示"您已加入"   弹窗提示确认加入
                       │
                       ▼
                    点击"加入"
                       │
                       ▼
            /device/join?token=xxx
                       │
                       ▼
                   绑定成功
```

---

## 十二、功能收益

上线该功能后，相框产品将从：

```
单用户设备 → 家庭共享设备
```

带来的收益包括：

- 提升照片更新频率
- 提升设备活跃度
- 提升用户留存

该能力是智能相框产品的重要增长点。
