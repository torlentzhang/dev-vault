---
type: concept
tags: [CSS, PostCSS, mobile, rem]
created: 2026-10-14
aliases: [rem 适配, flexible, amfe-flexible, lib-flexible, postcss-pxtorem, 动态根字号, rem 布局]
source: https://github.com/amfe/lib-flexible/blob/2.0/index.js · https://github.com/amfe/lib-flexible/blob/master/src/flexible.js · https://github.com/cuth/postcss-pxtorem#readme · https://github.com/amfe/lib-flexible
---
# rem 适配

## 一句话定义
把根元素字号绑到视口宽（约定 `1rem = 视口宽 / 10`），再用 PostCSS 插件把设计稿 px 换成 rem，实现整页等比缩放。

## 用于解决什么问题
不做等比换算，同一份设计稿要在各种宽度手机上成立，只能为每个尺寸段手写媒体查询、逐处调宽高字号，机型一多就维护不过来，比例也很难在各类屏幕上保持一致。

## 要点

### 机制：rem + 动态根字号
`rem` 是相对**根元素**（`html`）`font-size` 计算的长度 —— 根字号一动，全站所有 rem 值**同步**缩放。

`amfe-flexible@2.0` 的全部核心就这几行（摘自 [index.js](https://github.com/amfe/lib-flexible/blob/2.0/index.js)）：

```js
// set 1rem = viewWidth / 10
function setRemUnit () {
  var rem = docEl.clientWidth / 10
  docEl.style.fontSize = rem + 'px'
}
setRemUnit()
window.addEventListener('resize', setRemUnit)
window.addEventListener('pageshow', function (e) { if (e.persisted) setRemUnit() })
```

于是 `1rem` 恒等于**视口宽的 10%**：375 逻辑宽的视口上 `1rem = 37.5px`。

两个监听各自存在的理由：

| 监听 | 为什么需要 |
| --- | --- |
| `resize` | 旋转屏幕 / 改窗口尺寸后 `clientWidth` 已变，不重算布局就停留在旧比例 |
| `pageshow` + `e.persisted` | 从 **bfcache**（往返缓存）返回时页面是被"恢复"而非重新加载，视口可能已经变了，但**不一定会触发 `resize`**，所以要补一次重算 |

### 纯 CSS 版：连 JS 都不需要
```css
html { font-size: calc(100vw / 10); }
```

- 与上面的 JS 版**数值等价**（都是「视口宽 / 10」），且**视口变化由浏览器自动重算** —— `resize` 与 `pageshow` 两个补丁**全部消失**（在纯 CSS 版里它们是纯冗余）。
- 会丢掉的两样（都不属于「根字号计算」本身）：
  - `clientWidth` **不含**经典滚动条宽度，而 `100vw` **含** → 带经典滚动条的桌面环境下 `calc(100vw/10)` 略偏大，`100vw` 还容易撑出横向滚动条（见 [[视口单位]]）。
  - 运行时 API `rem2px()` / `px2rem()`（行内样式换算要用，classic 独有）与 DPR 标记（classic 的 `data-dpr` 属性、2.0 的 `.hairlines` 类）没有 CSS 对应物（见 [[1px 边框]]）。

### 构建期转换：postcss-pxtorem
```
rem 值 = px 值 ÷ rootValue
```

- 约定 **`rootValue = 设计稿宽 / 10`**：**750 稿 → `rootValue: 75`**。
- 算例：稿上 `100px` → `100 ÷ 75 = 1.3333rem` → 真机 `75 × 1.3333 = 100 物理 px = 50 逻辑 px`（@2x 稿的正确结果）。
- **默认值表（⚠️ 重点坑）** —— 取自 [README](https://github.com/cuth/postcss-pxtorem#readme)：

| 选项 | 默认值 | 说明 |
| --- | --- | --- |
| `rootValue` | `16` | 根字号，也支持传函数（按文件返回不同值） |
| `propList` | `['font', 'font-size', 'line-height', 'letter-spacing', 'word-spacing']` | ⚠️ **默认只转字体相关属性**，与 px-to-viewport 默认 `['*']` 不同 → 整页适配**必须显式写 `propList: ['*']`**，否则宽高边距全都不转 |
| `minPixelValue` | `0` | px-to-viewport 默认是 `1` |
| `mediaQuery` | `false` | 媒体查询里的 px 不转换 |
| `exclude` | `/node_modules/i` | px-to-viewport 默认不过滤 |
| `replace` | `true` | 直接替换属性值，不追加回退声明 |

- 豁免单条声明的小技巧：把单位写成**大写** —— `border: 1PX solid` 会被插件忽略，但浏览器仍接受（比加注释更轻）。

### classic 0.3.2 才解释得通的那些「怪现象」
`lib-flexible@0.3.2`（[src/flexible.js](https://github.com/amfe/lib-flexible/blob/master/src/flexible.js)）比 2.0 多做了四件事，那个年代的一堆配置噪音都能从这四件事解释：

**① 动态改写 `<meta viewport>`**

```js
scale = 1 / dpr
metaEl.setAttribute('content', 'initial-scale=' + scale + ', maximum-scale=' + scale +
  ', minimum-scale=' + scale + ', user-scalable=no')
```

→ 让 **1 CSS px = 1 物理像素**；iPhone 6（375 逻辑 / 750 物理 / dpr 2）上布局视口因此变成 **750**，这就是那个年代能直接用 750 稿标注的原因。源码自身的算术印证了这点：封顶判断写的是 `width / dpr > 540`，说明 `width` 拿到的是**物理宽度**。
→ 也意味着 classic 下 `1vw = 7.5 物理 px`，与今天 `width=device-width` 下 `1vw = 3.75 CSS px` 语义完全不同（见 [[meta viewport]]）。

**② `refreshRem` 的 540 封顶**

```js
var width = docEl.getBoundingClientRect().width;
if (width / dpr > 540) { width = 540 * dpr; }
var rem = width / 10;
```

→ 这是「等比缩放不能无限放大」**最早的工程答案**，与 [[vw 移动端适配]] 里的 `maxDisplayWidth: 600` 是同一件事。

**③ 正文字号钉死在 `12 × dpr`**：`doc.body.style.fontSize = 12 * dpr + 'px'` —— 在「1 CSS px = 1 物理像素」的前提下，视觉上正好 12pt，即**布局等比缩放、正文不跟着缩放**。

**④ `resize` 绑 300ms 防抖 + `pageshow` 的 `persisted`**，并把 `data-dpr` 写到 `<html>` 上、暴露 `rem2px()` / `px2rem()` API（于是 CSS 里能写 `[data-dpr="2"] ...`）。

### 与 vw 适配的关系
**rem 与 vw 本质是同一件事（整页等比缩放）的两种载体**，只差「缩放的载体」是根字号还是单位本身。

| 维度 | rem 适配 | vw 适配（[[vw 移动端适配]]） |
| --- | --- | --- |
| 缩放载体 | **根字号**（多一层间接） | 单位本身（直接） |
| 运行时依赖 | 需 JS 或 `calc()` 设定根字号 | 无 |
| 纯 CSS 可行 | ✅ `calc(100vw / 10)` | ✅ 天然 |
| 封顶 / 区间限制 | 容易：`clamp()` 改根字号即**全局生效** | 需逐值 `min()`，或插件支持 `maxDisplayWidth` |
| 行内样式转换 | 需运行时 `px2rem` / `rem2px` | 需运行时 `vw()` |
| 与原子化 CSS 契合度 | 原子化框架默认产出 rem，**天然契合**（反而是 vw 方案要先 rem→px 再转 vw） | 需 rem→px→vw 两段转换 |
| 兼容性 | rem 略好 | vw 更现代 |
| 官方态度 | **README 明确劝退** | 被推荐为继任方案 |

**但有一条边界**：**断点不受劫持影响**。媒体查询里的相对单位以**初始值**为基准，不受页面样式影响 —— Media Queries 4 §1.3 原话：`em` 是「relative to the initial value of `font-size` … **not any styling on the page**」。所以 flexible / `calc(100vw/10)` 改了根字号，**基于 `rem` 的断点（如 Tailwind v4 的 `sm: 40rem`）仍按浏览器默认字号换算**：断点与页面内的 rem 布局是两套互不干扰的尺子（见 [[媒体查询]]）。

### 已退场
lib-flexible README（[官方仓库](https://github.com/amfe/lib-flexible)）原话：

> 由于 `viewport` 单位得到众多浏览器的兼容，`lib-flexible` 这个过渡方案已经可以放弃使用，不管是现在的版本还是以前的版本，都存有一定的问题。建议大家开始使用 `viewport` 来替代此方案。

→ **vw 是 rem 的继任者，不是"新潮尝试"**；`rem + 动态根字号` 这一整套（含 flexible 本身）属于过渡方案。

## 我的理解 / 类比
- 根字号是「**整页缩放的总旋钮**」：拧一下，所有以 rem 量出来的长度一起变大变小。`vw` 则是「**每个长度自己带一把比例尺**」，不需要总旋钮。
- rem 多出的这一层间接，换来的是**可拦截性** —— 想给缩放设上下限，只要对那一个旋钮用 `clamp()`，全局生效；vw 得逐条值去限制。
- 代价来自同一处：多一个「总旋钮」就多一个**必须先被设置好**的依赖（JS 或 `calc`）；而这个旋钮本来握在用户手里，一旦被脚本接管，字号的可访问性优势也一起被接管了。

## 关联
- [[vw 移动端适配]] —— 同一件事的两种载体（根字号 vs 单位本身），那张卡记 vw 的换算与 WCAG 代价
- [[移动端适配]] —— 本卡是三层模型里「单位层」的一种选择
- [[meta viewport]] —— classic 方案的地基前提：动态改写 `initial-scale = 1/dpr` 才有「1 CSS px = 1 物理像素」
- [[PostCSS]] —— 构建期转换（`postcss-pxtorem`）所在的插件管道
- [[1px 边框]] —— DPR 探测那部分的去向（classic 的 `data-dpr` 属性、2.0 的 `.hairlines` 类）
