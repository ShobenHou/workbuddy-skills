# QA Process & Common Pitfalls

## QA Process

**Assume there are problems. Your job is to find them.**

Your first render is almost never correct. Approach QA as a bug hunt, not a confirmation step. If you found zero issues on first inspection, you weren't looking hard enough.

### Content QA

```bash
python -m markitdown output.pptx
```

Check for missing content, typos, wrong order.

**Check for leftover placeholder text:**

```bash
python -m markitdown output.pptx | grep -iE "xxxx|lorem|ipsum|placeholder|this.*(page|slide).*layout"
```

If grep returns results, fix them before declaring success.

### 坐标边界 QA（每张幻灯片必须检查）

幻灯片尺寸：**宽 10"，高 5.625"**。底部三色金条占据 y=5.37~5.625（0.255"）。

检查清单：
- `x + w ≤ 10`（元素不超出右边界）
- `y + h ≤ 5.235`（内容区元素不进入底部金条区）
- 底部金条：`y=5.37, h=0.255`（5.37+0.255=5.625，贴紧底部无空隙）
- 多列卡片：`x[i] = startX + i*(cardW + gap)`，验证最后一列 `x[n-1] + cardW ≤ 9.6`
- 卡片内容偏移：文字/元素 x 从卡片 x 至少 `+0.1"` 内缩，y 从卡片 y 至少 `+0.1"` 内缩

### Verification Loop

1. Generate slides -> Extract text with `python -m markitdown output.pptx` -> Review content
2. **List issues found** (if none found, look again more critically)
3. Fix issues
4. **Re-verify affected slides** — one fix often creates another problem
5. Repeat until a full pass reveals no new issues

**Do not declare success until you've completed at least one fix-and-verify cycle.**

### Per-Slide QA (for from-scratch creation)

```bash
python -m markitdown slide-XX-preview.pptx
```

Check for missing content, placeholder text, missing page number badge.

---

## Common Mistakes to Avoid

- **Don't repeat the same layout** — vary columns, cards, and callouts across slides
- **Don't center body text** — left-align paragraphs and lists; center only titles
- **Don't skimp on size contrast** — titles need 36pt+ to stand out from 14-16pt body
- **Don't default to blue** — pick colors that reflect the specific topic
- **Don't mix spacing randomly** — choose 0.3" or 0.5" gaps and use consistently
- **Don't style one slide and leave the rest plain** — commit fully or keep it simple throughout
- **Don't create text-only slides** — add images, icons, charts, or visual elements; avoid plain title + bullets
- **Don't forget text box padding** — when aligning lines or shapes with text edges, set `margin: 0` on the text box or offset the shape to account for padding
- **Don't use low-contrast elements** — icons AND text need strong contrast against the background
- **NEVER use accent lines under titles** — these are a hallmark of AI-generated slides; use whitespace or background color instead
- **NEVER use "#" with hex colors** — causes file corruption in PptxGenJS
- **NEVER encode opacity in hex strings** — use the `opacity` property instead
- **NEVER use async/await in createSlide()** — compile.js won't await
- **NEVER reuse option objects across PptxGenJS calls** — PptxGenJS mutates objects in-place

---

## Critical Pitfalls — PptxGenJS

### NEVER use async/await in createSlide()

```javascript
// WRONG - compile.js won't await
async function createSlide(pres, theme) { ... }

// CORRECT
function createSlide(pres, theme) { ... }
```

### NEVER use "#" with hex colors

```javascript
color: "FF0000"      // CORRECT
color: "#FF0000"     // CORRUPTS FILE
```

### NEVER encode opacity in hex strings

```javascript
shadow: { color: "00000020" }              // CORRUPTS FILE
shadow: { color: "000000", opacity: 0.12 } // CORRECT
```

### Prevent text wrapping in titles

```javascript
// Use fit:'shrink' for long titles
slide.addText("Long Title Here", {
  x: 0.5, y: 2, w: 9, h: 1,
  fontSize: 48, fit: "shrink"
});
```

### NEVER reuse option objects across calls

```javascript
// WRONG
const shadow = { type: "outer", blur: 6, offset: 2, color: "000000", opacity: 0.15 };
slide.addShape(pres.shapes.RECTANGLE, { shadow, ... });
slide.addShape(pres.shapes.RECTANGLE, { shadow, ... });

// CORRECT - factory function
const makeShadow = () => ({ type: "outer", blur: 6, offset: 2, color: "000000", opacity: 0.15 });
slide.addShape(pres.shapes.RECTANGLE, { shadow: makeShadow(), ... });
slide.addShape(pres.shapes.RECTANGLE, { shadow: makeShadow(), ... });
```

### PptxGenJS addShape 的 w/h 必须用属性名显式传递

```javascript
// ❌ 错误：JavaScript 对象简写语法会导致 w/h 错误地继承变量名
const projCardW = 3.05;
const projCardH = 2.15;
slide.addShape(pres.shapes.ROUNDED_RECTANGLE, {
  x: px, y: projCardY, projCardW, projCardH,  // ❌ 错误！PptxGenJS 不认识 projCardW/projCardH
  fill: { color: "F8F6F2" }
});

// ✅ 正确：必须用 w: 和 h: 显式赋值
slide.addShape(pres.shapes.ROUNDED_RECTANGLE, {
  x: px, y: projCardY, w: projCardW, h: projCardH,  // ✅ 正确
  fill: { color: "F8F6F2" }
});
```

**这是最常见的位置错乱来源！** 当 `w:` 或 `h:` 缺失时，形状会使用默认尺寸（通常是 1" 或 0），导致内容溢出或错位。

### 双层阴影叠加卡片会导致框图偏移

```javascript
// ❌ 禁止使用：阴影偏移层会造成视觉上的框图错位
slide.addShape(pres.shapes.ROUNDED_RECTANGLE, {
  x: x + 0.04, y: y + 0.05, w, h,        // ❌ 偏移层
  fill: { color: "1D1D1A", transparency: 60 }, ...
});
slide.addShape(pres.shapes.ROUNDED_RECTANGLE, { x, y, w, h, ... });  // 主体层

// ✅ 正确：使用单层圆角矩形 + 金色边框
slide.addShape(pres.shapes.ROUNDED_RECTANGLE, {
  x, y, w, h,
  fill: { color: "F8F6F2" },
  line: { color: "C6A86F", width: 1 },
  rectRadius: 0.08
});
```

### 底部金条坐标错误导致与底部之间有空隙

```javascript
// ❌ 错误：深金层高度太小，金条与幻灯片底部之间有空隙（5.37+0.135=5.505 ≠ 5.625）
slide.addShape(pres.shapes.RECTANGLE, { x: 0, y: 5.37, w: 10, h: 0.135, ... });

// ✅ 正确：深金底层 y=5.37, h=0.255（5.37+0.255=5.625，紧贴底部无空隙）
// 三层叠加（深金底层最厚，中金/浅金叠在上面）：
slide.addShape(pres.shapes.RECTANGLE, {
  x: 0, y: 5.37, w: 10, h: 0.255,   // 深金底层，贴底
  fill: { color: "AA8241" }, line: { type: "none" }
});
slide.addShape(pres.shapes.RECTANGLE, {
  x: 0, y: 5.37, w: 10, h: 0.135,   // 中金叠加
  fill: { color: "C6A86F" }, line: { type: "none" }
});
slide.addShape(pres.shapes.RECTANGLE, {
  x: 0, y: 5.37, w: 10, h: 0.05,    // 浅金细条，顶层装饰
  fill: { color: "EDD7AE" }, line: { type: "none" }
});
```
