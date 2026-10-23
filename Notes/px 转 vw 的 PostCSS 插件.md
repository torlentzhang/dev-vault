---
type: concept
tags: [CSS, PostCSS, mobile, 前端工程化]
created: 2026-10-16
aliases: [px 转 vw 插件, px-to-viewport, postcss-px-to-viewport, postcss-px-to-viewport-8-plugin, postcss-mobile-forever]
source: https://github.com/evrone/postcss-px-to-viewport · https://github.com/lkxian888/postcss-px-to-viewport-8-plugin · https://github.com/wswmsword/postcss-mobile-forever · https://www.npmjs.com/package/postcss-px-to-viewport-8-plugin · https://postcss.org/docs/writing-a-postcss-plugin
---
# px 转 vw 的 PostCSS 插件

## 一句话定义
挂在 PostCSS 管道上的构建期转换器：按 `viewportWidth` 指定的**设计稿宽度**，把 CSS 里的 `px` 声明改写成 `vw` —— [[vw 移动端适配]] 那套方案能落地，靠的就是它。

## 用于解决什么问题
靠人手换算时，375 稿上每个尺寸都要开计算器除一遍再写成 vw，设计稿一改就得全站重算；改用运行时脚本换算，则要接受首屏尺寸跳动和额外的运行时开销。

## 要点

### 生态三代，只选后两代
| 包 | 最新版 / 发布 | 定位 |
| --- | --- | --- |
| `postcss-px-to-viewport` | 1.1.1 / 2019-07-09 | 初代（evrone），事实停更 |
| `postcss-px-to-viewport-8-plugin` | 1.2.5 / 2024-03-27 | 社区 fork（lkxian888），把初代移植到 PostCSS 8 |
| `postcss-mobile-forever` | 5.0.0 / 2025-03-25 | 新一代（wswmsword），转换 + 一揽子封顶 |

月下载量量级（npm registry，2026-09-13 ~ 10-12）：`postcss-pxtorem` ≈ 108 万 > 初代 ≈ 2.5 万 > fork ≈ 2.1 万 > `postcss-mobile-forever` ≈ 7.7k。

- **新项目二选一**：只做「px→vw 一件事」用 fork；要「vw + 封顶 + 桌面端」一体化用 mobile-forever。
- **初代不要新用**（原因见下）。
- 走 `rem` 路线是另一个包：`postcss-pxtorem`，见 [[rem 适配]]。

### 换算公式
```
vw 值 = px 值 ÷ viewportWidth × 100
```
- `viewportWidth` **必须等于设计稿宽度**：375 稿填 375，750 稿填 750。稿是 750 却填 375，全站尺寸会系统性放大一倍。
- 算例：375 稿 `width: 75px` → `20vw`；750 稿 `font-size: 32px` → `4.26667vw`（= 375 CSS px 视口上的 16 CSS px）。
- 完整的三层单位换算（设计稿 px → CSS px → 物理像素）见 [[vw 移动端适配]]。

### 初代与 fork 的配置项（默认值取 fork 发布产物 `lib/index.js`）
| 选项 | 默认值 | 说明 |
| --- | --- | --- |
| `viewportWidth` | `320` | ⚠️ 必须改；⚠️ mobile-forever 的默认是 `750`，两代别混 |
| `unitPrecision` | `5` | 保留几位小数；mobile-forever 默认 `3` |
| `propList` | `['*']` | 通配符是 `*`；**fork README 里写作 `_`，与其实现不符**（实现判断的是 `*`） |
| `viewportUnit` / `fontViewportUnit` | `'vw'` | 字号走独立选项，可只把字号留在 `px` |
| `selectorBlackList` | `[]` | 字符串是「包含即匹配」：`['body']` 会命中 `.body-class` |
| `minPixelValue` | `1` | `≤1` 的值不转换，1px 边框因此保住（原因见 [[1px 边框]]） |
| `mediaQuery` | `false` | ⚠️ `@media` 里的 px 默认**不转换** |
| `replace` | `true` | 直接改属性值；`false` 则插入一条回退声明 |
| `exclude` | `undefined` | 想跳过 node_modules 里的组件库得自己传 |
| `landscape` / `landscapeWidth` | `false` / `568` | 额外产出 `@media (orientation: landscape)` |

```ts
// vite.config.ts（最小配置）
import pxToViewport from 'postcss-px-to-viewport-8-plugin'

postcss: {
  plugins: [
    pxToViewport({
      viewportWidth: 375,          // 必须等于设计稿宽度
      selectorBlackList: ['van-'], // 需要保留 px 的选择器（第三方组件库常用）
      exclude: [/node_modules/],
    }),
  ],
}
```

