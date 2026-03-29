# 农发行 PPT 设计规范

## 品牌背景

中国农业发展银行（农发行）官方PPT模板采用金融机构严谨风格，以**品牌红 + 古铜金**为核心配色，搭配白色背景与深色底板，体现国家政策性银行的权威感与专业性。

---

## 农发行标准配色

### 主色板（MANDATORY — 只使用此配色，不得引入其他颜色）

| 角色 | 颜色名 | Hex | 用途 |
|------|--------|-----|------|
| `theme.primary` | 农行品牌红 | `C7000A` | 封面装饰、强调色、关键数据 |
| `theme.secondary` | 深暗红 | `78000F` | 次要强调、深色背景上的点缀 |
| `theme.accent` | 标准金 | `C6A86F` | 标题下划线、装饰线、标签 |
| `theme.light` | 浅金 | `EDD7AE` | 卡片背景、浅色色块 |
| `theme.bg` | 纯白背景 | `FFFFFF` | 正文页背景 |

### 扩展色（可选，用于多级视觉层次）

| 颜色名 | Hex | 用途 |
|--------|-----|------|
| 深金 | `AA8241` | 底部横条深色层 |
| 中金 | `C6A86F` | 底部横条中色层（同accent） |
| 亮金 | `EDD7AE` | 底部横条浅色层（同light） |
| 深色文字 | `1D1D1A` | 正文文字、深色背景 |
| 辅助灰 | `666666` | 次要说明文字 |
| 白色 | `FFFFFF` | 深色背景上的标题文字 |
| 亮红 | `E9002F` | 强调数据、警示色（慎用） |

### 配色使用规则

```javascript
// 唯一合法的 theme 对象（编译脚本必须使用这组颜色）
const theme = {
  primary:   "C7000A",  // 农行品牌红
  secondary: "78000F",  // 深暗红
  accent:    "C6A86F",  // 标准金
  light:     "EDD7AE",  // 浅金
  bg:        "FFFFFF"   // 纯白背景
};
```

**禁止行为：**
- 禁止使用蓝色、绿色、紫色等非农发行品牌色
- 禁止自行调亮/调暗颜色（如减淡红色变为浅红）
- 深色幻灯片背景使用 `1D1D1A`，不得使用纯黑 `000000`

---

## 字体规范

| 用途 | 字体 | 备注 |
|------|------|------|
| **中文字体（所有中文文字）** | `微软雅黑` | MANDATORY |
| **英文/数字** | `Arial` | 次要使用 |

```javascript
// 所有中文文字必须指定
fontFace: "微软雅黑"

// 英文标注
fontFace: "Arial"
```

---

## 视觉风格

**整体风格**：`Sharp & Compact`（清晰紧凑，数据密集型，严谨专业）

| 参数 | 值 |
|------|-----|
| 圆角半径（小） | 0" |
| 圆角半径（中） | 0.03" |
| 元素内边距 | 0.1" ~ 0.15" |
| 元素间距 | 0.1" ~ 0.2" |
| 页面边距 | 0.4" |
| 区块间距 | 0.25" ~ 0.35" |

---

## 装饰元素系统

### 1. 底部三色金条（正文页标配）

每张正文页底部必须有三色金色横条装饰，位置在 y=5.35" 以下：

```javascript
// 底部三层金色横条（从下到上）
// 深金底层
slide.addShape(pres.shapes.RECTANGLE, {
  x: 0, y: 5.37, w: 10, h: 0.13,
  fill: { color: "AA8241" },
  line: { type: "none" }
});
// 中金中层
slide.addShape(pres.shapes.RECTANGLE, {
  x: 0, y: 5.28, w: 10, h: 0.09,
  fill: { color: "C6A86F" },
  line: { type: "none" }
});
// 浅金顶层（细线）
slide.addShape(pres.shapes.RECTANGLE, {
  x: 0, y: 5.23, w: 10, h: 0.05,
  fill: { color: "EDD7AE" },
  line: { type: "none" }
});
```

### 2. 标题区金色下划线

正文页标题下方必须有金色水平细线：

```javascript
// 标题下方金色分隔线
slide.addShape(pres.shapes.RECTANGLE, {
  x: 0.44, y: 0.8, w: 9.1, h: 0.02,
  fill: { color: "C6A86F" },
  line: { type: "none" }
});
```

### 3. 右下角麦穗圆弧装饰（正文页可选，结尾页必选）

麦穗圆弧图案 SVG 存放于 skill `assets/` 目录：
- `wheat-circle.svg` — 金色实心麦穗圆
- `wheat-circle-outline.svg` — 轮廓版麦穗圆
- `wheat-circle-light.svg` — 浅金麦穗圆

```javascript
// 正文页右下角装饰（可选，适合内容不满的页面）
// 注意：图片必须用绝对路径
const SKILL_ASSETS = require('path').resolve(__dirname, '../assets');
slide.addImage({
  path: `${SKILL_ASSETS}/wheat-circle.svg`,
  x: 7.2, y: 2.8, w: 2.8, h: 2.5,
  transparency: 15
});
```

### 4. 页码显示

正文页底部左侧显示"第N页"格式：

```javascript
// 页码（底部左侧，金色字）
slide.addText(`第 ${pageNum} 页`, {
  x: 0.15, y: 5.28, w: 1.0, h: 0.22,
  fontSize: 9, fontFace: "微软雅黑",
  color: "FFFFFF", bold: false,
  align: "center", valign: "middle"
});
```

---

## 幻灯片尺寸

| 项目 | 值 |
|------|-----|
| 布局 | LAYOUT_16x9 |
| 宽度 | 10" |
| 高度 | 5.625" |
| 安全边距 | 0.4" |

---

## 禁止事项

- **绝不使用渐变色**（使用纯色）
- **绝不在标题下方加蓝色/彩色下划线**（只用金色 C6A86F）
- **绝不使用粗体正文文字**（仅标题和子标题可加粗）
- **绝不在颜色值前加 #**（会导致文件损坏）
- **绝不将透明度编码进十六进制颜色字符串**（用 transparency 属性）
