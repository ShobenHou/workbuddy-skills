---
name: nfh-ppt
description: "Generate PowerPoint presentations in the style of Agricultural Development Bank of China (农发行/中国农业发展银行) official template. Use this skill when creating PPT/slides for 农发行, 中国农业发展银行, or any presentation requiring 农发行 brand style (red + gold color scheme). Triggers: 农发行PPT, 农发行幻灯片, 农发行模板, 农发行风格, 农发行汇报."
description_zh: "按照农发行官方模板风格生成演示文稿"
description_en: "Generate presentations in Agricultural Development Bank of China brand style"
license: proprietary
metadata:
  version: "1.0"
  category: productivity
  organization: 中国农业发展银行 软件开发中心
---

# 农发行 PPT 生成器

## 概述

To generate PowerPoint presentations matching the Agricultural Development Bank of China (农发行) official template style.
Use PptxGenJS to create slides with the exact color palette, layout patterns, and decorative elements from the 农发行 brand guidelines.

## Quick Reference

| 项目 | 值 |
|------|-----|
| **尺寸** | 10" × 5.625" (LAYOUT_16x9) |
| **主色（品牌红）** | `C7000A` |
| **辅色（深暗红）** | `78000F` |
| **装饰金** | `C6A86F` |
| **浅金** | `EDD7AE` |
| **背景白** | `FFFFFF` |
| **深色背景** | `1D1D1A` |
| **中文字体** | `微软雅黑`（MANDATORY） |
| **英文字体** | `Arial` |

## 参考文件

| 文件 | 内容 |
|------|------|
| [design-system.md](references/design-system.md) | 农发行配色规范、字体规范、装饰元素代码 |
| [slide-types.md](references/slide-types.md) | 4种幻灯片类型：封面页、小标题页、正文页、结尾页 |
| [pitfalls.md](references/pitfalls.md) | QA流程、常见错误、PptxGenJS关键陷阱 |
| [pptxgenjs.md](references/pptxgenjs.md) | PptxGenJS完整API参考 |

## Assets 目录 与 图片嵌入规则

> **🚨 关键规则：所有图片必须用 base64 data URI 嵌入，禁止使用 `path:` 本地路径！**  
> 使用路径引用会导致 PPTX 在其他机器上打开时图片消失/乱码！

`assets/` 目录包含从农发行官方模板中提取的原始图片资源：

| 文件 | 用途 |
|------|------|
| `wheat-circle.png` | 右侧/右下角金色麦穗圆弧装饰（最常用，封面必须） |
| `wheat-circle.svg` | 麦穗圆弧 SVG 矢量版（备用） |
| `footer-gold-bar.png` | 底部三色金条（所有页必须，官方模板提取） |
| `header-logo-bar.png` | 顶部左侧农行金色标志横条（封面必须） |
| `wheat-circle-outline.svg` | 麦穗圆弧轮廓版 |
| `wheat-circle-light.svg` | 麦穗圆弧浅金版 |
| `footer-stripe.svg` | 底部三色金条 SVG 矢量版 |
| `header-logo-decor.svg` | 标题区农行标志装饰 |
| `decor-circle.png` / `decor-circle2.png` | 装饰圆形图案 |
| `closing-decor.png` | 结尾页装饰图案 |

### 使用方式（必须）

在 `slides/` 目录下创建 `nfh-assets.js` 文件，将图片编码为 base64：

```javascript
// nfh-assets.js（由工具生成，或手动执行以下 Python 脚本生成）
// python3 -c "
// import base64
// imgs = {'FOOTER_GOLD_BAR':'assets/footer-gold-bar.png','WHEAT_CIRCLE':'assets/wheat-circle.png','HEADER_LOGO_BAR':'assets/header-logo-bar.png'}
// print('module.exports = {')
// for k,p in imgs.items():
//     d = base64.b64encode(open(p,'rb').read()).decode()
//     print(f'  {k}: \"data:image/png;base64,{d}\",')
// print('};')
// "

// 每个 slide 中引用：
const { FOOTER_GOLD_BAR, WHEAT_CIRCLE, HEADER_LOGO_BAR } = require("./nfh-assets.js");
// 使用 data 属性：
slide.addImage({ data: FOOTER_GOLD_BAR, x: 0, y: 5.04, w: 10, h: 0.60 });
```

---

## 从零创建 — 工作流

### 第一步：分析需求

Understand presentation topic, sections, audience, and content depth. No color palette selection needed — use the mandatory 农发行 palette from [design-system.md](references/design-system.md).

### 第二步：规划幻灯片大纲

