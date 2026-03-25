# Pad 端每日智能推荐照片方案（V2）

## 一、版本演进说明

### 1.1 V1 回顾

V1 方案采用 **"新照片优先 + 老照片顺序轮播"** 策略，解决了照片过多导致看不完的问题。核心逻辑：

- 7 天内新照片全量展示
- 老照片按时间倒序顺序轮播，每天取连续一段，补足至 100 张
- 服务端记录轮播位置，确保每张照片都有机会展示

**V1 的不足**：

| 问题 | 说明 |
|-----|------|
| 缺乏语义理解 | 不理解照片内容，无法按主题组织 |
| 机械式轮播 | 老照片按固定顺序轮播，缺少"惊喜感" |
| 无上下文感知 | 不考虑日期、季节、节日、相框地点等语境 |
| 忽视用户偏好 | 不参考点赞、观看等行为信号 |
| 无多样性保障 | 可能连续展示同一场景/人物的相似照片 |
| 缺少故事性 | 照片之间无叙事关联，像随机幻灯片 |
| 无隐私保护 | 未考虑数据传输给第三方服务时的隐私合规 |

### 1.2 V2 目标

**让相框像一位"懂你的家人"来挑选今天要看的照片。**

V1 解决了"看什么"（从 1000+ 张中选 100 张），V2 要解决"怎么看得更好"——选得**有意义**、排得**有故事**、看得**有惊喜**。

核心思路：将照片属性、今日上下文（含相框地点与天气）、用户偏好**脱敏后**喂给 AI 大模型，由 AI 直接输出最适合今天看的照片集合和展示顺序。开发者不写推荐算法，只组装数据和设计 Prompt。同时确保整个数据链路**隐私合规**，用户数据经脱敏处理后方可传给第三方 AI。

### 1.3 V1 vs V2 对比

| 能力维度 | V1 | V2 |
|---------|----|----|
| **选片逻辑** | 按上传时间分新/旧，顺序轮播 | **AI 大模型综合分析**，输出推荐列表 |
| **内容理解** | 无，所有照片同质对待 | AI 理解 ML Kit 标签、场景、人像、大区（**数据已脱敏**） |
| **时间感知** | 仅区分 7 天内/外 | AI 感知"历史今天"、季节、节日、相框地点、天气 |
| **用户偏好** | 不参考 | AI 参考点赞偏好、展示历史等行为信号 |
| **展示编排** | 新照片在前 + 老照片时间倒序 | AI 自动多主题交织排列 |
| **多样性** | 无保障 | AI 通过 Prompt 规则约束多样性 |
| **质量过滤** | 无 | 预过滤截图/文档 + AI 优选高质量横版照片 |
| **策略迭代** | 改代码 → 测试 → 部署 | **改 Prompt 即可，无需改代码** |
| **降级策略** | 无 | AI 异常自动回退 V1 轮播 |
| **隐私合规** | 无 | **数据脱敏传输 + 用户授权 + 服务商 DPA** |
| **客户端改动** | — | **零改动**，API 完全兼容（授权弹窗需 App 端配合） |

---

## 二、推荐系统架构

### 2.1 核心思路：让 AI 做决策

V1 和传统推荐系统需要开发者手写评分公式、调权重、写编排算法。V2 的核心转变是：**把照片数据和上下文喂给 AI 大模型，让 AI 直接输出推荐结果**。

开发者只需要做三件事：
1. **组织数据**——把照片属性、用户行为、今日上下文整理成结构化输入
2. **设计 Prompt**——告诉 AI 推荐目标和约束条件
3. **缓存结果**——将 AI 输出缓存，避免重复调用

### 2.2 整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                    服务端 (Java)                              │
│                                                             │
│  ① 数据准备                                                  │
│  ┌────────────┐  ┌────────────┐  ┌────────────────────────┐ │
│  │ 照片属性    │  │ 今日上下文  │  │ 用户行为               │ │
│  │ · ML标签   │  │ · 日期     │  │ · 点赞列表             │ │
│  │ · 场景分类  │  │ · 季节     │  │ · 展示历史             │ │
│  │ · 人脸     │  │ · 节日     │  │ · 上传者偏好           │ │
│  │ · 地点     │  │ · 周末     │  │                        │ │
│  │ · 拍摄时间  │  │ · 相框地点  │  │                        │ │
│  │ · 尺寸     │  │ · 天气     │  │                        │ │
│  └─────┬──────┘  └─────┬──────┘  └───────────┬────────────┘ │
│        │               │                     │              │
│        └───────────────┼─────────────────────┘              │
│                        ▼                                    │
│  ② 构建 Prompt + 调用 AI                                     │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                  AI 大模型 API                         │   │
│  │          (DeepSeek / Qwen / GPT / ...)                │   │
│  │                                                      │   │
│  │  输入: 脱敏照片元数据 + 上下文 + 推荐规则 Prompt        │   │
│  │  输出: 推荐照片 ID 有序列表 (JSON)                     │   │
│  └──────────────────────┬───────────────────────────────┘   │
│                         ▼                                   │
│  ③ 缓存 + 返回                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Redis 缓存当日结果 → 返回给 Pad 相框                  │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 2.3 AI 模型选型建议

| 模型 | 优势 | 数据存储 | 合规风险 | 适用场景 |
|-----|------|---------|---------|---------|
| **DeepSeek** | 性价比极高，中文理解优秀，支持长上下文 | 国内 | 较低，需签 DPA | **首选**，适合大量照片元数据输入 |
| **通义千问 (Qwen)** | 阿里云生态集成方便，国内访问稳定 | 国内（阿里云） | 低，合规体系完善 | 已使用阿里云的项目 |
| **GLM (智谱)** | 国内厂商，数据合规无忧 | 国内 | **最低** | 对数据安全要求最高的场景 |
| **GPT-4o-mini** | 推理能力强，成本适中 | **境外（美国）** | **高，涉及数据出境** | 仅作技术验证备选，不建议生产使用 |

