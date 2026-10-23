---
type: concept
tags: [CSS, 响应式, 视口单位]
created: 2026-10-14
aliases: [CSS 数学函数, min(), max(), clamp(), 流体尺寸, 流体排版, fluid typography, 响应式尺寸]
source: https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/clamp · https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/min · https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/max · https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/length · https://github.com/mdn/browser-compat-data · https://vue-zone.github.io/docs/vue3-vant-mobile/browser-adaptation
---
# CSS 数学函数

## 一句话定义
`min()` / `max()` / `clamp()` 是让浏览器在**解析时**从若干候选值里「取最小 / 取最大 / 夹进区间」的 CSS 函数；把它们与 `vw` 组合，就能让一个长度**随视口连续变化**，而不需要任何断点。

## 用于解决什么问题
只用 vw 写字号，小屏挤成一团、大屏夸张放大；退回媒体查询，就得为每个字号和间距各补一条断点，断点数量随设计稿膨胀，改一处尺寸要动好几处。

## 要点

### 三个函数的语义
| 函数 | 语义 | 例（数值为示意） |
| --- | --- | --- |
| `min(a, b, …)` | 取**最小**（最负）的那个值 | `width: min(90vw, 1200px)` → 宽屏最多 1200px，窄屏按 90vw 收 |
| `max(a, b, …)` | 取**最大**（最正）的那个值 | `width: max(20vw, 400px)` → 至少 400px，更宽的视口才按 20vw 长 |
| `clamp(最小值, 首选值, 最大值)` | 首选值在区间内就用它，越界则夹到边界 | `font-size: clamp(1rem, 2.5vw, 2rem)` |

- `min()` / `max()` 接受**一个或多个**逗号分隔的表达式（可以有超过两个参数，即多条约束）；`clamp()` **恰好三个**参数。
- **`clamp(MIN, VAL, MAX)` 的解析结果就是 `max(MIN, min(VAL, MAX))`** —— 这是 MDN 在 “Return value” 一节给出的解析规则，不是类比。
- 返回值类型跟随参数：`<length>` / `<percentage>` / `<number>` / `<angle>` / `<time>` / `<frequency>` / `<integer>`，所以它们**能用在任何接受这些类型的地方**，不限于尺寸。
- 数值算例（`min`）：`width: min(50vw, 200px)` —— 视口宽 < 400px 时 `50vw` 更小，取 `50vw`；视口 ≥ 400px 时取 `200px`，即**上限 200px**。（MDN `min()` 的官方例子）

### 与 `calc()` 的关系（最常见的误解）
MDN 原文：参数是完整的数学表达式，「you can use addition, subtraction, multiplication, and division **without using the `calc()` function itself**」。所以**不必再套一层 `calc()`**：

```css
/* ✅ 里面直接写算式 */
font-size: clamp(1rem, 2.5vw, 2rem);
padding: clamp(12px, 3vw, 32px);

/* 也可以混着写（clamp 内嵌 calc 完全合法；下面这行是 MDN 官方示例原样） */
width: clamp(100px, calc(30% / 2rem + 10px), 900px);
```

- 职责不同：**`calc()` 只会算**（加减乘除），**`min()` / `max()` / `clamp()` 只会比大小、夹区间**；三者可以互相嵌套（`clamp()` 里能放 `min()` / `max()`）。
- 每个参数可以用**不同单位**（上面 `1rem` 与 `2.5vw` 就是混用）。
- `+` / `-` 运算符**两侧必须有空格**（`-` 不带空格会被当成负号）。
- ⚠️ 已知例外：用在表格列 / 列组 / 行 / 行组 / 单元格的 width/height 上时，这类表达式 **可能被当作 `auto` 处理**（MDN 在三页里都重复了这条注意事项）。

### 典型用法：流体排版与流体间距
```css
/* 正文：下限保可读、上限防失控，中间随视口 */
body { font-size: clamp(16px, 4vw, 24px); }

/* 间距同理：小屏紧凑、大屏舒展，但两端都有边界 */
.section { padding: clamp(12px, 3vw, 32px); }

/* 内容列：窄屏留白、宽屏封顶 */
.container { width: min(90vw, 1200px); margin-inline: auto; }
```

MDN 对「用 `clamp()` 做字号」的评价：能让 font-size 随视口增长，但不会低于最小值、也不会高于最大值；**效果等同于经典的 Fluid Typography 写法，却只用一行，而且不用媒体查询**。

**这正是 [[vw 移动端适配]] 里那条结论的解法**：那张卡说「字号不要全交给 vw」，因为视口单位让文字无法被用户放大（WCAG F94）。`clamp()` 给字号加了下限与上限，让它在「可读」与「不失控」之间插值，而不是无条件等比缩放。