Classify every slide as one of the 4 types in [slide-types.md](references/slide-types.md):
1. **封面页** — 演示文稿第一张
2. **小标题页** — 章节切换（PART 01 / PART 02...）
3. **正文页** — 所有内容幻灯片（最多变体）
4. **结尾页** — 演示文稿最后一张

Ensure visual variety — do NOT repeat the same content layout across slides.

### 第三步：生成幻灯片 JS 文件

Create one JS file per slide in `slides/` directory. Each file exports a synchronous `createSlide(pres, theme)` function.

**告知每个 subagent 的关键信息（Generate up to 5 slides concurrently）：**
1. 文件命名：`slides/slide-01.js`, `slides/slide-02.js`, 等
2. 图片引用：`const { FOOTER_GOLD_BAR, WHEAT_CIRCLE, HEADER_LOGO_BAR } = require("./nfh-assets.js");`（必须用 `data:` base64 嵌入，禁止 `path:` 路径）
3. 最终 PPTX 输出至：`slides/output/`
4. 尺寸：10" × 5.625" (LAYOUT_16x9)
5. **中文字体必须是 `微软雅黑`**（不得使用 Microsoft YaHei，两者不同）
6. 颜色：6位十六进制不带 # （如 `"C7000A"`）
7. 使用下方 theme 对象 contract
8. 参考 [pptxgenjs.md](references/pptxgenjs.md)
9. **`addShape` 的 `w:` 和 `h:` 必须显式写出**，不能用 JS 对象简写 `{myW, myH}`
10. **底部金条**：三层矩形，最底层 `y=5.37, h=0.255`（5.37+0.255=5.625 贴底），无页码
11. **每个 slide 文件末尾必须加 `if (require.main === module)` 预览块**，方便逐页生成单独 PPTX 预览

### 第四步：编译为最终 PPTX

Create `slides/compile.js`:

```javascript
// slides/compile.js
const pptxgen = require('pptxgenjs');
const pres = new pptxgen();
pres.layout = 'LAYOUT_16x9';

// 农发行标准主题色（MANDATORY — 不得修改）
const theme = {
  primary:   "C7000A",  // 品牌红
  secondary: "78000F",  // 深暗红
  accent:    "C6A86F",  // 标准金
  light:     "EDD7AE",  // 浅金
  bg:        "FFFFFF"   // 纯白背景
};

for (let i = 1; i <= N; i++) {  // N = 总幻灯片数
  const num = String(i).padStart(2, '0');
  const slideModule = require(`./slide-${num}.js`);
  slideModule.createSlide(pres, theme);
}

pres.writeFile({ fileName: './output/presentation.pptx' })
  .then(() => console.log('✅ 生成完成'));
```

Run: `cd slides && node compile.js`

### 第五步：预览 + 位置自检（必须执行）

编译完成后，必须运行文本提取来自检内容是否完整，同时对坐标进行边界检查：

```bash
# 1. 文本内容检查
python3 -m markitdown output/presentation.pptx

# 2. 坐标边界检查（确保所有元素在 10×5.625 范围内）
node -e "
const slides = require('./compile.js'); // 或逐个require slide 文件
// 检查原则：所有元素 x+w <= 10, y+h <= 5.625
console.log('边界检查：幻灯片宽度=10, 高度=5.625');
console.log('底部金条应在 y=5.37, h=0.255（合计5.625）');
"
```

**位置错乱自检清单（每次生成后核对）：**
- [ ] 所有卡片的 `x + w` ≤ 10（不超出右边界）
- [ ] 所有卡片的 `y + h` ≤ 5.235（不进入底部金条区域）
- [ ] 底部金条：最底层 `y=5.37, h=0.255`（5.37+0.255=5.625，紧贴底部）
- [ ] 多列布局：每列 `x` = `startX + i*(cardW + gap)`，确保 `gap` 足够（最少 0.1"）
- [ ] 无页码文字（正文页不加页码）
- [ ] 卡片内文字 x/y 要从卡片边界内缩：至少 `+0.1"` 偏移



## Slide Output Format

