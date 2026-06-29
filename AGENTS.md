# 项目设计规范

本项目是一个纯静态中文个人作品集网站，由原生 HTML + CSS + JavaScript 构成。修改时请优先复用已有的 UI 模式，保持视觉统一。

---

## 1. 技术约束

- **不使用任何前端框架**（React / Vue / Tailwind 等）。
- **不引入构建工具**。页面直接用浏览器打开即可。
- CSS 采用 `<style>` 内联在每个 HTML 文件中，当前为复制式复用。
- 交互使用原生 JavaScript（`IntersectionObserver`、DOM 事件等）。
- 字体从 Google Fonts 加载：
  - 标题/展示：`Noto Serif SC`
  - 正文/UI：`Noto Sans SC`
  - 首页大标题：`Lilita One`
  - 文字作品编号：`Oswald`

---

## 2. 色彩系统

所有页面使用同一套 `:root` 变量，**不要新增主色**。

```css
:root {
  --green: #1a9e3f;        /* 页面主背景色 */
  --green-dark: #0f7a2e;   /* 导航栏、星星分隔条背景 */
  --green-light: #d4f5dc;  /* 统计卡片、联系链接背景 */
  --yellow: #f5e642;       /* 强调色：标题、按钮、徽章、标签 */
  --crimson: #d72660;      /* 强调色：图标、按钮、高亮 */
  --crimson-dark: #a81d4a; /* 较少使用 */
  --cream: #fffbe8;        /* 卡片底色 */
  --ink: #1a1a1a;          /* 主要文字色 */
}
```

使用规则：
- 页面背景：`var(--green)`
- 卡片/内容区背景：`var(--cream)`
- 大标题/section-title：`var(--yellow)` + `text-shadow: 2px 2px 0 #000;`
- 装饰/图标：`var(--crimson)` 或 `var(--yellow)`
- 正文：深色用 `#444` / `#555`，浅色用 `#fff`

---

## 3. 字体规范

```css
--serif: "Noto Serif SC", serif;
--sans: "Noto Sans SC", sans-serif;
```

| 用途 | 字体 | 字重 | 备注 |
|------|------|------|------|
| 导航 Logo | var(--serif) | 900 | 黄色，带黑影 |
| Section 标题 | var(--serif) | 900 | 26px，黄色 |
| 卡片大标题 | var(--serif) | 700-900 | 黑色 |
| 正文/描述 | var(--sans) | 400 | 14px，行高 1.7-1.9 |
| 标签/小字 | var(--sans) | 700 | 11-13px |
| 按钮文字 | var(--serif) | 700 | 15px |

---

## 4. 通用组件样式

### 4.1 卡片（Card）

最基础的容器样式，广泛用于 about-card、contact-card、favorite-card、timeline-card 等。

```css
.card-base {
  background: var(--cream);
  border: 3px solid #000;
  border-radius: 20px;
  box-shadow: 6px 6px 0 #000;
  padding: 1.5rem;
}
```

变体：
- **小卡片**（timeline-card、resume-item）：`border-radius: 14px; box-shadow: 4px 4px 0 #000;`
- **标签/徽章**：`border-radius: 100px;`

### 4.2 按钮

两类主按钮：

```css
/* 红色主按钮 */
.btn-crimson {
  background: var(--crimson);
  color: #fff;
  border: 3px solid #000;
  box-shadow: 4px 4px 0 #000;
  padding: 12px 28px;
  font-family: var(--serif);
  font-size: 15px;
  font-weight: 700;
  border-radius: 100px;
  cursor: pointer;
  transition: all 0.1s;
  letter-spacing: 0.06em;
}
.btn-crimson:hover {
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0 #000;
}

/* 黄色次按钮 */
.btn-yellow {
  background: var(--yellow);
  color: var(--ink);
  border: 3px solid #000;
  box-shadow: 4px 4px 0 #000;
  padding: 12px 28px;
  font-family: var(--serif);
  font-size: 15px;
  font-weight: 700;
  border-radius: 100px;
  cursor: pointer;
  transition: all 0.1s;
  letter-spacing: 0.06em;
}
.btn-yellow:hover {
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0 #000;
}
```

### 4.3 标签（Tag）

