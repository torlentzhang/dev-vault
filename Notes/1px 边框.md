---
type: concept
tags: [CSS, mobile, DPR, 物理像素]
created: 2026-10-14
aliases: [1px 边框, hairlines, hairline, data-dpr, devicePixelRatio, 物理像素, 设备像素]
source: https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/length · https://github.com/amfe/lib-flexible/blob/2.0/index.js · https://github.com/amfe/lib-flexible/blob/master/src/flexible.js
---
# 1px 边框

## 一句话定义
你写下的 `border: 1px` 在屏幕上到底多细，**不取决于那个 `1`**，而取决于两件事：设备的 **DPR**（物理像素 ÷ CSS 像素），以及浏览器对 `border-width` 这类长度的**设备像素取整规则**。

## 要点

### 前置：CSS 像素、物理（设备）像素、DPR
| 概念 | 含义 | iPhone 6 |
| --- | --- | --- |
| CSS 像素（CSS px） | 写样式、做布局用的逻辑单位 | 视口宽 375 |
| 物理（设备）像素 | 屏幕上真实发光的点 | 宽 750 |
| DPR（`devicePixelRatio`） | 物理像素 ÷ CSS 像素 | 2 |

```
物理像素 = CSS px × DPR
```

- JS 里读 `window.devicePixelRatio`；两个 flexible 脚本都读了它（classic 用 `win.devicePixelRatio`，2.0 用 `window.devicePixelRatio || 1`）。
- 750 设计稿之所以是「@2x 稿」，就是这个等式倒过来用（见 [[vw 移动端适配]] 的换算一节）。

### 取整规则（已核实，MDN 原文）
MDN 的 `<length>` 页在「Specified vs. computed values」一节明确：对 `border-width`、`outline-width`、`column-rule-width`、`outline-offset` 这类属性，**计算值会被取整为整数个设备像素**，以保证合理的视觉呈现：

- **非零、小于 1 设备像素**的值 → **向上取整为 1** 个设备像素；
- **大于 1 设备像素**的值 → **向下取整到最近的整数设备像素**。

MDN 原例（DPR 3 的屏幕）：

| 声明 | 设备像素 | 取整 | 计算值 |
| --- | --- | --- | --- |
| `border-width: 1.5px` | 4.5 | 向下 → 4 | 约 `1.33px` |
| `outline-width: 0.2px` | 0.6 | 向上 → 1 | 约 `0.33px` |

套到最常见的三种情况（由上面的等式直接得出，非 MDN 原文）：

| 声明 | 设备像素 | 说明 |
| --- | --- | --- |
| `border-width: 1px` @ DPR 2 | 2 | 比设计稿的「1 物理像素」粗一倍 |
| `border-width: 1px` @ DPR 3 | 3 | 更粗 |
| `border-width: 0.5px` @ DPR 2 | 1 | 正好 1 个设备像素（**前提：浏览器支持半像素值**） |

也就是说：如果设计稿上的「1px」指的是 **1 个物理像素**（@2x 稿的常见约定），那么 `border-width: 1px` 在 DPR 2 上渲染出来是 **2 个物理像素**，比稿子粗一倍。想让线真的细到 1 个设备像素，只有两条路 —— **用浏览器支持的半像素值**（如 `0.5px`），或者**改写地基，让 1 CSS px 就等于 1 物理像素**（下节）。

### classic flexible 是怎么处理的（源码证据）

**① 改地基：把视口缩放成「1 CSS px = 1 物理像素」** —— classic `src/flexible.js`

```js
scale = 1 / dpr
metaEl.setAttribute('content', 'initial-scale=' + scale + ', maximum-scale=' + scale +
  ', minimum-scale=' + scale + ', user-scalable=no')
```

`initial-scale = 1/dpr` 之后，1 个 CSS 像素正好铺满 1 个物理像素，于是 `1px` **天然就是最细的 1 个设备像素**，不需要任何半像素技巧。代价是布局视口宽度变成物理宽度（iPhone 6 上 375 → 750），这正是 [[rem 适配]] 讲的整套地基前提。
（细节：classic 只在识别为 iPhone 时按 `devicePixelRatio` 取 2 或 3，其他设备一律 `dpr = 1`，即不做缩放。）

**② 探测 0.5px 支持** —— amfe-flexible 2.0 `index.js`

