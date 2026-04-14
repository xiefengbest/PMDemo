# CloudWallpaper Creator Detail Plan B Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在仓库内新增一个可直接预览的 H5 原型 `好看云壁纸/creator-detail-plan-b.html`，实现“创作者详情页 Plan B：内容浏览优先、点作品进合辑详情、保留横向合辑入口”，并复用现有视觉与订阅/操作交互。

**Architecture:** 单文件原型，复用 `creator-detail.html` 的 CSS/组件（合辑详情页 push、壁纸操作 action sheet、订阅状态管理与 toast），新增“作品流聚合”与“横向合辑入口”。通过 URL 参数 `creatorId` 选择创作者数据。

**Tech Stack:** HTML + Tailwind CDN + Font Awesome + 原生 JS（与现有原型一致）。

---

## File Structure

**Create**
- `好看云壁纸/creator-detail-plan-b.html`: Plan B 原型页面

**Reference**
- `好看云壁纸/creator-detail.html`: 复用样式与交互（合辑详情页、订阅按钮、壁纸 action sheet、toast）
- `好看云壁纸/index.html`: 二级页 push 动效规范（Plan B 中用 `translateX` 风格实现）

---

### Task 1: 基于 Plan A 抽出可复用能力

**Files:**
- Read: `好看云壁纸/creator-detail.html`

- [ ] **Step 1: 读取 Plan A 原型，列出可直接复用的 DOM/JS 块**
  - 合辑详情页容器（push/返回）
  - 壁纸网格 + action sheet（单独设置/加入轮播池）
  - 订阅状态 `subscribedAlbums` 与订阅/取消订阅按钮状态切换
  - Toast 组件

- [ ] **Step 2: 明确 Plan B 需要新增的最小模块**
  - sticky 创作者 header
  - 横向合辑入口（不含订阅按钮）
  - 作品流 `creatorWorks`（从 albums wallpapers 扁平化）
  - `openAlbumDetail(albumId, focusIndex)` 支持 focus 定位/高亮（可降级为滚动到对应元素）

---

### Task 2: 新增 `creator-detail-plan-b.html`（页面骨架 + 样式复用）

**Files:**
- Create: `好看云壁纸/creator-detail-plan-b.html`

- [ ] **Step 1: 创建页面骨架**
  - 视口适配脚本与全局字体/背景（与 Plan A 一致）
  - 引入 Tailwind/Font Awesome（与 Plan A 一致）
  - 主页面结构：Header + 横向合辑入口 + 作品网格
  - 二级页面：合辑详情 push 容器（可直接复用 Plan A 的 `#album-detail-modal`）
  - action sheet + toast（复用 Plan A）

- [ ] **Step 2: 拷贝 mock 数据结构**
  - `CREATORS`（直接从 Plan A 复制，保证内容一致可比）

---

### Task 3: 实现 Plan B 数据与交互

**Files:**
- Modify: `好看云壁纸/creator-detail-plan-b.html`

- [ ] **Step 1: URL 参数解析与创作者选择**

```js
function getQueryParam(name) {
  const u = new URL(window.location.href);
  return u.searchParams.get(name);
}
const creatorId = getQueryParam('creatorId') || 'creator1';
```

- [ ] **Step 2: 生成横向合辑入口**
  - 点击合辑卡：`openAlbumDetail(album.id, 0)`

- [ ] **Step 3: 生成作品流**

```js
function buildCreatorWorks(creator) {
  const works = [];
  creator.albums.forEach(album => {
    (album.wallpapers || []).forEach((src, idxInAlbum) => {
      works.push({ src, albumId: album.id, albumName: album.name, idxInAlbum, update: album.update });
    });
  });
  return works;
}
```

- [ ] **Step 4: 点作品进入合辑详情并定位**
  - `openAlbumDetail(albumId, focusIndex)`：
    - 打开合辑详情
    - 渲染合辑壁纸网格时给每个 item 标注 `data-idx`
    - 打开后 `scrollIntoView()` 到 focus item，并临时加 `selected` 高亮

- [ ] **Step 5: 订阅/取消订阅与 action sheet**
  - 复用 Plan A：`subscribeCurrentAlbum()` / `unsubscribeCurrentAlbum()` / `openWallpaperSheet()` / `showToast()`

---

### Task 4: 手工验证（浏览器可用）

**Files:**
- Verify: `好看云壁纸/creator-detail-plan-b.html`

- [ ] **Step 1: 页面可打开且默认展示 creator1**
  - 预期：看到创作者 header、横向合辑入口、作品流网格

- [ ] **Step 2: 点击横向合辑入口打开合辑详情**
  - 预期：右滑/推入效果出现（或同等 push），能返回

- [ ] **Step 3: 点击作品进入合辑详情并定位**
  - 预期：进入对应合辑，定位到对应壁纸并短暂高亮

- [ ] **Step 4: 订阅/取消订阅与按钮状态正确**
  - 预期：订阅按钮在合辑详情页切换；toast 正常；返回后仍保持状态

- [ ] **Step 5: 壁纸 action sheet 可用**
  - 预期：点击壁纸弹 action sheet，“单独设置/加入轮播池”均 toast 返回并关闭

