# 小豆泥合辑专题配置工具

用于配置小豆泥专区的合辑信息，支持主题配置和小组件配置，生成标准 JSON 数据供 Android App 使用。

## 功能特性

- 🎨 **两种合辑类型**：普通合辑（头图+背景色）、介绍合辑（介绍长图）
- 📱 **多平台主题配置**：支持小米、OPPO、vivo、华为的 Deeplink 和浏览器分享地址
- 🧩 **多小组件配置**：支持日历、时间、待办事项、电量四种组件类型
- 📦 **资源上传验证**：图片尺寸、文件大小自动校验
- 🎯 **模拟数据**：一键填充测试数据，方便调试
- 📋 **JSON 导出**：带字段说明的 JSON 输出，支持一键复制

## 使用方法

1. 打开 `collection-config.html` 文件
2. 点击「配置专区信息」按钮
3. 选择所属专区
4. 配置合辑类型和相关信息
5. 点击「保存」生成 JSON

## 资源限制

### 头图（普通合辑）
| 限制项 | 要求 |
|--------|------|
| 格式 | JPG / PNG / WebP |
| 宽度 | 1080px |
| 文件大小 | ≤ 500KB |

### 介绍长图（介绍合辑）
| 限制项 | 要求 |
|--------|------|
| 格式 | JPG / PNG / WebP |
| 宽度 | 1080px |
| 单张文件大小 | ≤ 2MB |
| 数量 | 支持多张，按顺序从上到下拼接展示 |

### 预览图（主题/小组件）
| 限制项 | 要求 |
|--------|------|
| 格式 | JPG / PNG / WebP |
| 宽度 | 1080px |
| 文件大小 | ≤ 500KB |

### 小组件资源包
| 限制项 | 要求 |
|--------|------|
| 格式 | ZIP |
| 文件大小 | ≤ 20MB |

## JSON 数据结构

### 普通合辑
```json
{
  "zone": "xiaodounni",
  "collectionType": "normal",
  "headerImageUrl": "https://cdn.example.com/header.png",
  "backgroundColor": "#f0f4ff",
  "theme": {
    "previewUrl": "https://cdn.example.com/theme/preview.png",
    "xiaomi": {
      "deeplink": "xiaomi://theme/detail?packId=xxx",
      "shareBrowserUrl": "https://zhuti.xiaomi.com/detail/share/xxx"
    },
    "oppo": {
      "deeplink": "oaps://theme/detail?from=h5&rtp=theme&id=xxx",
      "shareBrowserUrl": "https://theme-h5-static.heytap.com/..."
    },
    "vivo": {
      "deeplink": "resdetail://resdetailhost?pkg=h5.share&restype=1&id=xxx",
      "shareBrowserUrl": "https://ai-h5.vivo.com.cn/itheme-share-detail/..."
    }
  },
  "widgets": [
    {
      "type": "calendar",
      "previewUrl": "https://cdn.example.com/widget/calendar_preview.png",
      "default": {
        "resourceUrl": "https://cdn.example.com/widget/calendar.zip"
      },
      "xiaomi": {
        "deeplink": "xiaomi://widget/calendar/xxx",
        "shareBrowserUrl": "https://widget.xiaomi.com/calendar/xxx"
      }
    }
  ]
}
```

### 介绍合辑
```json
{
  "zone": "xiaodounni",
  "collectionType": "intro",
  "introImageUrls": [
    "https://cdn.example.com/intro_1.png",
    "https://cdn.example.com/intro_2.png",
    "https://cdn.example.com/intro_3.png"
  ]
}
```

## 字段说明

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `zone` | string | ✅ | 所属专区标识 |
| `collectionType` | string | ✅ | 合辑类型：`normal` 普通合辑 / `intro` 介绍合辑 |
| `headerImageUrl` | string | 普通合辑必填 | 头图 CDN 地址 |
| `introImageUrls` | array | 介绍合辑必填 | 介绍长图 CDN 地址数组，按顺序拼接展示 |
| `backgroundColor` | string | ❌ | 背景色（默认白色时不输出） |
| `theme` | object | ❌ | 主题配置（仅普通合辑） |
| `theme.previewUrl` | string | ❌ | 主题预览图 CDN 地址 |
| `theme.{platform}.deeplink` | string | ❌ | App 内跳转地址 |
| `theme.{platform}.shareBrowserUrl` | string | ❌ | 浏览器分享地址 |
| `widgets` | array | ❌ | 小组件列表（仅普通合辑） |
| `widgets[].type` | string | ✅ | 组件类型：`calendar` / `clock` / `todo` / `battery` |
| `widgets[].previewUrl` | string | ❌ | 小组件预览图 CDN 地址 |
| `widgets[].default.resourceUrl` | string | ❌ | 自研 ZIP 资源包地址 |
| `widgets[].{platform}.deeplink` | string | ❌ | 平台 App 跳转地址 |
| `widgets[].{platform}.shareBrowserUrl` | string | ❌ | 平台浏览器分享地址 |

## 支持平台

### 主题配置
- 小米 (xiaomi)
- OPPO (oppo)
- vivo (vivo)
- 华为 (huawei) - 暂未上架

### 小组件配置
- 默认 (default) - 自研资源包
- 小米 (xiaomi)
- OPPO (oppo)
- vivo (vivo)
- 华为 (huawei)

## 小组件类型

| 类型值 | 名称 |
|--------|------|
| `calendar` | 日历组件 |
| `clock` | 时间组件 |
| `todo` | 待办事项组件 |
| `battery` | 电量组件 |

## 浏览器兼容性

- Chrome 90+
- Safari 14+
- Firefox 88+
- Edge 90+

> 注：屏幕取色功能（EyeDropper API）仅支持 Chrome 95+ 和 Edge 95+

## 技术栈

- 纯原生 HTML/CSS/JavaScript
- 无需构建工具
- 无外部依赖（仅使用 Google Fonts）