**选型关键考量**：
- 输入为纯文本（照片元数据 JSON），不需要多模态能力
- 需要支持较长上下文（1000 张照片的元数据约 50-80K tokens）
- **优先选择国内模型**，降低网络延迟和合规风险
- **数据合规**：若使用境外模型，即使数据已脱敏，仍可能触发《数据安全法》数据出境安全评估要求，建议避免
- **数据处理协议（DPA）**：无论选择哪家模型，均需与服务商签订 DPA，确保不使用用户数据训练模型、调用完即销毁

---

## 三、喂给 AI 的数据

AI 不需要看到照片本身，只需要结构化的元数据。服务端负责将照片属性、用户行为、今日上下文组装成 JSON，作为 AI 的输入。

### 3.1 照片元数据（每张照片）

从项目已有的数据模型中提取，组装为精简的 JSON 对象。**传给 AI 的数据经过脱敏处理**（详见第十章"隐私合规与数据安全"），使用临时索引号替代真实 ID，敏感字段做模糊化处理：

```json
{
  "idx": 1,
  "uploadTime": "2026-02-28",
  "shootTime": "2026-02-15",
  "labels": ["Person", "Portrait", "Indoor"],
  "scene": "Person",
  "hasPortrait": true,
  "region": "华东",
  "width": 1920,
  "height": 1080,
  "isLiked": true,
  "likeCount": 3,
  "author": "A",
  "displayCount": 2,
  "lastDisplayDate": "2026-02-25",
  "clusterId": 5
}
```

| 字段 | 来源 | 传给 AI 的值 | 脱敏说明 |
|-----|------|-------------|---------|
| `idx` | PhotoItem.id | 临时索引号（1, 2, 3...） | 不传真实 ID，服务端维护 idx→realId 映射表 |
| `uploadTime` | PhotoEntity.dateAdded | 原值 | 日期无敏感性 |
| `shootTime` | SelectImgBean.dateTime | 原值 | 日期无敏感性 |
| `labels` | PhotoEntity.labels | 原值 | ML Kit 通用标签，无个人信息 |
| `scene` | SceneClusterStrategy | 原值 | 场景分类主标签 |
| `hasPortrait` | FaceDetectorWrapper | 布尔值 | 原字段 `hasFace` 模糊化为"是否含人像"，不传人脸特征数据 |
| `region` | SelectImgBean.city/province | 模糊化为大区（华东/华南/...） | 精确城市信息不传给 AI，仅服务端用于本地回忆匹配 |
| `width` / `height` | PhotoEntity | 原值 | 分辨率无敏感性 |
| `isLiked` / `likeCount` | Pad PhotoItem | 原值 | 行为数据，已与用户身份解耦 |
| `author` | PhotoItem.authorName | 代号（A, B, C...） | 不传真实姓名/称呼，服务端维护代号映射 |
| `displayCount` | 展示记录 | 原值 | 统计数据 |
| `lastDisplayDate` | 展示记录 | 原值 | 日期无敏感性 |
| `clusterId` | PhotoEntity.clusterId | 原值 | 聚类编号，无敏感性 |

> **脱敏原则**：传给第三方 AI 的数据遵循**最小必要 + 不可逆向识别**原则。AI 只需要知道"这张照片是人像、拍于华东地区、作者 A 上传"就足以做出推荐决策，无需知道具体城市或真实姓名。服务端在收到 AI 返回的 `idx` 列表后，再映射回真实照片 ID。

### 3.2 预过滤（减少 Token 消耗）

在喂给 AI 之前，服务端先做简单过滤，减少输入数据量：

```
过滤规则：
  1. 排除 isScreenshot = true 的照片
  2. 排除 isDocument = true 的照片
  3. 排除 isQrCode = true 的照片
  4. 同一 clusterId 下最多保留 5 张（按质量排序）
  5. 若过滤后仍超过 500 张，按上传时间取最近 500 张
```

> 500 张照片的元数据约 30-50K tokens，在主流大模型的上下文窗口范围内。

### 3.3 今日上下文

```json
{
  "today": "2026-03-03",
  "weekday": "周二",
  "season": "春季",
  "holiday": null,
  "frameCity": "杭州",
  "frameProvince": "浙江",
  "weather": "晴，15℃",
  "totalPhotos": 1200,
  "targetCount": 100
}
```

| 字段 | 来源 | 说明 |
|-----|------|------|
| `today` | 系统时间 | 当前日期 |
| `weekday` | 系统时间 | 星期几 |
| `season` | 月份推算 | 当前季节（按相框所在城市气候区修正） |
| `holiday` | 节日库 | 当日节日（含全国性节日 + 相框所在地区域性节日） |
| `frameCity` | 设备绑定信息 | 相框所在城市，用于本地化推荐 |
| `frameProvince` | 设备绑定信息 | 相框所在省份 |
| `weather` | 天气 API | 相框所在城市当日天气概况（可选，降级时不传） |
| `totalPhotos` | DB 统计 | 用户照片总数 |
| `targetCount` | 固定值 | 本次需要推荐的照片数量 |

> **相框地点的作用**：① 结合照片拍摄地做"本地回忆"推荐（如在杭州的相框优先推杭州拍的照片）；② 修正季节判断（南北方差异大，哈尔滨 3 月仍为冬季）；③ 触发区域性节日推荐（如泼水节、那达慕）；④ 天气联动调整推荐风格（雨天推温馨室内照，晴天推户外风景照）。

### 3.4 用户行为摘要

```json
{
  "likedLabels": {"Person": 12, "Landscape": 8, "Food": 5, "Pet": 3},
  "likedAuthors": {"A": 15, "B": 8, "C": 5},
  "recentlyShownIdxs": [1, 5, 12, ...]
}
```

> - 不传完整展示历史，只传聚合后的偏好摘要和近期已展示列表，控制 Token 用量。
> - `likedAuthors` 使用与 3.1 节一致的作者代号（A/B/C），不传真实姓名。
> - `recentlyShownIdxs` 使用临时索引号，与 3.1 节的 `idx` 对应。

---

## 四、AI Prompt 设计

### 4.1 Prompt 结构