```css
.tag {
  background: var(--yellow);
  border: 2px solid #000;
  border-radius: 100px;
  padding: 5px 14px;
  font-family: var(--sans);
  font-size: 12px;
  font-weight: 700;
  color: var(--ink);
  box-shadow: 2px 2px 0 #000;
  letter-spacing: 0.05em;
}
```

变体：
- 绿色标签（vtl-tag、timeline-tag）：背景 `var(--green-light)`，文字 `var(--green-dark)`

### 4.4 Section 标题栏

每个内容区块的标题统一结构：

```html
<div class="section-header">
  <div class="sec-icon">✦</div>
  <div class="section-title">标题文字</div>
</div>
```

```css
.section-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 1.5rem;
}
.section-title {
  font-family: var(--serif);
  font-weight: 900;
  font-size: 26px;
  color: var(--yellow);
  text-shadow: 2px 2px 0 #000;
  letter-spacing: 0.06em;
}
.sec-icon {
  background: var(--crimson);
  color: #fff;
  border: 2px solid #000;
  border-radius: 8px;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  box-shadow: 2px 2px 0 #000;
  flex-shrink: 0;
}
```

### 4.5 星星分隔条（Star Divider）

页面各区块之间使用统一的分隔条：

```html
<div class="star-divider">
  <div class="spin-dot d-y"></div>
  <span class="spin-star s1">★</span>
  <div class="spin-dot d-c"></div>
  <span class="spin-star s2">✦</span>
  <!-- ... 交替排列 -->
</div>
```

规则：
- 背景：`var(--green-dark)`
- 上下边框：`3px solid #000`
- 圆点使用 `spin-dot`，颜色类 `d-y`（黄）、`d-c`（红）、`d-w`（白）
- 星星使用 `spin-star`，尺寸类 `s1`–`s9`，持续旋转动画
- 新增分隔条时保持相同结构，可打乱星星/圆点顺序增加变化

### 4.6 导航栏

首页导航使用 `.nav-links` 锚点链接；子页面使用 `.nav-back` 返回首页。

```css
nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 2rem;
  background: var(--green-dark);
  border-bottom: 3px solid #000;
  position: sticky;
  top: 0;
  z-index: 100;
}
.nav-logo {
  font-family: var(--serif);
  font-weight: 900;
  font-size: 20px;
  color: var(--yellow);
  text-shadow: 2px 2px 0 #000;
  letter-spacing: 0.05em;
}
.nav-back {
  font-family: var(--sans);
  font-size: 13px;
  font-weight: 700;
  color: #fff;
  text-decoration: none;
  padding: 6px 14px;
  border-radius: 100px;
  border: 2px solid #fff;
  transition: all 0.15s;
  letter-spacing: 0.05em;
  display: flex;
  align-items: center;
  gap: 4px;
}
.nav-back:hover {
  background: var(--yellow);
  color: var(--ink);
  border-color: #000;
}
```

### 4.7 弹窗/灯箱（Lightbox / Modal）

统一结构：

```html
<div id="xxxLightbox" class="xxx-lightbox">
  <div class="xxx-lightbox-backdrop" onclick="closeXxxLightbox()"></div>
  <div class="xxx-lightbox-content">
    <button class="xxx-lightbox-close" onclick="closeXxxLightbox()">✕</button>
    <!-- 内容 -->
  </div>
</div>
```

样式要点：
- 背景幕：`rgba(0, 0, 0, 0.85)` + `backdrop-filter: blur(4px);`
- 内容容器：`var(--green)` 或 `var(--cream)` 背景，`3px solid #000`，圆角，阴影
- 关闭按钮：圆形，黑边，悬停变黄
- 支持 `Escape` 键关闭

---

## 5. 动效规范

### 5.1 滚动出现动画（Pop）

最常用动效，类名为 `.pop` 或 `.pop-once`：

```css
.pop {
  opacity: 0;
  transform: scale(0.7);
  transition: none;
}
.pop.visible {
  opacity: 1;
  transform: scale(1);
  transition:
    opacity 0.45s cubic-bezier(0.34, 1.56, 0.64, 1),
    transform 0.45s cubic-bezier(0.34, 1.56, 0.64, 1);
}
```

- `.pop`：进入视口时添加 `visible`，部分页面会在离开后移除。
- `.pop-once`：只出现一次，不重复触发。
- `.pop-group .pop`：可设置子元素 `transition-delay` 实现错开出现。