- 忽略单条声明的注释：`/* px-to-viewport-ignore */`（同行）、`/* px-to-viewport-ignore-next */`（下一行）；**mobile-forever 用的是 `/* mobile-ignore */` / `/* mobile-ignore-next */`**，名字可在 `comment` 配置里改。
- 转换正则自带保护：**引号字符串与 `url(...)` 不会被转换**（`getUnitRegexp` 把这两种情形排除在外）。

### 初代为什么不能新用
`postcss-px-to-viewport@1.1.1` 用的是 PostCSS 7 时代的 `postcss.plugin()` 工厂写法；PostCSS 8 换成「插件对象声明 `postcssPlugin` + 按节点类型注册 visitor」的新 API，旧写法被保留成兼容壳，**能跑但会刷弃用警告**（fork 的 README「问题」一节引的就是那条警告）。能跑 ≠ 没欠债。

### fork 相对初代改了什么（读 1.2.5 发布产物）
- 改成 PostCSS 8 原生插件写法（`postcssPlugin: 'postcss-px-to-viewport'` + `Once` / `OnceExit`）。
- `viewportWidth` / `landscapeWidth` 支持传函数，可按文件路径返回不同设计稿宽度（例如 `file => file.includes('vant') ? 375 : 750`）。
- 横屏规则挪到 `OnceExit` 再追加，修掉「生产环境竖屏样式覆盖横屏」的顺序问题。
- ⚠️ **README 与 `types.d.ts` 都声明了 `include`，但 1.2.5 的实现里没有任何地方读它** —— 想让某些文件走不同宽度，用 `viewportWidth` 函数，别指望 `include`。

### mobile-forever：把「封顶」也做进插件
同一个包的四种模式（README 原文表述）：

| 模式 | 激活方式 | 实现手段 |
| --- | --- | --- |
| `vw-mode` | 只设 `viewportWidth` | 纯 `px→vw`，不封顶 |
| `max-vw-mode` | 再加 `maxDisplayWidth` | `min(vw, px)` + `appSelector` 居中 |
| `mq-mode` | 设 `viewportWidth` + `enableMediaQuery` | `@media` 区分桌面端与横屏，产包较大 |
| `rem-mode` | `mobileUnit: 'rem'` 或 `basicRemWidth` | 根字号 `vw` + `@media` 配合 `rem` |

```ts
// 最小可用（max-vw-mode）
mobile({ viewportWidth: 375, appSelector: '#app', maxDisplayWidth: 600 })
```
- 产出形如 `height: min(9.6vw, 53.76px)`，`#app` 上补 `max-width` + 左右 `auto` —— 因此**样式文件里必须存在 `#app {}` 这样的空规则**，否则该模式不生效（README 明写）。
- 额外能力：默认矫正 `position: fixed` 元素的包含块（`rootContainingBlockSelectorList` / `appContainingBlock`）、支持逻辑属性转换。
- 作者自己在 README 顶部挂了大字警告：**不限制最大宽度的 vw / rem 伸缩视图会破坏浏览器缩放，存在可访问性问题**，并建议优先考虑响应式设计 —— 这段立场与 [[vw 移动端适配]] 的 WCAG 一节互证。

### 这类插件共同的边界
- **只管构建期 CSS**：行内 `style="width: 100px"` 转不了，要另写运行时换算。
- **只换单位**：不做弹性、不设断点，「适配」全部来自等比缩放本身。
- **不封顶**：裸 vw 在桌面端会把整页撑到不可用，必须配 `maxDisplayWidth` 或自己包 `min()`。
- **第三方组件库要单独处理**：`selectorBlackList` / `exclude` / `rootContainingBlockSelectorList`，或先 rem→px 再转 vw（原子化 CSS 默认产出 rem）。

## 我的理解 / 类比
它是「**单位翻译器**」而不是「布局方案」：输入设计稿语言（px），输出屏幕比例语言（vw），中间那把比例尺就是 `viewportWidth`。所以配置里最要命的从来不是插件选项有多少，而是**比例尺有没有量错稿子**。三代插件的演进也很直白——初代把翻译做出来了，fork 把 API 搬到新框架上，mobile-forever 则承认了「纯比例语言在桌面端说不通」，于是把封顶一起塞进了翻译器。

## 关联
- [[vw 移动端适配]] —— 原理与实践：那张讲为什么要 px→vw（含 WCAG 代价与选型判断），本卡讲实现它的插件
- [[PostCSS]] —— 容器：本卡这类插件挂在它的插件管道上
- [[rem 适配]] —— 同一思路的另一条路线（px→rem），那张记 `postcss-pxtorem` 的机制与默认值
- [[移动端适配]] —— 上游选型：插件只在「④ vw 等比缩放 / ⑤ 混合」方案下才登场