```
┌──────────────────────────────────────────┐
│  System Prompt（固定，定义角色和规则）      │
│  ┌────────────────────────────────────┐  │
│  │ 你是一个智能相框的照片推荐助手...     │  │
│  │ 推荐规则：                          │  │
│  │  · 新照片优先                       │  │
│  │  · 历史今天回忆                     │  │
│  │  · 多主题交织                       │  │
│  │  · 多样性约束                       │  │
│  │  · 覆盖公平性                       │  │
│  │  · 输出格式                         │  │
│  └────────────────────────────────────┘  │
│                                          │
│  User Prompt（每次请求动态生成）           │
│  ┌────────────────────────────────────┐  │
│  │ 今日上下文 JSON                     │  │
│  │ 用户行为摘要 JSON                   │  │
│  │ 照片列表 JSON                       │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

### 4.2 System Prompt

```text
你是一个智能相框的照片推荐助手。你的任务是从用户的照片库中，挑选出最适合今天展示的照片集合。

## 数据说明
- 照片数据已做脱敏处理：idx 为临时索引号（非真实 ID），author 为代号（非真实姓名），region 为模糊大区（非精确位置）
- 你只需基于这些脱敏后的属性做推荐决策，无需推测原始信息

## 相框播放行为
- 相框每小时刷新一次照片列表，每 10 秒自动切换下一张
- 用户可能从列表的任意位置开始观看，不一定从头开始
- 因此你需要让不同主题的照片均匀分布在列表中，确保任意位置开始看都内容丰富

## 推荐规则（按优先级排列）

1. **新照片优先**：近 3 天内上传的照片必须入选，放在列表靠前位置
2. **历史今天**：往年今天（同月同日）拍摄的照片优先入选，唤起回忆
3. **用户偏好**：用户点赞多的标签类型（如人物、风景）适当增加比例
4. **内容多样**：确保结果覆盖多种场景（人物、风景、美食、宠物、旅行等），避免同类照片扎堆
5. **多主题交织**：不同主题的照片以 3~5 张为一小组交替排列，不要把同类照片连续排超过 5 张
6. **去重去相似**：同一 clusterId 的照片最多选 3 张，且不要连续排列
7. **覆盖公平**：优先选择 displayCount 低的照片，让每张照片都有机会展示
8. **质量过滤**：优先横版照片（width > height），优先高分辨率照片
9. **季节与天气关联**：根据上下文中的季节和天气信息调整推荐风格（如春天多推户外/花卉，雨天多推温馨室内照）
10. **地点亲和**：如果上下文中提供了相框所在大区（frameRegion），则与照片 region 相同大区的照片适当提升权重，营造"本地回忆"氛围

## 输出要求
- 从候选照片中选出 {targetCount} 张
- 输出为 JSON 数组，只包含照片 idx，按推荐展示顺序排列
- 前 3 个位置必须是最新上传的照片
- 不要输出任何解释，只输出 JSON

## 输出格式
```json
[1, 35, 42, ...]
```
```

### 4.3 User Prompt 模板

```text
## 今日上下文
{contextJson}

## 用户偏好
{preferenceJson}

## 候选照片列表（共 {count} 张）
{photosJson}

请从以上候选照片中选出 {targetCount} 张，按推荐展示顺序输出 ID 数组。
```

### 4.4 AI 输出解析

```
AI 返回示例：
[1, 35, 42, 8, 99, 23, 67, ...]

解析规则：
  1. 提取 JSON 数组
  2. 校验每个 idx 是否存在于候选照片的临时索引映射中（过滤无效值）
  3. 去重
  4. 将 idx 映射回真实照片 ID（通过服务端维护的 idx→realId 映射表）
  5. 若数量不足 targetCount，从剩余候选照片中按 uploadTime 倒序补足
  6. 若数量超出 targetCount，截取前 targetCount 个
```

---

## 五、相框播放行为与刷新机制

### 5.1 相框实际播放行为

```
刷新周期：每小时请求一次服务端，获取最新照片列表
播放方式：收到列表后，每 10 秒自动切换下一张，100 张约 17 分钟一轮
用户切入：用户可能在任意位置开始观看（不一定从第 1 张开始）
循环播放：播放到末尾后从头循环，直到下一次整点刷新
```

### 5.2 刷新机制

```
                    每小时刷新周期
  ┌──────────┬──────────┬──────────┬──────────┐
  │  8:00    │  9:00    │  10:00   │  11:00   │  ...
  │  刷新     │  刷新     │  刷新     │  刷新    │
  └────┬─────┴────┬─────┴────┬─────┴────┬─────┘
       │          │          │          │
       ▼          ▼          ▼          ▼
    返回当日      同一天内     有新照片     无变化
    推荐列表      返回缓存     更新缓存     返回缓存
```

| 场景 | 触发方式 | 处理方式 |
|-----|---------|---------|
| **当日首次请求** | 整点刷新 | 完整计算推荐列表，写入 Redis 缓存 |
| **同日后续请求（无新照片）** | 整点刷新 | 直接返回缓存结果 |
| **新照片上传** | **Push 立即触发** | 新照片插入缓存列表头部，推送相框立即刷新（秒级生效） |
| **整点刷新（有新照片已插入）** | 整点刷新 | 返回已更新的缓存列表 |
| **次日首次请求** | 整点刷新 | 旧缓存过期，重新完整计算 |

**新照片优先展示的完整链路**：

```
用户手机上传照片
      │
      ▼
服务端保存照片 → 插入 Redis 缓存列表头部 → Push 通知相框
                                                │
                                                ▼
                                  相框收到推送，立即请求照片列表
                                                │
                                                ▼
                                  服务端返回更新后的列表（新照片在最前）
                                                │
                                                ▼
                                  相框从列表头部开始播放，新照片立即展示