### 5.2 悬浮反馈

可点击元素普遍使用：

```css
transform: translate(2px, 2px);
box-shadow: 2px 2px 0 #000;  /* 或更小 */
```

营造“按下”的卡通感。

### 5.3 首页特有动效

- 大标题 `00 SPACE`：SVG 描边 + 填充 + 阴影淡入，依赖 `Lilita One` 字体加载完成。
- 副标题：逐词从左侧滑入。
- 顶部徽章：掉落回弹动画。

> 修改首页标题时，需要同步更新 SVG 内 `text` 元素的字符和描边长度。

---

## 6. 布局与间距

### 6.1 内容区

```css
.section {
  padding: 1.5rem 2rem;
  max-width: 800px;      /* 首页/工作经历 */
  /* 或 900px/1100px 用于工具、文字作品等更宽内容 */
  margin: 0 auto;
}
```

### 6.2 响应式断点

主要断点为 **720px** 和 **640px**：

```css
@media (max-width: 720px) {
  /* 工具卡片单列、摄影作品轮播缩小等 */
}

@media (max-width: 640px) {
  /* 工作经历垂直时间线变左对齐 */
}
```

### 6.3 Grid/Flex 模式

- 卡片列表：`display: flex; flex-direction: column; gap: 16px;`
- 两列网格：`display: grid; grid-template-columns: 1fr 1fr; gap: 24px;`
- 工具卡片图文并排：`grid-template-columns: 1.2fr 1fr;`（移动端变单列）
- 文件夹卡片：`display: grid; grid-template-columns: 1fr; gap: 28px;`

---

## 7. 页面特有模式

### 7.1 首页（index.html）

- Hero 区全屏居中，大标题使用 SVG 字符动画。
- 个人介绍使用 `.hero-card`。
- 作品入口使用 `.folder-puffy` 文件夹卡片。
- 联系区使用 `.contact-card`。

### 7.2 工具箱（tools.html）

- 每个工具使用 `.tool-card tool-card--hero`。
- 左侧文字 + 右侧媒体面板（tabs + image/prompt）。
- 图片支持点击放大（`openLightbox`）。
- 视频教程使用 modal + `<video>`。

### 7.3 文字作品（writings.html）

- 每条作品使用 `.writing-item`。
- 布局：封面图 + 大号编号 + 信息卡片。
- 奇偶项通过 `.writing-item--number-left` 切换左右顺序。
- 编号使用 `Oswald` 字体，尺寸 `clamp(160px, 22vw, 300px)`。

### 7.4 摄影（photography.html）

- 每组照片一个 `.coverflow.coverflow-v2` 轮播。
- 图片路径：`images/photography/sea/` 和 `images/photography/woods/`。
- 点击当前图片打开 `.photo-lightbox` 全屏浏览。

### 7.5 工作经历（work-experience.html）

- 使用 `.vtl` 垂直时间线 + `.timeline` 卡片两种样式并存。
- 时间线节点包含：时间段徽章、职位、地点、描述、标签。

---

## 8. 修改守则

1. **保持色彩一致**：不要引入新的主色调，优先使用 `:root` 变量。
2. **复用组件样式**：新增卡片/按钮/标签时，直接复制已有同名类，不要重新发明。
3. **保持黑色粗边框 + 硬阴影**：这是项目核心视觉特征。
4. **动画曲线优先使用**：`cubic-bezier(0.34, 1.56, 0.64, 1)`（弹性回弹）。
5. **新增页面时**：复制一个现有子页面作为模板，保留导航栏、星星分隔条、页脚结构。
6. **图片资源**：放到 `images/` 下对应子文件夹，命名避免中文和空格。
7. **交互组件**：弹窗/灯箱需要同步写打开、关闭、键盘 `Escape` 支持。
8. **响应式**：任何新增布局都要在 720px 以下测试，必要时单列化。

---

## 9. 当前已知待统一项

- ✅ 已统一：子页面导航 Logo 和页脚已替换为“YINGLIN”。
- `tools.html` 中数据分析 Agent 的 JSON 采用 JS 内嵌 Blob 下载，如需改为直接文件下载可简化。
- `work-experience.html` 中部分列表格式较乱，有时间可整理为统一结构。
