# 农发行 PPT 幻灯片类型规范

将每张幻灯片归类为以下 4 种类型之一。

---

## 类型 1：封面页（Cover Page）

**触发**：演示文稿第一张，用于标题和场合说明。

### 布局（来自官方模板精确还原）

```
|  白色背景（全页）                                          |
|  [logo栏图片]  x=0.74 y=0.51 w=2.53 h=0.28              |
|                                                            |
|  标题文字（金色 #C6A86F，54pt）                           |
|  x=0.27 y=1.52 w=8.11 h=0.76                             |
|                                                            |
|  副标题关键词（金色小字）                                  |
|                                                            |
|  部门 · 日期（金色小字 16pt）                             |
|  x=1.20 y=3.20 w=4.68 h=0.31                             |
|                                                            |
|  [右侧麦穗圆弧图片]  x=5.82 y=1.88 w=4.18 h=3.21        |
|                                                            |
|  [底部金条图片]   x=0 y=5.047 w=10 h=0.596               |
```

### 实现要点

```javascript
function createCoverSlide(pres, opts) {
  const slide = pres.addSlide();
  // ✅ 模板背景：白色（通过theme2 bg1=lt1=FFFFFF定义）
  slide.background = { color: "FFFFFF" };

  const ASSETS = require('path').resolve(__dirname, '../assets');

  // ── 1. 底部三色金条（来自官方模板 image2.png）──
  // 使用真实图片素材，如果文件不存在则用代码替代
  try {
    slide.addImage({
      path: `${ASSETS}/footer-gold-bar.png`,
      x: 0, y: 5.047, w: 10, h: 0.596
    });
  } catch(e) {
    // Fallback：用代码绘制三层金条
    slide.addShape(pres.shapes.RECTANGLE, {
      x: 0, y: 5.047, w: 10, h: 0.596,
      fill: { color: "AA8241" }, line: { type: "none" }
    });
    slide.addShape(pres.shapes.RECTANGLE, {
      x: 0, y: 5.047, w: 10, h: 0.35,
      fill: { color: "C6A86F" }, line: { type: "none" }
    });
    slide.addShape(pres.shapes.RECTANGLE, {
      x: 0, y: 5.047, w: 10, h: 0.12,
      fill: { color: "EDD7AE" }, line: { type: "none" }
    });
  }

  // ── 2. 右侧麦穗圆弧装饰（来自官方模板 image23.png/image24.svg）──
  try {
    slide.addImage({
      path: `${ASSETS}/wheat-circle.png`,
      x: 5.824, y: 1.880, w: 4.176, h: 3.212
    });
  } catch(e) {
    try {
      slide.addImage({
        path: `${ASSETS}/wheat-circle.svg`,
        x: 5.824, y: 1.880, w: 4.176, h: 3.212
      });
    } catch(e2) { /* 无图片时跳过 */ }
  }

  // ── 3. 顶部logo横条（来自官方模板 image4.png）──
  try {
    slide.addImage({
      path: `${ASSETS}/header-logo-bar.png`,
      x: 0.740, y: 0.510, w: 2.530, h: 0.278
    });
  } catch(e) {
    // Fallback：金色横条
    slide.addShape(pres.shapes.RECTANGLE, {
      x: 0.74, y: 0.51, w: 2.53, h: 0.278,
      fill: { color: "C6A86F" }, line: { type: "none" }
    });
  }

  // ── 4. 主标题（金色，54pt，微软雅黑加粗）──
  // 模板原位：x=0.27 y=1.52 w=8.11 h=0.76 + 下方延伸
  slide.addText(opts.title || "演示文稿标题", {
    x: 0.27, y: 1.40, w: 5.3, h: 1.60,
    fontSize: 40, fontFace: "微软雅黑",
    color: "C6A86F", bold: true,
    align: "left", valign: "middle",
    fit: "shrink"
  });

  // ── 5. 副标题（金色，关键词）──
  if (opts.subtitle) {
    slide.addText(opts.subtitle, {
      x: 0.27, y: 3.05, w: 5.3, h: 0.45,
      fontSize: 16, fontFace: "微软雅黑",
      color: "C6A86F", bold: false,
      align: "left", valign: "middle"
    });
  }

  // ── 6. 部门 · 日期（金色，16pt）──
  // 模板原位：x=1.20 y=3.20 w=4.68 h=0.31
  const deptText = [
    opts.dept || "软件开发中心研发一部",
    opts.date || "2025年12月"
  ].filter(Boolean).join("    ");
  slide.addText(deptText, {
    x: 1.20, y: 3.55, w: 4.68, h: 0.40,
    fontSize: 16, fontFace: "微软雅黑",
    color: "C6A86F", bold: false,
    align: "left", valign: "middle"
  });

  return slide;
}
```

### 关键设计原则（来自模板实测）