```

**为什么不每小时重新计算？**
- 推荐结果在一天内保持稳定，避免用户每小时看到完全不同的照片排列
- 唯一的动态变化是"新照片上传"，通过 Push + 增量插入实现秒级生效
- 整点刷新仅作为兜底机制（防止推送丢失）
- 减少服务端计算压力

---

## 六、服务端 Java 实现

AI 驱动后，服务端代码大幅简化。不再需要评分服务、编排服务等复杂模块，核心工作变为：**组装数据 → 调 AI → 缓存结果**。

### 6.1 数据模型

```java
/**
 * 喂给 AI 的照片脱敏属性（不含真实 ID、真实地点、真实姓名）
 */
@Data
public class PhotoMetaForAI {
    private Integer idx;           // 临时索引号，非真实 ID
    private String uploadTime;
    private String shootTime;
    private List<String> labels;
    private String scene;
    private Boolean hasPortrait;   // 是否含人像（模糊化，不暴露人脸检测细节）
    private String region;         // 大区（华东/华南/...），非精确城市
    private Integer width;
    private Integer height;
    private Boolean isLiked;
    private Integer likeCount;
    private String author;         // 代号（A/B/C），非真实姓名
    private Integer displayCount;
    private String lastDisplayDate;
    private Integer clusterId;
}

/**
 * 今日上下文（含相框地点信息）
 */
@Data
public class DailyContext {
    private String today;
    private String weekday;
    private String season;
    private String holiday;
    private String frameCity;      // 相框所在城市
    private String frameProvince;  // 相框所在省份
    private String weather;        // 相框所在城市天气（可选）
    private Integer totalPhotos;
    private Integer targetCount;
}

/**
 * 用户偏好摘要（使用脱敏代号）
 */
@Data
public class UserPreferenceSummary {
    private Map<String, Integer> likedLabels;
    private Map<String, Integer> likedAuthors;  // key 为作者代号（A/B/C）
    private List<Integer> recentlyShownIdxs;    // 临时索引号列表
}

/**
 * idx ↔ 真实照片 ID 的映射关系（仅服务端持有，不传给 AI）
 */
@Data
public class PhotoIdxMapping {
    private Map<Integer, Long> idxToRealId;     // idx → 真实照片 ID
    private Map<String, String> authorAliasMap;  // 代号 → 真实作者名
}
```

### 6.2 AI 推荐服务（核心，含脱敏处理）

```java
@Service
public class AIRecommendService {

    @Value("${ai.api.url}")
    private String aiApiUrl;

    @Value("${ai.api.key}")
    private String aiApiKey;

    @Value("${ai.model:deepseek-chat}")
    private String aiModel;

    @Autowired
    private ObjectMapper objectMapper;

    // System Prompt 与 4.2 节保持一致（含数据脱敏说明、地点亲和规则等）
    private static final String SYSTEM_PROMPT = """
        你是一个智能相框的照片推荐助手。你的任务是从用户的照片库中，
        挑选出最适合今天展示的照片集合。
        
        ## 数据说明
        - 照片数据已做脱敏处理：idx 为临时索引号，author 为代号，region 为模糊大区
        - 你只需基于这些脱敏后的属性做推荐决策，无需推测原始信息
        
        ## 相框播放行为
        - 相框每小时刷新一次，每 10 秒自动切换下一张
        - 用户可能从任意位置开始观看
        - 不同主题的照片需均匀分布，确保任意位置开始看都内容丰富
        
        ## 推荐规则（按优先级）
        1. 近 3 天内上传的照片必须入选，放在靠前位置
        2. 往年今天（同月同日）拍摄的照片优先入选
        3. 用户点赞多的标签类型适当增加比例
        4. 确保结果覆盖多种场景，避免同类照片扎堆
        5. 不同主题以 3~5 张为一组交替排列，同类不超过 5 张连续
        6. 同一 clusterId 最多选 3 张，且不连续排列
        7. 优先选 displayCount 低的照片
        8. 优先横版照片和高分辨率照片
        9. 根据季节和天气调整推荐风格
        10. 与相框所在大区相同 region 的照片适当提升权重
        
        ## 输出要求
        - 选出指定数量的照片
        - 输出 JSON 数组，只包含照片 idx，按推荐顺序排列
        - 前 3 个位置必须是最新上传的照片
        - 只输出 JSON，不要解释
        """;

    /**
     * 调用 AI 获取推荐结果（输入为脱敏数据，输出 idx 列表需映射回真实 ID）
     */
    public List<Long> getRecommendation(
            List<PhotoMetaForAI> desensitizedPhotos,
            PhotoIdxMapping mapping,
            DailyContext context,
            UserPreferenceSummary preference) throws Exception {

        String userPrompt = buildUserPrompt(desensitizedPhotos, context, preference);

        Map<String, Object> request = Map.of(
            "model", aiModel,
            "temperature", 0.3,
            "messages", List.of(
                Map.of("role", "system", "content", SYSTEM_PROMPT),
                Map.of("role", "user", "content", userPrompt)
            )
        );

        String response = callAiApi(request);
        return parseAndMapPhotoIds(response, mapping, desensitizedPhotos);
    }

    private String buildUserPrompt(
            List<PhotoMetaForAI> photos,
            DailyContext context,
            UserPreferenceSummary preference) throws Exception {

        return String.format("""
            ## 今日上下文
            %s
            
            ## 用户偏好
            %s
            
            ## 候选照片列表（共 %d 张）
            %s
            
            请从以上候选照片中选出 %d 张，按推荐展示顺序输出 idx 数组。
            """,
            objectMapper.writeValueAsString(context),
            objectMapper.writeValueAsString(preference),
            photos.size(),
            objectMapper.writeValueAsString(photos),
            context.getTargetCount()
        );
    }

    /**
     * 解析 AI 返回的 idx 列表 → 映射为真实照片 ID → 校验并补足
     */
    private List<Long> parseAndMapPhotoIds(
            String aiResponse, PhotoIdxMapping mapping, List<PhotoMetaForAI> candidates) {

        Set<Integer> validIdxs = candidates.stream()
            .map(PhotoMetaForAI::getIdx)
            .collect(Collectors.toSet());

        List<Integer> idxs = objectMapper.readValue(aiResponse, new TypeReference<>() {});

        List<Long> result = idxs.stream()
            .filter(validIdxs::contains)
            .distinct()
            .map(idx -> mapping.getIdxToRealId().get(idx))
            .filter(Objects::nonNull)
            .collect(Collectors.toList());

        if (result.size() < 100) {
            Set<Long> selected = new HashSet<>(result);
            mapping.getIdxToRealId().values().stream()
                .filter(id -> !selected.contains(id))
                .limit(100 - result.size())
                .forEach(result::add);
        }

        return result;
    }
}
```

### 6.3 数据准备服务（含脱敏处理）

```java
@Service
public class RecommendDataService {