```javascript
// slide-XX.js
const pptxgen = require("pptxgenjs");
const { FOOTER_GOLD_BAR, WHEAT_CIRCLE, HEADER_LOGO_BAR } = require("./nfh-assets.js");

const slideConfig = {
  type: 'content',   // cover | section | content | closing
  index: 2,
  title: '幻灯片标题'
};

// 必须是同步函数（非 async）
function createSlide(pres, theme) {
  const slide = pres.addSlide();
  slide.background = { color: "FFFFFF" };

  // ── 标题区 ──
  slide.addText("页面标题", {
    x: 0.44, y: 0.32, w: 9.0, h: 0.46,
    fontSize: 22, fontFace: "微软雅黑",
    color: "1D1D1A", bold: true,
    align: "left", valign: "middle"
  });
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0.44, y: 0.80, w: 9.1, h: 0.02,
    fill: { color: "C6A86F" }, line: { type: "none" }
  });

  // ── 内容区（y 从 0.92 开始，最大到 5.2 不超出金条区域）──
  // 所有卡片/形状：y + h ≤ 5.2（给金条留出空间）
  // 所有卡片/形状：x + w ≤ 9.8（留出右边距）

  // ── 底部三色金条（贴底，y=5.37, h=0.255, 5.37+0.255=5.625 ✅）──
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0, y: 5.37, w: 10, h: 0.255,
    fill: { color: "AA8241" }, line: { type: "none" }
  });
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0, y: 5.37, w: 10, h: 0.135,
    fill: { color: "C6A86F" }, line: { type: "none" }
  });
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0, y: 5.37, w: 10, h: 0.05,
    fill: { color: "EDD7AE" }, line: { type: "none" }
  });
  // ❌ 不加页码

  return slide;
}

// 单独预览（每个 slide 文件末尾都加这段，用于逐页检查）
if (require.main === module) {
  const pres = new pptxgen();
  pres.layout = 'LAYOUT_16x9';
  const theme = { primary: "C7000A", secondary: "78000F", accent: "C6A86F", light: "EDD7AE", bg: "FFFFFF" };
  createSlide(pres, theme);
  pres.writeFile({ fileName: `./output/slide-preview-${slideConfig.index}.pptx` })
    .then(() => console.log(`✅ slide-${slideConfig.index} 预览已生成`));
}

module.exports = { createSlide, slideConfig };
```

> **⚠️ 关键陷阱 — addShape 的 w/h 必须显式写 `w:` 和 `h:`**  
> JavaScript 对象简写 `{ x, y, cardW, cardH }` 不等于 `{ x, y, w: cardW, h: cardH }`。  
> PptxGenJS 只识别 `w` 和 `h` 属性名，简写会导致尺寸错误、框图位置错乱！

```javascript
// ❌ 错误（JS 简写，PptxGenJS 收不到 w/h）
slide.addShape(pres.shapes.ROUNDED_RECTANGLE, { x: px, y: py, cardW, cardH, ... });

// ✅ 正确（显式赋值）
slide.addShape(pres.shapes.ROUNDED_RECTANGLE, { x: px, y: py, w: cardW, h: cardH, ... });
```

---

## Theme Object Contract（强制）

| Key | 颜色 | 含义 |
|-----|------|------|
| `theme.primary` | `C7000A` | 品牌红，标题、强调 |
| `theme.secondary` | `78000F` | 深暗红，次要强调 |
| `theme.accent` | `C6A86F` | 标准金，装饰线、标签 |
| `theme.light` | `EDD7AE` | 浅金，背景色块 |
| `theme.bg` | `FFFFFF` | 纯白背景 |

**绝不使用其他 key 名称**（如 background, text, muted 等）。

---

## 必须遵守的品牌规则

1. **中文字体必须是 `微软雅黑`**，不是 "Microsoft YaHei"
2. **底部三色金条**：所有正文页/章节页必须有；三层叠加矩形，深金底层 `y=5.37, h=0.255`（贴底无空隙，5.37+0.255=5.625），中金/浅金层叠在上方
3. **标题下金色细线**：正文页标题下方必须有 `C6A86F` 横线（y≈0.78~0.82）
4. **品牌红（C7000A）严格限制**：仅用于章节页 PART 文字、最关键1-2个数据强调；禁止大面积使用
5. **徽章/圆点/边条/进度条**：一律使用金色 `C6A86F`，不用红色
6. **背景**：正文页 bg=`FFFFFF`，封面页 `FFFFFF`，结尾页 `1D1D1A`
7. **框图使用单层圆角矩形**：带金色边框，`rectRadius: 0.08`；**禁止使用双层阴影叠加方案**（会造成框图错位）
8. **图片必须 base64 嵌入**：使用 `data:` 属性，禁止 `path:` 本地路径
9. **麦穗装饰图案**：封面右侧必须使用 `WHEAT_CIRCLE`，顶部必须使用 `HEADER_LOGO_BAR`
10. **禁止页码**：正文页底部不添加任何页码文字

---

## 依赖

```bash
pip install "markitdown[pptx]"  # QA 文字提取
npm install pptxgenjs            # 幻灯片生成
```
