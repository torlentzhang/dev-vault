---
type: concept
tags: [CSS, mobile, DPR, 物理像素, hairlines]
created: 2026-10-14
aliases: [1px 边框, hairlines, hairline, devicePixelRatio, data-dpr, 物理像素, 设备像素]
source: https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/length · https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/border-width · https://drafts.csswg.org/css-transforms-1/ · https://github.com/amfe/lib-flexible/blob/2.0/index.js
---
# 1px 边框

## 一句话定义
你写的 `border: 1px` 里的 `1` 是 **CSS 像素**，不是屏幕上的点。DPR 2 的屏上它会渲染成 **2 个物理像素**，比 @2x 设计稿里那个「1px」（= 1 个物理像素）粗一倍——这个落差是「1px 边框」这个问题的全部起点。

## 要点

### 为什么 1px 会变粗：CSS 像素 ≠ 物理像素
| 概念 | 含义 | iPhone 6 |
| --- | --- | --- |
| CSS 像素 | 写样式、做布局用的逻辑单位 | 视口宽 375 |
| 物理像素 | 屏幕上真实发光的点 | 宽 750 |
| DPR（`devicePixelRatio`） | 物理像素 ÷ CSS 像素 | 2 |

```
物理像素 = CSS px × DPR
```

- JS 里读 `window.devicePixelRatio`。
- 750 设计稿之所以是「@2x 稿」，就是这个等式倒过来用（见 [[vw 移动端适配]] 的换算一节）。

### 更关键的一层：`border-width` 会被取整为整数设备像素（MDN 原文）
MDN `<length>` 页「Specified vs. computed values」一节明确：对 `border-width`、`outline-width`、`column-rule-width`、`outline-offset` 这类属性，**计算值会被取整为整数个设备像素**，以保证合理的视觉呈现：

- **非零、小于 1 设备像素**的值 → **向上取整为 1** 个设备像素；
- **大于 1 设备像素**的值 → **向下取整到最近的整数设备像素**。

MDN 原例（DPR 3 的屏幕）：

| 声明 | 设备像素 | 取整 | 计算值 |
| --- | --- | --- | --- |
| `border-width: 1.5px` | 4.5 | 向下 → 4 | 约 `1.33px` |
| `outline-width: 0.2px` | 0.6 | 向上 → 1 | 约 `0.33px` |

由这条规则直接得到两个结论：

- **`1px` 在 DPR 2 上是 2 个设备像素**，在 DPR 3 上是 3 个。
- **边框不会因为「太小」而消失**：只要值非零，小于 1 设备像素会被**向上取整为 1**，最细就是 1 个设备像素。边框真的消失，只可能是计算值本身为 0（如 `border-width: 0`、`border-style: none`，或构建期把值算成了 0）——这与「取整为 0」是两件事。

### 三条路：怎么让线真的细到 1 个设备像素
| 路线 | 做法 | 在链条的哪一环解决 | 现状 |
| --- | --- | --- | --- |
| ① 半像素值 | 直接写 `border-width: 0.5px` | 声明值本身 | 需浏览器支持，要探测 |
| ② 伪元素 + `transform: scale` | 画 1px 边框再缩 0.5 倍 | **渲染阶段** | ✅ 今天主流 |
| ③ 改写地基 | `initial-scale = 1/dpr`，让 1 CSS px = 1 物理像素 | 重定义 CSS 像素 | ❌ 已退场 |

### 路线①：半像素值 + 能力探测
`0.5px` 在 DPR 2 上正好等于 1 个设备像素（**前提：浏览器支持半像素值**）。amfe-flexible 2.0 的做法是**探而不是猜**——塞一个 `0.5px` 透明边框的测试元素进 DOM，量它的 `offsetHeight`：