    @Autowired
    private PhotoRepository photoRepository;
    @Autowired
    private DisplayRecordService displayRecordService;
    @Autowired
    private DeviceService deviceService;
    @Autowired
    private WeatherService weatherService;

    private static final int MAX_CANDIDATES = 500;
    private static final int MAX_PER_CLUSTER = 5;

    private static final Map<String, String> PROVINCE_TO_REGION = Map.ofEntries(
        Map.entry("浙江", "华东"), Map.entry("江苏", "华东"), Map.entry("上海", "华东"),
        Map.entry("广东", "华南"), Map.entry("广西", "华南"), Map.entry("海南", "华南"),
        Map.entry("北京", "华北"), Map.entry("天津", "华北"), Map.entry("河北", "华北"),
        Map.entry("湖北", "华中"), Map.entry("湖南", "华中"), Map.entry("河南", "华中"),
        Map.entry("四川", "西南"), Map.entry("重庆", "西南"), Map.entry("云南", "西南"),
        Map.entry("陕西", "西北"), Map.entry("甘肃", "西北"), Map.entry("新疆", "西北"),
        Map.entry("黑龙江", "东北"), Map.entry("吉林", "东北"), Map.entry("辽宁", "东北")
        // ... 其余省份
    );

    /**
     * 准备喂给 AI 的脱敏数据 + 映射表
     */
    public DesensitizedResult prepareCandidates(Long userId) {
        List<PhotoRecommendContext> allPhotos = photoRepository
            .findAllWithRecommendContext(userId);

        List<PhotoRecommendContext> filtered = allPhotos.stream()
            .filter(p -> !Boolean.TRUE.equals(p.getIsScreenshot()))
            .filter(p -> !Boolean.TRUE.equals(p.getIsDocument()))
            .filter(p -> !Boolean.TRUE.equals(p.getIsQrCode()))
            .collect(Collectors.groupingBy(
                p -> p.getClusterId() != null ? p.getClusterId() : -p.getPhotoId().intValue()))
            .values().stream()
            .flatMap(group -> group.stream().limit(MAX_PER_CLUSTER))
            .sorted(Comparator.comparing(PhotoRecommendContext::getUploadTime).reversed())
            .limit(MAX_CANDIDATES)
            .collect(Collectors.toList());

        // 构建脱敏映射
        Map<Integer, Long> idxToRealId = new HashMap<>();
        Map<String, String> authorAliasMap = new HashMap<>();
        AtomicInteger idxCounter = new AtomicInteger(1);
        AtomicInteger authorCounter = new AtomicInteger(0);

        List<PhotoMetaForAI> desensitized = filtered.stream().map(p -> {
            int idx = idxCounter.getAndIncrement();
            idxToRealId.put(idx, p.getPhotoId());

            String authorAlias = authorAliasMap.computeIfAbsent(
                p.getAuthorName(),
                name -> String.valueOf((char) ('A' + authorCounter.getAndIncrement()))
            );

            PhotoMetaForAI meta = new PhotoMetaForAI();
            meta.setIdx(idx);
            meta.setUploadTime(p.getUploadTime().toString());
            meta.setShootTime(p.getShootTime() != null ? p.getShootTime().toString() : null);
            meta.setLabels(p.getLabels());
            meta.setScene(p.getScene());
            meta.setHasPortrait(p.getHasFace());
            meta.setRegion(PROVINCE_TO_REGION.getOrDefault(p.getProvince(), "其他"));
            meta.setWidth(p.getWidth());
            meta.setHeight(p.getHeight());
            meta.setIsLiked(p.getIsLiked());
            meta.setLikeCount(p.getLikeCount());
            meta.setAuthor(authorAlias);
            meta.setDisplayCount(p.getDisplayCount());
            meta.setLastDisplayDate(p.getLastDisplayDate() != null ? p.getLastDisplayDate().toString() : null);
            meta.setClusterId(p.getClusterId());
            return meta;
        }).collect(Collectors.toList());

        PhotoIdxMapping mapping = new PhotoIdxMapping();
        mapping.setIdxToRealId(idxToRealId);
        mapping.setAuthorAliasMap(authorAliasMap);

        return new DesensitizedResult(desensitized, mapping);
    }

    public DailyContext buildContext(Long userId, int totalPhotos) {
        LocalDate today = LocalDate.now();
        DeviceInfo device = deviceService.getFrameDevice(userId);

        DailyContext ctx = new DailyContext();
        ctx.setToday(today.toString());
        ctx.setWeekday(today.getDayOfWeek().getDisplayName(TextStyle.SHORT, Locale.CHINESE));
        ctx.setFrameCity(device.getCity());
        ctx.setFrameProvince(device.getProvince());
        ctx.setSeason(getSeason(today.getMonthValue(), device.getProvince()));
        ctx.setHoliday(detectHoliday(today, device.getProvince()));
        ctx.setTotalPhotos(totalPhotos);
        ctx.setTargetCount(100);

        try {
            ctx.setWeather(weatherService.getTodayBrief(device.getCity()));
        } catch (Exception e) {
            // 天气获取失败不影响推荐
        }

        return ctx;
    }