### 它为什么算「无断点」方案
| | 媒体查询（见 [[媒体查询]]） | 数学函数 |
| --- | --- | --- |
| 取值方式 | **分档**：跨过断点才换一套值 | **连续插值**：任何视口宽度都有对应值 |
| 视觉表现 | 换挡瞬间会「跳」一下 | 平滑变化，没有跳变 |
| 擅长对象 | 整块布局（换列数、换方向、显示/隐藏） | **单个长度值**（字号、间距、宽度、圆角） |
| 代价 | 断点数量随设计稿增长 | 上下限要自己定；表达式可读性差一些 |

两者**不是替代关系**：复杂布局仍要断点，数学函数擅长把「一堆只为字号 / 间距写的断点」压掉。

### 与本库两条既有结论的接缝
**① [[vw 移动端适配]] 的 `maxDisplayWidth` 就是靠这几个函数实现的。** 该库文档原话：「桌面端访问的特性是通过 CSS 函数 `min()`、`max()`、`calc()` 实现的」，并同时提醒 —— **有老旧设备投放场景就要关掉它**（数学函数在旧浏览器里会让整条声明失效）。它内置的运行时行内样式转换函数也正是这个形状：

```ts
const cssF = n > 0 ? 'min' : 'max'
return `${cssF}(${vwN}vw, ${maxN}px)`   // 例如 min(20vw, 120px)
```

→ 所以 `maxDisplayWidth` 不是插件发明的黑魔法，它就是本卡的 `min()` 封顶。

**② [[rem 适配]] 里「rem 的优点是能被 `clamp` 全局拦截」** —— 对根字号用 `clamp`，就能一键限定整页缩放区间：

```css
html { font-size: clamp(1rem, 2.5vw, 2rem); }
```

⚠️ **这里有一个必须知道的细节**：MDN 的 `rem` 条目写明 —— 「**When used within the root element `font-size`, it represents its initial value**」，并补充「常见浏览器默认是 `16px`，但**用户自定义偏好会改变它**」。

→ 也就是写在根字号那一行里的 `1rem` / `2rem`，**不是**引用它自己刚算出来的字号，而是引用**初始字号**（浏览器默认值，或用户偏好改过的值）。

→ **推论（由上面两条 MDN 陈述推出，MDN 未如此表述）**：把上下限写成 `rem` 而不是 `px`，能让缩放区间**跟随用户的字号偏好** —— 用户把默认字号调大，整页缩放的上限也随之抬高。这比用 `px` 写死上下限更照顾可访问性。

### 浏览器支持
| 函数 | Chrome | Firefox | Safari |
| --- | --- | --- | --- |
| `min()` | 79 | 75 | **11.1** |
| `max()` | 79 | 75 | **11.1** |
| `clamp()` | 79 | 75 | **13.1** |

（起始版本取自 MDN 官方 BCD 数据 `mdn/browser-compat-data`；Edge / Chrome Android / Safari iOS / Samsung Internet / Opera 在该数据里标注为 `mirror`，即跟随对应桌面实现。MDN 页面同时把三者标为 Baseline **Widely available**。）

注意 `clamp()` 的 Safari 门槛（13.1）比 `min()` / `max()`（11.1）更高：**两者不是同一批可用的特性**。[[vw 移动端适配]] 的 `maxDisplayWidth` 走的是 `min()` / `max()`，门槛是 11.1；VueZone 文档提示的「老旧设备」风险，指的就是不支持这些数学函数的浏览器会让**整条声明失效**（而不是降级成相邻的可用值）。

## 我的理解 / 类比
- `clamp()` 是**两端带挡板的弹簧**：中间自由伸缩，撞到挡板就停 —— 所以它「连续」而不「跳变」；媒体查询是**换挡**，不到换挡点不动，一换挡整块样式瞬变。想让单个尺寸平滑变化用弹簧，想让整块布局换形态用换挡。
- `calc()` 是**计算器**（只会算），`min()` / `max()` / `clamp()` 是**裁判**（只会比大小、夹区间）。记住这条，就不会再写出 `clamp(calc(1rem), calc(2.5vw), calc(2rem))` 这种多此一举的写法。

## 关联
- [[vw 移动端适配]] —— 上游机制：那张卡的 `maxDisplayWidth` 桌面封顶，底层就是这里的 `min()` / `max()` / `calc()`
- [[rem 适配]] —— 对根字号用 `clamp()` 即可一键限定整页缩放区间（注意「`rem` 在根字号里取初始值」那条细节）
- [[媒体查询]] —— 对照关系：媒体查询是分档（离散跳变），数学函数是连续插值；两者互补而非替代（在原子化框架里写断点则是 [[断点变体]]）
- [[移动端适配]] —— 归属：本卡是该框架卡「② 流式 / 弹性」那一档里 `min` / `max` / `clamp` 的具体机制
