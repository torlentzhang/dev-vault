---
type: concept
tags: [CSS, mobile, web, 视口单位]
created: 2026-10-14
aliases: ["viewport meta", "布局视口", "width=device-width", "initial-scale", "viewport-fit"]
source: https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/meta/name/viewport · https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/length · https://github.com/amfe/lib-flexible/blob/master/src/flexible.js
---
# meta viewport

## 一句话定义
`<meta name="viewport">` 用来声明页面的**布局视口**宽度；而 `vw` 这类视口单位就是「布局视口的百分比」，所以它还顺带决定了这些单位实际等于多少。

## 要点

### 它做什么：把布局视口设为设备的 CSS 像素宽度
`width=device-width` 把**布局视口**宽度设为设备的 **CSS 像素宽度**（如 iPhone 的 375 / 393），**不是**物理像素宽度（750 / 1179）。

### 不写的后果
移动端会退回一个约 **980px 的虚拟视口**再整体缩小显示：页面看起来「能用」，但**[[媒体查询]]基本失效**，字号小到看不清。

### 反常识点：`initial-scale=1` 通常不必写
MDN 现行文档推荐的写法就是一行：

```html
<!-- ✅ 现行推荐：一行足够 -->
<meta name="viewport" content="width=device-width">
```

并明确说明 **`initial-scale=1` 通常不必写** —— 省略时由浏览器自行决定初始缩放；只有在**内容溢出导致「不希望的缩小」**时，才需要显式把它加上。

```html
<!-- ❌ 不要照抄的历史模板（lib-flexible 那个年代的写法） -->
<meta name="viewport"
      content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no">
```

上面这行里的三处缩放限制要分开看：`initial-scale=1` 只是「不必要」；而 **`maximum-scale=1` + `user-scalable=no` 是禁止页面缩放** —— WCAG 1.4.4（Resize Text）的测试程序把「浏览器的 zoom 功能」列为必须能把文字放大到 200% 的手段之一，所以禁用缩放属于无障碍欠债。

### `viewport-fit`：刘海屏的开关
| 取值 | 含义 |
| --- | --- |
| `auto` | 默认；不改变初始布局视口 |
| `contain` | 视口内接屏幕的最大矩形 |
| `cover` | 铺满整个屏幕；此时**安全区必须自己补**（见 [[安全区适配]]） |

### 与 vw 语义的关系（本卡与 [[vw 移动端适配]] 的接缝）
同样的 CSS，在两种 viewport 声明下各算一遍（iPhone 6：375 逻辑 pt = **750 物理 px**，dpr = 2）：

| | 现代 `width=device-width` | classic flexible `initial-scale=1/dpr` |
| --- | --- | --- |
| 布局视口宽度 | **375** CSS px | **750** CSS px（= 物理宽度） |
| 1 CSS px 等于 | 2 物理 px | 1 物理 px |
| `width: 4.26667vw` | 16 CSS px = **32 物理 px** | 32 CSS px = **32 物理 px** ✅ 物理尺寸相同 |
| `width: 100px` | 100 CSS px = 200 物理 px | 100 CSS px = 100 物理 px ❌ 缩了一半 |
| `@media (max-width: 480px)` | 命中（375 ≤ 480） | 不命中（750 > 480） ❌ 断点整体偏移 |

- **机制**：`vw` 是**布局视口**的百分比，视口本身变了它跟着变，所以**物理尺寸不变**；`px` 是绝对单位、媒体查询断点按 CSS px 判断，两者都被地基整体搬家。（这条规律跨全部适配方案，总述见 [[移动端适配]]）
- **结论**：**任何「px 换 vw 是多少」的答案，脱离 viewport 声明都是不完整的。** [[vw 移动端适配]] 卡里 `viewportWidth: 375` 的算例，前提正是本卡说的 `width=device-width`。
- （classic 那列「布局视口 = 物理宽度」是**由源码反推**：它判断是否封顶时写的是 `width / dpr > 540`，说明取到的 `width` 是物理宽度。）

## 我的理解 / 类比
viewport 声明是给浏览器的一张**坐标系地图**：`vw` / `rem` 是在地图上「按比例量距离」—— 地图整体换大换小，量出来的实际距离不变；`px` 则是地图上的「固定刻度」—— 地图一换，同一个刻度代表的实际距离就变了。所以换地基时，用比例单位写的东西安然无恙，用 px 写的东西集体搬家。

## 关联
- [[视口单位]] —— 分工：本卡定义「视口是什么、有多大」，那张讲 `vw` / `vh` / `dvh` / `svh` / `lvh` 等具体单位
- [[移动端适配]] —— 本卡是那条「地基 / 单位 / 策略」三层模型里的**地基层**
- [[vw 移动端适配]] —— 接缝：那张卡的 `px ÷ viewportWidth × 100` 换算，只有在 viewport 声明确定之后才成立
- [[安全区适配]] —— `viewport-fit=cover` 是那张卡的前置条件