    public UserPreferenceSummary buildPreference(Long userId, Map<String, String> authorAliasMap) {
        UserPreferenceSummary pref = new UserPreferenceSummary();
        pref.setLikedLabels(aggregateLikedLabels(userId));

        Map<String, Integer> rawAuthorStats = aggregateLikedAuthors(userId);
        Map<String, Integer> aliasedStats = rawAuthorStats.entrySet().stream()
            .collect(Collectors.toMap(
                e -> authorAliasMap.getOrDefault(e.getKey(), "?"),
                Map.Entry::getValue
            ));
        pref.setLikedAuthors(aliasedStats);

        // recentlyShownIdxs 需在调用方根据映射表转换
        return pref;
    }
}
```

### 6.4 每日推荐引擎（入口）

```java
@Service
public class DailyRecommendationEngine {

    @Autowired
    private RecommendDataService dataService;
    @Autowired
    private AIRecommendService aiService;
    @Autowired
    private DailyPhotoSelectionService v1Service;
    @Autowired
    private StringRedisTemplate redisTemplate;

    private static final int DAILY_TARGET = 100;

    public List<Long> getDailyRecommendation(Long userId) {
        String today = LocalDate.now().toString();
        String cacheKey = "recommend:daily:" + userId + ":" + today;

        // 1. 尝试读取缓存
        String cached = redisTemplate.opsForValue().get(cacheKey);
        if (cached != null) {
            return parseCachedIds(cached);
        }

        // 2. 准备数据
        List<PhotoMetaForAI> candidates = dataService.prepareCandidates(userId);
        if (candidates.size() <= DAILY_TARGET) {
            return candidates.stream().map(PhotoMetaForAI::getId).collect(Collectors.toList());
        }

        // 3. 调用 AI 推荐（失败时降级 V1）
        List<Long> result;
        try {
            DailyContext context = dataService.buildContext(candidates.size());
            UserPreferenceSummary preference = dataService.buildPreference(userId);
            result = aiService.getRecommendation(candidates, context, preference);
        } catch (Exception e) {
            log.warn("AI 推荐失败，降级为 V1 轮播策略", e);
            return v1Service.getDailyPhotoIds(userId);
        }

        // 4. 缓存至次日凌晨
        long ttl = Duration.between(
            LocalDateTime.now(),
            LocalDate.now().plusDays(1).atStartOfDay()
        ).getSeconds();
        redisTemplate.opsForValue().set(cacheKey, joinIds(result), ttl, TimeUnit.SECONDS);

        return result;
    }
}
```

### 6.5 API 接口（兼容 V1）

```java
@RestController
@RequestMapping("/api/photo")
public class PhotoController {

    @Autowired
    private DailyRecommendationEngine recommendEngine;
    @Autowired
    private PhotoRepository photoRepository;

    /**
     * 接口路径和参数与 V1 完全一致，客户端（Pad）无需改动
     */
    @PostMapping("/list")
    public ApiResponse<PhotoListResult> getPhotoList(@RequestBody PhotoListRequest request) {
        Long userId = request.getUserId();

        List<Long> recommendedIds = recommendEngine.getDailyRecommendation(userId);

        List<PhotoItem> dailyPhotos = photoRepository.findByIdsInOrder(recommendedIds);

        PhotoListResult result = new PhotoListResult();
        result.setResult(dailyPhotos);
        return ApiResponse.success(result);
    }
}
```

### 6.6 新照片实时处理

```java
@Service
public class PhotoUploadService {

    @Autowired
    private StringRedisTemplate redisTemplate;

    public void onPhotoUploaded(Long userId, Long photoId) {
        photoRepository.save(newPhoto);

        // 将新照片插入缓存列表头部
        String today = LocalDate.now().toString();
        String cacheKey = "recommend:daily:" + userId + ":" + today;
        String cached = redisTemplate.opsForValue().get(cacheKey);
        if (cached != null) {
            redisTemplate.opsForValue().set(cacheKey, photoId + "," + cached);
        }

        // Push 通知相框立即刷新
        pushService.sendToFrame(userId, PushMessage.NEW_PHOTO, photoId);
    }
}
```

---

## 七、缓存与存储设计

### 7.1 Redis Key 设计

| Key 格式 | Value 格式 | 用途 | 过期策略 |
|----------|-----------|------|---------|
| `recommend:daily:{userId}:{date}` | `id1,id2,id3,...` | 每日 AI 推荐结果缓存 | 次日凌晨过期 |
| `display:stats:{userId}` | Hash: `{photoId}` → `{count}:{lastTime}` | 展示统计 | 永不过期 |
| `carousel:position:{userId}` | `{position}:{date}` | V1 轮播位置（保留兼容） | 永不过期 |

### 7.2 缓存策略

```
每日首次请求 → 调用 AI → 缓存结果 → 当日后续请求直接返回缓存
新照片上传   → 插入缓存头部 → Push 相框 → 秒级生效
次日首次请求 → 旧缓存过期 → 重新调用 AI
```

---

## 八、V1 → V2 平滑迁移

### 8.1 灰度策略

```java
@Service
public class RecommendStrategyRouter {

    @Autowired
    private DailyPhotoSelectionService v1Service;
    @Autowired
    private DailyRecommendationEngine v2Engine;
    @Autowired
    private FeatureFlagService featureFlagService;

