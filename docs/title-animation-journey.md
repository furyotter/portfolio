# 首页主标题「00 SPACE」动画方案演进记录

> 记录从发现问题、分析根因、多次尝试到最终落地方案的完整过程。

---

## 一、最初的效果与问题

首页 Hero 区的主标题「00 SPACE」使用 SVG `<text>` 元素 + `stroke-dasharray` 动画，实现「一笔一划写出再填充」的效果。

**遇到的视觉问题：**
- 某些字母在动画过程中看起来像「多条线分开运动」。
- 细描边先画完，然后填充，再隐藏描边，步骤之间有轻微割裂感。
- 用户反馈：笔画和描边像是分开的，不像一个整体在画出来。

---

## 二、根因分析

### 2.1 字形的多轮廓结构

字体 `Lilita One` 是粗体无衬线，很多字母由多个闭合轮廓组成：

- **0**：外圈一个轮廓 + 内圈一个轮廓
- **P**：外轮廓 + 内部封闭空间
- **A**：外轮廓 + 中间三角洞
- **S、C、E**：相对简单，基本是一条外轮廓

SVG `<text>` 的 `stroke` 动画会同时作用在字形的**所有轮廓**上。

### 2.2 固定 `stroke-dasharray: 1200` 太大

代码里固定写了：

```css
stroke-dasharray: 1200;
stroke-dashoffset: 1200;
```

但每个字母的实际路径长度只有约 **300–700**。当 `dasharray` 远远大于实际路径总长时，动画过程中「已画出的部分」会同时覆盖所有轮廓，导致：

- 外圈和内圈的线条同时开始动
- 每个轮廓的起点和方向由字体文件决定，可能方向不一致
- 看起来就是一个字母里「好几条线各自为政」

### 2.3 描边与填充分离

原来的动画流程是三步：

1. 画黄色细描边
2. 填充黄色
3. 隐藏描边

三个动作时间不同步，进一步放大了「线条和填充不在同一个节奏」的违和感。

---

## 三、尝试过的方案

### 方案 A：加粗描边、去掉填充

**思路：** 如果只有一个粗的「笔画」被画出，没有单独的填充/隐藏步骤，也许就不会割裂。

**改动：**
- `stroke-width` 从 3 改为 9
- 去掉 `titleFillSolid` 和 `titleHideStroke` 动画
- 只保留描边绘制动画

**结果：** 问题依然存在。因为核心矛盾没有解决——我们还是在用 `stroke-dasharray` 去画一个由多个轮廓组成的 `<text>`。只要字形有内外多个闭合路径，描边动画就会同时作用在所有路径上。

---

### 方案 B：动态计算每个字母的描边长度

**思路：** 用 JavaScript 根据每个字母的包围盒动态估算路径长度，让 `stroke-dasharray` 接近真实总长。

**改动：**

```javascript
const bbox = el.getBBox();
const pathLength = (bbox.width + bbox.height) * 4.5;
el.style.strokeDasharray = pathLength;
el.style.strokeDashoffset = pathLength;
```

**结果：** 相比固定 1200 有所改善，但因为 `<text>` 无法获取精确路径长度，估算总有偏差；而且多轮廓问题只是被缓解，没有被根除。

---

### 方案 C：手写 SVG 几何路径

**思路：** 放弃 `<text>`，改用 `<path>` 自己画字母。每个字母做成单一 `<path>`，多轮廓作为子路径按顺序绘制，从而彻底解决多线问题。

**第一次实现：** 用脚本生成圆角矩形、方块等几何形状拼出字母。

**结果：**
- **00**：两个圆角矩形加内孔，效果尚可。
- **SPACE**：完全走样。S 看起来像「8」，E 像三个方块堆叠，A 太宽太扁，整体认不出是 SPACE。

**结论：** 手写几何路径无法还原 Lilita One 的字母形态，可读性太差。

---

## 四、最终方案：提取真实字体字形为 SVG Path

