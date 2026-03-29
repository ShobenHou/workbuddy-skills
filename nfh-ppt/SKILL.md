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

## Assets 目录

`assets/` 目录包含从农发行官方模板中提取的原始图片资源：

| 文件 | 用途 |
|------|------|
| `wheat-circle.svg` | 右侧/右下角金色麦穗圆弧装饰（最常用） |
| `wheat-circle-outline.svg` | 麦穗圆弧轮廓版 |
| `wheat-circle-light.svg` | 麦穗圆弧浅金版 |
| `footer-stripe.svg` | 底部三色金条 SVG（可选用图片替代代码绘制） |
| `header-logo-decor.svg` | 标题区农行标志装饰 |
| `decor-circle.png` / `decor-circle2.png` | 装饰圆形图案 |
| `closing-decor.png` | 结尾页装饰图案 |

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
2. Assets 路径：`const ASSETS = require('path').resolve(__dirname, '../assets');`（需从 skill 目录复制 assets 到工作目录，或使用绝对路径）
3. 最终 PPTX 输出至：`slides/output/`
4. 尺寸：10" × 5.625" (LAYOUT_16x9)
5. **中文字体必须是 `微软雅黑`**（不得使用 Microsoft YaHei，两者不同）
6. 颜色：6位十六进制不带 # （如 `"C7000A"`）
7. 使用下方 theme 对象 contract
8. 参考 [pptxgenjs.md](references/pptxgenjs.md)

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

### 第五步：QA 验证

See [pitfalls.md](references/pitfalls.md#qa-process).

```bash
python3 -m markitdown output/presentation.pptx
```

Check: all text content present, no placeholder text, page numbers on all non-cover slides.

---

## Slide Output Format

```javascript
// slide-01.js
const pptxgen = require("pptxgenjs");
const path = require("path");

// Assets 路径（使用绝对路径确保图片能被找到）
const ASSETS = path.resolve(__dirname, "../../assets");
// 若 assets 已复制到 slides/ 旁边，则：
// const ASSETS = path.resolve(__dirname, "../assets");

const slideConfig = {
  type: 'cover',   // cover | section | content | closing
  index: 1,
  title: '演示文稿标题'
};

// 必须是同步函数（非 async）
function createSlide(pres, theme) {
  const slide = pres.addSlide();
  // ... 幻灯片内容 ...
  return slide;
}

// 单独预览
if (require.main === module) {
  const pres = new pptxgen();
  pres.layout = 'LAYOUT_16x9';
  const theme = {
    primary:   "C7000A",
    secondary: "78000F",
    accent:    "C6A86F",
    light:     "EDD7AE",
    bg:        "FFFFFF"
  };
  createSlide(pres, theme);
  pres.writeFile({ fileName: "./output/slide-01-preview.pptx" });
}

module.exports = { createSlide, slideConfig };
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
2. **底部三色金条**：所有正文页必须有（浅金/中金/深金三层横条，y≈5.23-5.505）
3. **标题下金色细线**：正文页标题下方必须有 `C6A86F` 横线
4. **品牌红只用于**：封面装饰、章节编号圆、关键数据强调、底部横条
5. **背景白色**：正文页 bg=`FFFFFF`，封面/章节页 bg=`1D1D1A`
6. **麦穗装饰图案**：来自 assets/ 目录，用于封面、章节页、结尾页的右侧区域

---

## 依赖

```bash
pip install "markitdown[pptx]"  # QA 文字提取
npm install pptxgenjs            # 幻灯片生成
```