```js
// detect 0.5px supports
if (dpr >= 2) {
  var fakeBody = document.createElement('body')
  var testElement = document.createElement('div')
  testElement.style.border = '.5px solid transparent'
  fakeBody.appendChild(testElement)
  docEl.appendChild(fakeBody)
  if (testElement.offsetHeight === 1) {
    docEl.classList.add('hairlines')
  }
  docEl.removeChild(fakeBody)
}
```

做法是**探而不是猜**：塞一个 `0.5px` 透明边框的测试元素进 DOM，量它的 `offsetHeight`；只要它渲染出了 1 个设备像素（布局高度就是 1），就给 `<html>` 挂上 `hairlines` 类，CSS 里据此启用更细的边框样式。

> 归属说明：`hairlines` 探测是 **amfe-flexible 2.0** 才有的；classic 0.3.2 的 `flexible.js` 里没有这段（已通读全文确认）。

**③ 把 DPR 暴露给 CSS** —— classic `src/flexible.js`

```js
docEl.setAttribute('data-dpr', dpr)
```

于是 CSS 能按 DPR、或按「浏览器有没有半像素能力」分别写（**以下为示意写法，不是 flexible 源码里的样式**）：

```css
/* 静态 DPR 标记：DPR 2 的屏，用 0.5px 去凑 1 个设备像素 */
[data-dpr="2"] .hairline { border-width: 0.5px; }

/* 能力标记：只有上面探测通过、浏览器真支持 0.5px 时才启用 */
.hairlines .hairline { border-width: 0.5px; }
```

两者的信息类型不同：`data-dpr` 说的是「**这块屏是多少倍**」（静态事实），`hairlines` 说的是「**这个浏览器画不画得出半像素**」（能力探测）。要哪种回退策略，取决于你更怕哪一种失败。

> ⚠️ **未核实范围**：社区里另有多套 1px 方案（如 `transform: scale()`、`box-shadow`、渐变背景、`border-image` 等），本卡**未核实这些做法**，故不展开。本卡只讲机制，以及上面这套有源码依据的路线。

### 为什么这件事和「缩放方案」绑在一起
同一道题（「1px 到底多细」），两条路线用了完全不同的手段 —— 这是本卡与 [[vw 移动端适配]] 的接缝：

| 路线 | 手段 | 在链条的哪一环解决 |
| --- | --- | --- |
| vw 适配 | 插件用 **`minPixelValue: 1`** 让 `1px` **不参与 px→vw 转换** | **转换阶段**：保护 1px，不让它变成小数 vw |
| classic flexible | **改写视口缩放**，让 1 CSS px = 1 物理像素 | **地基阶段**：重定义 CSS 像素本身 |

`minPixelValue` 的必要性正来自上面的取整规则：如果 `1px` 也被换成 vw，它会变成一个小数视口单位，再被浏览器按设备像素取整 —— 边框就会在**不同 DPR 上时粗时细，甚至整条消失**。

## 我的理解 / 类比
`border-width` 像**按整数块砖砌墙**：浏览器只会用整数个设备像素砌，半块砖要么不要、要么补成一整块 —— 这正是「非零且小于 1 设备像素时向上取整」的含义。
- 砖越小（DPR 越高），你对「1px」的手感就越失真：在 DPR 2 的屏上，你以为砌了 1 块，其实砌了 2 块，墙自然比图纸厚一倍。
- 想砌得更细只有两个办法：**换更小的砖**（半像素值，还得先探测浏览器给不给），或者**把「砖」的规格改掉**（缩放视口，让 1 块砖就等于 1 个设备像素）—— 后者就是 classic flexible 干的事。

## 关联
- [[rem 适配]] —— 这套 DPR 标记是 flexible 系列脚本留下的（`data-dpr` 来自 classic 0.3.2，`hairlines` 来自 amfe-flexible 2.0），本卡是它的下游细节
- [[vw 移动端适配]] —— `minPixelValue: 1` 让 1px 不参与转换，正是为了躲开本卡的取整规则
- [[移动端适配]] —— 这属于总览里「DPR 维度」那条正交的线：与尺寸缩放无关，要单独处理
- [[视口单位]] —— `vw` / `dvh` 的物理尺寸都建立在「1 CSS px = DPR 个物理像素」这个前提上
