---
type: concept
tags: [CSS, PostCSS, mobile, 视口单位, 响应式]
created: 2026-10-14
aliases: [vw 适配, 视口适配, px-to-viewport, postcss-px-to-viewport, postcss-px-to-viewport-8-plugin, postcss-mobile-forever, 移动端等比缩放]
source: https://github.com/evrone/postcss-px-to-viewport · https://github.com/wswmsword/postcss-mobile-forever · https://www.npmjs.com/package/postcss-px-to-viewport-8-plugin · https://www.w3.org/WAI/WCAG21/Techniques/failures/F94 · https://github.com/amfe/lib-flexible · https://vue-zone.github.io/docs/vue3-vant-mobile/browser-adaptation
---
# vw 移动端适配

## 一句话定义
vw 移动端适配是指：按设计稿 1:1 写 `px`，构建期由 PostCSS 插件按设计稿宽度把 `px` 换算成 `vw`，让整页随视口宽度等比缩放，从而在任意宽度的手机上还原设计稿比例。

## 要点

### 换算只有一个公式，但设计稿 px ≠ 屏幕 px
```
vw 值 = px 值 ÷ viewportWidth × 100
```

| 设计稿 | 稿上写的 | 编译结果（unitPrecision 5） | 375 CSS px 视口上渲染 |
| --- | --- | --- | --- |
| 375（@1x 稿） | `width: 75px` | `20vw` | 75 CSS px |
| 750（@2x 稿） | `font-size: 32px` | `4.26667vw` | **16 CSS px**（DPR 2 → 32 物理像素） |

- **750 宽的设计稿是 @2x 稿**（给 375 CSS px 屏幕画的两倍图），所以稿上 32px 落到真机就是 16 CSS px —— 「比稿子小一半」正是精确还原，不是缩水。
- 记牢三层单位：**设计稿 px → CSS px（÷ 倍图倍数）→ 物理像素（× DPR）**。混层是这类适配最高频的错。
- 配置铁律：`viewportWidth` 必须等于设计稿宽度，代码里照抄稿上的数字；稿是 750 却把值填 375，全站尺寸会系统性放大一倍。

### 关键配置项与它们的坑
```ts
// vite.config.ts
import pxToViewport from 'postcss-px-to-viewport-8-plugin'

postcss: {
  plugins: [
    pxToViewport({
      viewportWidth: 375,          // 必须等于设计稿宽度
      unitPrecision: 5,
      fontViewportUnit: 'vw',      // ⚠️ 字号也转 vw 是无障碍问题的源头（见「代价」）
      selectorBlackList: ['van-'], // 需要保留 px 的选择器（第三方组件库常用）
      minPixelValue: 1,            // 1px 不转换，保住 1px 边框
      mediaQuery: false,           // ⚠️ 媒体查询里的 px 默认不转换
      exclude: [/node_modules/],
    }),
  ],
}
```

默认值（原版与 fork 一致）：`unitToConvert: 'px'`、`viewportWidth: 320`、`unitPrecision: 5`、`propList: ['*']`、`viewportUnit` / `fontViewportUnit: 'vw'`、`minPixelValue: 1`、`mediaQuery: false`、`replace: true`。

其它边界：
- **只作用于构建期的 CSS**：行内 `style="width: 100px"` 转不了，要另写运行时换算（vue3-vant-mobile 模板的 `inline-px-to-vw.ts` 就是手算 `min(xvw, 上限px)`）。
- **单条声明可豁免**：`/* px-to-viewport-ignore */`（同行）、`/* px-to-viewport-ignore-next */`（下一行）。
- **它不动布局逻辑**：只换单位，不做弹性、不设断点，「适配」全部来自等比缩放本身。

### 生态现状：初代已停更，换 fork 或新一代插件
| 包                                 | 最新版 / 发布时间         | 状态                       |
| --------------------------------- | ------------------ | ------------------------ |
| `postcss-px-to-viewport`          | 1.1.1 / 2019-07-09 | ❌ 事实停更；PostCSS 8 下刷弃用警告  |
| `postcss-px-to-viewport-8-plugin` | 1.2.5 / 2024-03-27 | ✅ 社区 fork，为 PostCSS 8 而生 |
| `postcss-mobile-forever`          | 5.0.0 / 2025-03-25 | ✅ 活跃，额外解决桌面端拉伸           |
| `postcss-pxtorem`                 | 6.1.0 / 2024-01-20 | 另一条路线（rem），非本卡主题         |

（下载量取自 npm registry，只统计官方源、镜像站安装通常不计入，只看量级。）