    public List<Long> getRecommendedPhotoIds(Long userId) {
        if (featureFlagService.isEnabled("recommend_v2", userId)) {
            return v2Engine.getDailyRecommendation(userId);
        }
        return v1Service.getDailyPhotoIds(userId);
    }
}
```

### 8.2 降级机制

| 异常场景 | 降级策略 |
|---------|---------|
| AI API 超时（> 10s） | 自动回退 V1 轮播 |
| AI API 返回格式异常 | 自动回退 V1 轮播 |
| AI API 余额不足 | 自动回退 V1 轮播 |
| 照片总量 ≤ 100 张 | 跳过 AI，直接返回全部 |

---

## 九、成本与性能

### 9.1 AI 调用成本估算

| 项目 | 估算 |
|-----|------|
| 单次输入 Token | 500 张照片 × ~100 tokens/张 ≈ **50K tokens** |
| 单次输出 Token | 100 个 ID ≈ **500 tokens** |
| 每用户每日调用 | **1 次**（缓存命中后不再调用） |
| DeepSeek 单价 | 输入 ¥1/M tokens，输出 ¥2/M tokens |
| **单用户日成本** | ≈ **¥0.05 + ¥0.001 ≈ ¥0.05** |
| 1000 用户/月 | ≈ **¥1,500/月** |

> 成本极低，且随用户量线性增长。可通过调整预过滤阈值（减少候选数）进一步降低。

### 9.2 延迟分析

| 步骤 | 预估耗时 |
|-----|---------|
| 数据准备（DB 查询 + 预过滤 + JSON 序列化） | 100-300ms |
| AI API 调用（DeepSeek） | 2-5s |
| 结果解析 + 校验 | 10ms |
| **总计（首次请求）** | **2-6s** |
| **缓存命中** | **< 50ms** |

> 首次请求耗时较长（2-6s），但一天只发生一次。后续所有请求均命中缓存，响应 < 50ms。

### 9.3 优化手段

```
1. 凌晨预计算：定时任务为活跃用户预调用 AI，用户首次请求直接命中缓存
2. 异步调用：首次请求先返回 V1 结果，后台异步调用 AI 更新缓存，下次刷新生效
3. 减少 Token：精简 JSON 字段名（如 id→i, labels→l），减少 30-40% Token
4. 模型选择：使用更快的小模型（如 DeepSeek-V3-0324）降低延迟
```

---

## 十、隐私合规与数据安全

本方案涉及将用户照片元数据传给第三方 AI 服务处理，属于《个人信息保护法》规定的"个人信息处理"行为，需严格遵守相关法律法规。

### 10.1 涉及的法律法规

| 法律法规 | 核心要求 | 与本方案的关联 |
|---------|---------|--------------|
| **《个人信息保护法》(PIPL)** | 处理敏感个人信息需取得单独同意；遵循最小必要原则 | 人脸检测结果、地理位置、家庭成员关系均属敏感/个人信息 |
| **《数据安全法》** | 数据分类分级保护；数据出境安全评估 | 若使用境外模型（如 GPT），需进行数据出境评估 |
| **《GB/T 35273 个人信息安全规范》** | 收集、存储、传输个人信息的技术规范 | 传输给 AI 的数据需加密，不留存 |
| **《生成式人工智能服务管理暂行办法》** | 使用生成式 AI 服务的合规要求 | 本方案使用大模型推荐，需遵守 |
| **GDPR**（如涉及海外市场） | 数据最小化、存储限制、处理合法性 | 需根据目标市场评估 |

### 10.2 数据分类与风险等级

| 数据类型 | 原始字段 | 敏感等级 | 本方案处理方式 |
|---------|---------|---------|--------------|
| **人脸/生物识别** | hasFace、人脸特征 | 🔴 极高 | 模糊化为 `hasPortrait`（布尔值），**不传任何人脸特征数据** |
| **精确位置** | city、province | 🔴 高 | 模糊化为大区 `region`（华东/华南/...），精确位置仅服务端使用 |
| **家庭成员姓名** | authorName（妈妈/爸爸） | 🟠 中高 | 替换为代号 `author`（A/B/C），映射表仅服务端持有 |
| **用户行为画像** | likedLabels、likedAuthors | 🟠 中 | 传聚合统计值（非明细），作者名已代号化 |
| **照片真实 ID** | photoId | 🟡 低 | 替换为临时索引 `idx`（1/2/3...），映射表仅服务端持有 |
| **拍摄/上传时间** | shootTime、uploadTime | 🟢 低 | 原值传递（日期信息敏感性低，且为推荐核心依据） |
| **照片尺寸** | width、height | 🟢 低 | 原值传递 |

### 10.3 数据脱敏策略总结

```
原始数据（服务端）                    脱敏后数据（传给 AI）
┌──────────────────────┐          ┌──────────────────────┐
│ id: 1001             │    →     │ idx: 1               │
│ hasFace: true        │    →     │ hasPortrait: true     │
│ city: "杭州"         │    →     │ region: "华东"        │
│ province: "浙江"     │    →     │ (省略)               │
│ authorName: "妈妈"   │    →     │ author: "A"          │
│ ...其他字段...        │    →     │ ...原值或脱敏值...     │
└──────────────────────┘          └──────────────────────┘
                                           │
                                           ▼
                                    AI 返回 idx 列表
                                    [1, 35, 42, ...]
                                           │
                                           ▼
                              服务端通过映射表还原真实 ID
                              [1001, 2035, 3042, ...]
```

**核心原则**：传给第三方 AI 的数据**不可逆向识别具体个人**。即使 AI 服务商数据泄露，攻击者也无法从 `idx: 1, author: A, region: 华东` 还原出具体用户身份。

### 10.4 用户知情同意

| 环节 | 要求 | 实现方式 |
|-----|------|---------|
| **首次启用** | 明确告知用户 AI 推荐功能的数据处理方式 | App 端弹窗说明 + 勾选同意 |
| **单独同意** | 涉及人像分析结果传输，需单独授权 | 独立于隐私政策的授权弹窗 |
| **可关闭** | 用户有权随时关闭 AI 推荐 | App 设置页提供开关，关闭后回退 V1 轮播 |
| **可撤回** | 用户可撤回授权并要求删除数据 | 提供"清除 AI 推荐数据"按钮 |
| **隐私政策更新** | 需在隐私政策中说明 AI 推荐的数据处理 | 更新 App 隐私政策文档 |

**知情同意文案示例**：

```
"智能推荐"功能说明

为了给您提供更贴心的照片推荐体验，我们会将照片的场景标签、
拍摄时间等脱敏后的元数据发送至 AI 服务进行分析。

我们承诺：
· 不会传输照片原图或任何可识别您身份的信息
· 照片地点已模糊化为大区（如"华东"），不传精确位置
· 家庭成员名称已替换为代号，AI 无法知道具体是谁
· AI 服务商不会使用您的数据训练模型
· 数据处理完成后即时销毁，不做任何留存

您可以随时在"设置 → 智能推荐"中关闭此功能。