| 元素 | 颜色 | 说明 |
|------|------|------|
| **背景** | `#FFFFFF` 白色 | ❌ 不使用黑色背景！ |
| **主标题** | `#C6A86F` 金色 | 54pt 微软雅黑加粗 |
| **部门日期** | `#C6A86F` 金色 | 16pt |
| **底部金条** | 图片 footer-gold-bar.png | 三层金色叠加条 |
| **右侧装饰** | 图片 wheat-circle.png/.svg | 农行麦穗圆弧图案 |
| **logo栏** | 图片 header-logo-bar.png | 顶部左侧金色标志 |

### ⚠️ 常见错误（必须避免）

- ❌ 不要设置黑色背景 `#0a0a0a` 或 `#1D1D1A`
- ❌ 不要添加大块红色矩形（左竖条、底部横条）
- ❌ 不要用白色文字（模板是金色在白色背景上）
- ✅ 背景必须是白色 `FFFFFF`
- ✅ 必须引用 assets 目录中的图片（底部金条 + 右侧麦穗 + logo栏）

---

## 类型 2：小标题页 / 章节过渡页（Section Divider）

**触发**：章节切换，显示 PART 0X + 章节名称。

### 布局

```
| 白色上区（~45%高）    PART 0X  |
| ──────────────────────────────── |（金色分隔线）
| 深色下区（~55%高）              |
| 大字章节标题（白色）            |
| 右下角麦穗装饰                  |
```

### 实现要点

```javascript
function createSectionSlide(pres, theme, partNum, title) {
  const slide = pres.addSlide();
  slide.background = { color: "FFFFFF" };

  // 下半区深色背景
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0, y: 2.4, w: 10, h: 3.225,
    fill: { color: "1D1D1A" },
    line: { type: "none" }
  });

  // 金色分隔横线
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0, y: 2.37, w: 10, h: 0.05,
    fill: { color: "C6A86F" },
    line: { type: "none" }
  });

  // PART 0X（品牌红，上区）
  slide.addText(`PART ${String(partNum).padStart(2, '0')}`, {
    x: 0.8, y: 0.9, w: 4.0, h: 0.8,
    fontSize: 36, fontFace: "Arial",
    color: "C7000A", bold: true,
    align: "left", valign: "middle",
    charSpacing: 4
  });

  // 章节标题（深色，大字）
  slide.addText(title, {
    x: 1.0, y: 2.8, w: 7.5, h: 1.5,
    fontSize: 40, fontFace: "微软雅黑",
    color: "FFFFFF", bold: true,
    align: "left", valign: "middle",
    fit: "shrink"
  });

  // 右下角装饰
  const ASSETS = require('path').resolve(__dirname, '../assets');
  slide.addImage({
    path: `${ASSETS}/wheat-circle.svg`,
    x: 7.0, y: 2.5, w: 2.8, h: 2.8,
    transparency: 30
  });

  // 页码
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0, y: 5.37, w: 10, h: 0.255,
    fill: { color: "1D1D1A" },
    line: { type: "none" }
  });

  return slide;
}
```

### 字体层级

| 元素 | 字号 | 颜色 | 加粗 |
|------|------|------|------|
| PART 0X | 36pt | `C7000A` | 是 |
| 章节标题 | 36-44pt | `FFFFFF` | 是 |
| 简介文字（可选） | 16-18pt | `C6A86F` | 否 |

---

## 类型 3：正文页（Content Page）

**触发**：所有正文内容幻灯片，是最常用的页面类型。

### 布局结构

```
|──────────────────────────────────────────|
| 标题区（顶部，白色背景）                  |
| 标题文字（黑色/深色，左对齐）             |
|________________金色细线__________________|
|                                           |
| 内容区（白色背景，各种布局变体）          |
|                                           |
|──────────────────────────────────────────|
| 底部三色金条（浅金/中金/深金）            |
|__________________________________________| ← 5.625"
```

### 标题区（必须）

```javascript
// 标题文字（左对齐，深色）
slide.addText("页面标题", {
  x: 0.44, y: 0.32, w: 9.0, h: 0.46,
  fontSize: 22, fontFace: "微软雅黑",
  color: "1D1D1A", bold: true,
  align: "left", valign: "middle"
});

// 标题下方金色细线（必须）
slide.addShape(pres.shapes.RECTANGLE, {
  x: 0.44, y: 0.8, w: 9.1, h: 0.02,
  fill: { color: "C6A86F" },
  line: { type: "none" }
});
```

### 底部三色金条（必须）

```javascript
// 深金底层
slide.addShape(pres.shapes.RECTANGLE, {
  x: 0, y: 5.37, w: 10, h: 0.135,
  fill: { color: "AA8241" },
  line: { type: "none" }
});
// 中金层
slide.addShape(pres.shapes.RECTANGLE, {
  x: 0, y: 5.28, w: 10, h: 0.09,
  fill: { color: "C6A86F" },
  line: { type: "none" }
});
// 浅金顶层
slide.addShape(pres.shapes.RECTANGLE, {
  x: 0, y: 5.23, w: 10, h: 0.05,
  fill: { color: "EDD7AE" },
  line: { type: "none" }
});
```

### 内容区布局变体