- **初代为什么不能用**：它用的是 PostCSS 7 时代的 `postcss.plugin()` 工厂写法。PostCSS 8 换了插件 API（插件自己声明 `postcssPlugin` + 用 visitor 按节点类型注册回调），旧写法被保留成一个**只警告一次的兼容壳** —— 所以它**还能跑，但会刷 `postcss.plugin was deprecated`**，且拿不到新 API 的遍历优化。**能跑 ≠ 没欠债。**
- **rem + flexible 路线已被官方劝退**：`amfe/lib-flexible` README 原话——「由于 `viewport` 单位得到众多浏览器的兼容，`lib-flexible` 这个过渡方案已经可以放弃使用……建议大家开始使用 `viewport` 来替代」。所以 vw 不是新潮尝试，它是 rem 的继任者。

### 现代形态：vw 要封顶，字号要兜底
裸的等比缩放在桌面端会把整页撑到不可用，所以现在的用法都是「vw + 上限」：

```ts
// vue3-vant-mobile 模板的 postcss.config.ts（模板已内置）
'postcss-mobile-forever': {
  appSelector: '#app',
  viewportWidth: 375,          // Vant 设计稿宽度
  maxDisplayWidth: 600,        // 视口单位封顶，桌面端不再无限放大
  rootContainingBlockSelectorList: ['.van-tabbar', '.van-popup', /* … */], // fixed 元素修正
}
```

`maxDisplayWidth` 是用 `min()` / `max()` / `calc()` 实现的：移动端等比放大，超过 600px 就封顶不再长。

### 代价：拿视口单位做字号会踩 WCAG
[WCAG 2.1 技术 F94](https://www.w3.org/WAI/WCAG21/Techniques/failures/F94) 的标题就是 *Failure of Success Criterion 1.4.4 due to incorrect use of viewport units to resize text*，描述原文：

> As these units are relative to the viewport, it means they cannot be resized by zooming or adjusting text-size.

原因很直观：字号写 `4vw`，用户放大 200% 时视口 CSS 宽度减半，一乘一除，**物理尺寸原地不动**。所以：

- **布局宽高用 vw 问题不大，字号用 vw 是真问题**。务实做法是字号留在 px / rem，或用 `clamp()` 设上下限（F94 也注明：若用媒体查询在不同屏幕调整字号，可能不构成失败）。
- 其它成本：第三方组件（尤其 `position: fixed` 的弹层、tabbar）要单独进黑名单或 `rootContainingBlockSelectorList`；原子化 CSS 默认产出 `rem`，需先 `@unocss/preset-rem-to-px` 变回 px 再转 vw。

### 它还是主流吗
**分两层看，混着答就会错：**

| 层面 | 结论 | 依据 |
| --- | --- | --- |
| 国内单端移动 H5（设计稿驱动） | ✅ 仍是常见默认方案 | 新一代插件 2025–2026 仍在维护；移动端模板把它设为默认（vue3-vant-mobile 文档原话：「默认采用 vw 方案」） |
| 整个 Web / 多端项目 | ❌ 不是通用主流答案 | 通用主流是响应式设计：媒体查询 + Flex / Grid + `clamp()`，不整体缩放 |

| 场景 | 合适做法 |
| --- | --- |
| 营销活动页 / 单端 H5 / 视觉稿严格 1:1 还原 | px→vw，配 `maxDisplayWidth` + 字号兜底 ✅ |
| 中后台、PC + 移动同构、组件库 | 响应式 + 断点，别做全局 vw 缩放 ❌ |
| 长期演进的 App 内嵌 H5 | 响应式为主 + 局部 vw / `clamp()` ⚠️ |

> 这一节是**归纳判断**（依据列中的事实已核实），不是规范结论。

## 我的理解 / 类比
vw 适配像把整页当成**一张矢量图**整体缩放：所以它在任何屏幕上都能还原设计稿比例——这正是它好用的原因；也正因为是「整体缩放」，图里的字**不会**因为用户想看清而单独变大，这是它无障碍上绕不过去的代价。再记一个锚：设计稿 px 是「图纸上的毫米」，vw 是「屏幕的百分比」，构建期插件就是那把自动换算的比例尺。

## 关联
- [[PostCSS]] —— 依赖关系：vw 适配是 PostCSS 插件管道上的一环（那张讲管道，本卡讲管道上挂的插件）
- [[User-Agent 与移动端切换]] —— 边界互补：那张收尾说「布局适配优先响应式」，本卡接着回答「那什么场景仍该用 vw」
- [[断点变体]] —— 对比关系：视口断点（条件生效）vs 整页等比缩放（无条件缩放），两种互斥的适配思路