[同意并开启]  [暂不开启]
```

### 10.5 第三方数据处理协议（DPA）

与 AI 服务商签订的数据处理协议需包含以下条款：

| 条款 | 要求 |
|-----|------|
| **数据用途限制** | 仅用于本次 API 调用的推理计算，不得用于其他目的 |
| **禁止模型训练** | 明确禁止使用调用数据训练或微调模型 |
| **即时销毁** | API 调用完成后，输入输出数据应立即销毁，不做任何形式的留存 |
| **数据存储位置** | 明确约定数据仅在中国境内处理和存储 |
| **安全措施** | 传输加密（TLS 1.2+）、访问控制、审计日志 |
| **违约责任** | 数据泄露时的通知义务和赔偿责任 |
| **审计权** | 我方有权对服务商的数据处理行为进行审计 |

### 10.6 技术安全措施

| 措施 | 说明 |
|-----|------|
| **传输加密** | 所有 AI API 调用使用 HTTPS（TLS 1.2+） |
| **API Key 管理** | AI API Key 存储在密钥管理服务（如阿里云 KMS），不硬编码 |
| **请求日志脱敏** | 服务端日志中不记录完整的 Prompt 内容，仅记录请求摘要 |
| **缓存加密** | Redis 中的推荐结果缓存仅存储照片 ID 列表，不存储元数据 |
| **访问控制** | AI 推荐接口需要有效的设备 Token 认证 |
| **异常监控** | 监控 AI API 调用异常（超时、格式错误、余额不足），及时告警 |

### 10.7 合规检查清单

在正式上线前需完成以下检查：

- [ ] 完成个人信息保护影响评估（PIA）
- [ ] 更新 App 隐私政策，增加 AI 推荐功能说明
- [ ] 实现用户知情同意弹窗及授权管理
- [ ] 与 AI 服务商签订数据处理协议（DPA）
- [ ] 确认选用的 AI 模型数据存储在中国境内
- [ ] 验证数据脱敏逻辑：传给 AI 的数据不含真实 ID、姓名、精确位置
- [ ] 传输链路全程 HTTPS 加密
- [ ] API Key 使用密钥管理服务，不硬编码
- [ ] 实现"关闭 AI 推荐"功能开关并测试回退 V1 逻辑
- [ ] 实现"清除 AI 推荐数据"功能
- [ ] 服务端日志中不记录完整 Prompt 内容

---

## 十一、效果度量

### 11.1 核心指标

| 指标 | 定义 | 目标 |
|-----|------|-----|
| **照片覆盖率** | 30 天内被展示过的照片 / 总照片数 | ≥ 80% |
| **点赞转化率** | 每日被点赞照片数 / 每日展示照片数 | 较 V1 提升 30% |
| **多样性指数** | 每日展示照片的场景类别数 / 总场景类别数 | ≥ 0.7 |
| **AI 成功率** | AI 正常返回 / 总调用次数 | ≥ 99% |

### 11.2 A/B 测试

```
实验组：V2 AI 推荐
对照组：V1 顺序轮播
分流：按 userId 哈希，各 50%
周期：14 天
核心指标：点赞转化率、照片覆盖率
```

---

## 十二、方案总结

### 12.1 V1 vs V2 对比

| 维度 | V1 | V2 |
|-----|----|----|
| 选片逻辑 | 时间分类 + 顺序轮播 | **AI 大模型智能推荐** |
| 内容理解 | 无 | AI 理解 ML 标签、场景、人像、大区（脱敏后） |
| 上下文感知 | 无 | AI 感知日期、季节、节日、相框地点、天气 |
| 用户偏好 | 无 | AI 参考点赞偏好和展示历史 |
| 编排策略 | 新照片在前 + 老照片顺序 | AI 自动多主题交织 |
| 开发成本 | 中（需写评分/编排算法） | **低（组装数据 + 写 Prompt）** |
| 可迭代性 | 改代码 + 重新部署 | **改 Prompt 即可，无需改代码** |
| 降级策略 | 无 | AI 异常自动回退 V1 |
| 隐私合规 | 无特殊处理 | **数据脱敏 + 用户授权 + DPA 协议** |
| 客户端改动 | — | **零改动**（AI 推荐授权弹窗需 App 端配合） |

### 12.2 服务端改动清单

| 改动点 | 类型 | 说明 |
|-------|------|------|
| `AIRecommendService` | **新增** | AI API 调用与结果解析（含 idx→ID 映射） |
| `RecommendDataService` | **新增** | 数据准备（预过滤 + 脱敏 + JSON 组装） |
| `PhotoIdxMapping` | **新增** | idx ↔ 真实 ID 映射关系 |
| `DailyRecommendationEngine` | **新增** | 推荐入口（缓存 + 降级） |
| `RecommendStrategyRouter` | **新增** | V1/V2 灰度路由 |
| `PhotoController` | 修改 | 接入策略路由 |
| `PhotoUploadService` | 修改 | 新照片实时插入缓存 |

> 相比自研算法方案（需 11 个类），AI 方案仅需 **5 个新类 + 2 个修改**，开发量减少约 50%。

### 12.3 核心优势：Prompt 即策略

```
传统方案：改推荐策略 = 改代码 → 测试 → 部署 → 生效
AI 方案：改推荐策略 = 改 Prompt → 立即生效

示例：
  想增加"宠物照片比例" → 在 Prompt 中加一条规则即可
  想支持"母亲节特别推荐" → 在上下文中传入节日信息，Prompt 已覆盖
  想调整"新照片优先级" → 修改 Prompt 中的优先级描述
```

---

## 十三、后续扩展

| 方向 | 说明 |
|-----|------|
| **AI 图说生成** | 在推荐的同时，让 AI 为每日照片集生成一段温馨文字，显示在相框上 |
| **用户反馈闭环** | 支持"不喜欢"操作，将反馈追加到下次 Prompt 中 |
| **多用户协同** | 家庭多成员偏好融合，在 Prompt 中描述各成员喜好 |
| **动态 Prompt 进化** | 基于 A/B 测试数据，自动优化 Prompt 措辞 |
| **合规自动化** | 接入合规检测平台，自动扫描传给 AI 的数据是否满足脱敏要求 |