```js
// detect 0.5px supports —— amfe-flexible 2.0 index.js
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

渲染出 1 个设备像素（布局高度就是 1）就说明支持，给 `<html>` 挂上 `hairlines` 类，CSS 里据此启用更细的边框：

```css
.hairlines .hairline { border-width: 0.5px; }
```

> 归属说明：`hairlines` 探测是 **amfe-flexible 2.0** 才有的；classic 0.3.2 的 `flexible.js` 里没有这段。

### 路线②：伪元素 + `transform: scale`（推荐）
画一条正常的 1px 边框，再把它整体压扁一半：

```css
.hairline::after {
  content: '';
  position: absolute;
  box-sizing: border-box;
  border-bottom: 1px solid #ddd;
  transform: scaleY(0.5);   /* DPR 2：2 设备像素 × 0.5 = 1 个物理像素 */
}
```

**为什么它绕得过去**：取整规则只作用于 `border-width` 这类属性的 **computed value**；而 transform 是在元素**尺寸与定位完成之后**才应用的渲染阶段变换——W3C CSS Transforms Level 1 §1.1 原文：these effects are applied ***after elements have been sized and positioned***。既然取整发生在它之前，缩放就不再经过那道门槛。

> ⚠️ 官方文本**没有直写**「scale 不受 border-width 取整约束」这句话，上述结论是由「取整属 computed value 阶段」+「transform 在其后应用」两条官方陈述**推出**的，非官方原句。

**代价（有官方出处）**：transform 非 `none` 时会创建**层叠上下文**，并成为其内部 `position: fixed` / `absolute` 后代的**包含块**；它不影响布局，只影响溢出区域。

**组件库怎么选**（据各自官方发布物，两家路线并不一致）：

- **Vant 4 用路线②**：`.van-cell:after{ …; border-bottom:1px solid var(--van-cell-border-color); transform:scaleY(.5) }`；`.van-divider--hairline` 竖线用 `scaleX(.5)`。
- **Ant Design Mobile v5 用路线①**：`border-bottom: solid var(--adm-card-header-border-width, 0.5px)`，另有 `div.adm-px-tester` 做 DPR 探测。

### 路线③：改写地基（已退场，仅作背景）
classic `lib-flexible` 把 `<meta viewport>` 的 `initial-scale` 设为 `1/dpr`，让 1 CSS 像素正好铺满 1 个物理像素，于是 `1px` **天然就是最细的 1 个设备像素**，不需要任何半像素技巧。代价是布局视口宽度变成物理宽度（iPhone 6 上 375 → 750）。它同时把 DPR 写到 `<html data-dpr="2">` 上，好让 CSS 能按倍数分别写样式——这就是 `data-dpr` 属性与 `.hairlines` 类的来源。

这套地基前提（含为什么被劝退）见 [[rem 适配]]，本卡不再展开。

### 与 vw 适配的交界：`minPixelValue: 1`
px→vw 插件默认 `minPixelValue: 1`，让 `1px` **不参与 px→vw 转换**。必要性正来自上面的取整规则：一旦 `1px` 被换算成 vw，它的有效宽度就随视口浮动，再叠加设备像素取整，边框会在**不同屏幕 / DPR 上时粗时细**（注意：是时粗时细，**不会消失**）。配置细节见 [[px 转 vw 的 PostCSS 插件]]。

## 我的理解 / 类比
`border-width` 像**按整数块砖砌墙**：浏览器只会用整数个设备像素砌，半块砖要么不要、要么补成一整块——这正是「非零且小于 1 设备像素时向上取整」的含义。
- 砖越小（DPR 越高），你对「1px」的手感就越失真：在 DPR 2 的屏上，你以为砌了 1 块，其实砌了 2 块，墙自然比图纸厚一倍。
- 想砌得更细有三条路：**换更小的砖**（路线①，还得先探测浏览器给不给）、**砌完再整体压扁**（路线②，不动砖的规格，直接绕过砌墙规则）、或者**把「砖」的规格改掉**（路线③，已退场）。

## 关联
- [[rem 适配]] —— 路线③（classic 改写地基）的完整出处与退场原因，本卡只保留结论
- [[px 转 vw 的 PostCSS 插件]] —— `minPixelValue: 1` 保护 1px 不被转成 vw 的具体配置
- [[移动端适配]] —— 本卡属于总览里「DPR 维度」那条与尺寸缩放正交的线
- [[视口单位]] —— `vw` 的物理尺寸都建立在「1 CSS px = DPR 个物理像素」这个前提上