### 4.1 核心洞察

要同时满足两个条件：

1. **动画可控**：每个字母是单一 `<path>`，外轮廓和内圈按顺序绘制。
2. **形态还原**：字母轮廓必须和原字体 Lilita One 一致。

唯一可靠的方式就是：**把 Lilita One 字体文件里的真实字形轮廓提取出来，转成 SVG `<path>`。**

### 4.2 执行步骤

1. **下载字体**  
   从 Google Fonts 获取 Lilita One 的 TTF 文件：
   ```bash
   curl -L -o /tmp/LilitaOne.ttf \
     "https://fonts.gstatic.com/s/lilitaone/v17/i7dPIFZ9Zz-WBtRtedDbUEY.ttf"
   ```

2. **提取字形轮廓**  
   使用 Python 的 `fontTools` 库读取字体，对每个字符（0, S, P, A, C, E）调用 `SVGPathPen`，将字形轮廓转换为 SVG path 数据。

3. **坐标转换**  
   字体坐标是 Y 轴向上，SVG 是 Y 轴向下，因此需要对 Y 轴取反并缩放：
   ```python
   scale = 0.1
   transform = TransformPen(pen, (scale, 0, 0, -scale, 0, 0))
   ```

4. **定位到原位置**  
   根据每个字形的包围盒计算水平居中偏移，再平移到原来的 `x`、`y` 基线位置：
   ```python
   trans_x = target_x - glyph_center_x
   trans_y = target_baseline_y
   ```

5. **替换 HTML 中的 SVG**  
   把原来的 `<text>` 标题完全替换为 `<path>` 标题，每个字母一个 `<g transform="...">` 包裹的 `<path>`。

### 4.3 动画配合

- CSS 保留 `stroke-dasharray` 动画，但不在 CSS 里写死长度。
- JavaScript 对每个 `<path>` 调用 `getTotalLength()`，设置精确的 `stroke-dasharray` 和 `stroke-dashoffset`。
- 因为每个字母是一个单一 `<path>`，其子路径会按顺序绘制，外圈画完再画内圈。
- 添加 `fill-rule: evenodd`，确保带孔字母的填充/镂空正确。

### 4.4 阴影处理

原来用 6 层偏移的 `<text>` 做阴影。改为 SVG `<filter>`：

```xml
<filter id="titleShadow">
  <feOffset in="SourceAlpha" dx="5" dy="5" result="offset"/>
  <feFlood flood-color="#000"/>
  <feComposite in2="offset" operator="in" result="shadow"/>
  <feMerge>
    <feMergeNode in="shadow"/>
    <feMergeNode in="SourceGraphic"/>
  </feMerge>
</filter>
```

简洁且与真实字形轮廓匹配。

---

## 五、最终效果

- **形态**：00 和 SPACE 都与原 Lilita One 字体完全一致，可读性高。
- **动画**：每个字母按真实轮廓一笔一划绘制；外圈完成后，内圈（如 0、P、A 的孔）再继续绘制，不再有「多线分开」的错乱感。
- **风格**：00 最终填充为黄色实心；SPACE 保持黄色描边风格；黑色投影统一由 SVG filter 生成。

---

## 六、关键 takeaway

| 问题 | 失败方案 | 成功关键 |
|------|---------|---------|
| `<text>` 多轮廓同时描边 | 加粗描边、动态 dasharray | 不用 `<text>`，改用 `<path>` |
| 手写路径不像原字体 | 几何图形拼接 | 直接提取字体真实字形 |
| 填充/镂空方向 | — | `fill-rule: evenodd` |
| 阴影与轮廓对齐 | 多层 `<text>` 阴影 | SVG filter 投影 |

---

## 七、相关文件

- `index.html`：标题 SVG、CSS 动画、JS 动画逻辑
- 字体源：`https://fonts.googleapis.com/css2?family=Lilita+One`
- 工具：`fontTools`（Python）用于提取字形轮廓