#### 变体 A：两列文字+图表

```javascript
// 左列文字 (x:0.44 ~ 4.5)，右列图表/数据 (x:5.0 ~ 9.5)
```

#### 变体 B：时间线/流程

```javascript
// 水平时间轴：每个节点圆圈（品牌红 C7000A）+ 连接线（金色 C6A86F）
```

#### 变体 C：数据看板

```javascript
// 数字卡片（白底+金色左边条）
slide.addShape(pres.shapes.RECTANGLE, {
  x: cardX, y: cardY, w: 2.2, h: 1.5,
  fill: { color: "FFFFFF" },
  line: { color: "C6A86F", width: 1 }
});
slide.addShape(pres.shapes.RECTANGLE, {
  x: cardX, y: cardY, w: 0.06, h: 1.5,
  fill: { color: "C6A86F" },
  line: { type: "none" }
});
// 大数字（品牌红）
slide.addText("85%", {
  x: cardX + 0.15, y: cardY + 0.1, w: 1.9, h: 0.8,
  fontSize: 48, fontFace: "Arial",
  color: "C7000A", bold: true
});
```

#### 变体 D：列表+图标

```javascript
// 列表项用品牌红实心圆作为 bullet
slide.addShape(pres.shapes.OVAL, {
  x: bulletX, y: bulletY, w: 0.1, h: 0.1,
  fill: { color: "C7000A" },
  line: { type: "none" }
});
```

### 页码（所有非封面页必须）

```javascript
// 页码文字在底部金条区域内（白色）
slide.addText(`${pageNum}`, {
  x: 0.1, y: 5.32, w: 0.5, h: 0.2,
  fontSize: 9, fontFace: "微软雅黑",
  color: "FFFFFF", bold: false,
  align: "center", valign: "middle"
});
```

---

## 类型 4：结尾页（Closing Page）

**触发**：演示文稿最后一张。

### 布局

```
| 深色全屏背景                            |
| 中央大字（如：谢谢！/ Thank You！）      |
| 副文字（联系方式 / 致谢语）             |
| 右侧大型麦穗圆弧图案（显著展示）         |
| 底部品牌红横条                          |
```

### 实现要点

```javascript
function createClosingSlide(pres, theme) {
  const slide = pres.addSlide();
  slide.background = { color: "1D1D1A" };

  // 底部品牌红横条
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0, y: 5.37, w: 10, h: 0.255,
    fill: { color: "C7000A" },
    line: { type: "none" }
  });

  // 大型麦穗装饰（右侧）
  const ASSETS = require('path').resolve(__dirname, '../assets');
  slide.addImage({
    path: `${ASSETS}/wheat-circle.svg`,
    x: 5.0, y: 0.3, w: 4.5, h: 4.8
  });

  // 主文字
  slide.addText("谢谢！", {
    x: 0.5, y: 1.8, w: 4.5, h: 1.2,
    fontSize: 60, fontFace: "微软雅黑",
    color: "FFFFFF", bold: true,
    align: "left", valign: "middle"
  });

  // 金色细线
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0.5, y: 3.1, w: 3.0, h: 0.03,
    fill: { color: "C6A86F" },
    line: { type: "none" }
  });

  // 联系方式或致谢语（金色）
  slide.addText("软件开发中心研发一部", {
    x: 0.5, y: 3.2, w: 4.5, h: 0.5,
    fontSize: 16, fontFace: "微软雅黑",
    color: "C6A86F", bold: false,
    align: "left", valign: "middle"
  });

  return slide;
}
```

---

## 常见辅助组件

### 红色数字徽章（用于 PART 编号）

```javascript
slide.addShape(pres.shapes.OVAL, {
  x: xPos, y: yPos, w: 0.55, h: 0.55,
  fill: { color: "C7000A" },
  line: { type: "none" }
});
slide.addText("01", {
  x: xPos, y: yPos, w: 0.55, h: 0.55,
  fontSize: 16, fontFace: "Arial",
  color: "FFFFFF", bold: true,
  align: "center", valign: "middle"
});
```

### 金色左边条卡片

```javascript
// 卡片容器
slide.addShape(pres.shapes.RECTANGLE, {
  x: x, y: y, w: w, h: h,
  fill: { color: "FFFFFF" },
  line: { color: "EDD7AE", width: 1 }
});
// 左侧金条
slide.addShape(pres.shapes.RECTANGLE, {
  x: x, y: y, w: 0.06, h: h,
  fill: { color: "C7000A" },
  line: { type: "none" }
});
```

### 进度条

```javascript
// 背景条（浅金）
slide.addShape(pres.shapes.RECTANGLE, {
  x: 0.5, y: yPos, w: 8.5, h: 0.12,
  fill: { color: "EDD7AE" },
  line: { type: "none" }
});
// 进度填充（品牌红）
slide.addShape(pres.shapes.RECTANGLE, {
  x: 0.5, y: yPos, w: 8.5 * ratio, h: 0.12,
  fill: { color: "C7000A" },
  line: { type: "none" }
});
```
