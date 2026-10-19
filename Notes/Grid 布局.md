---
type: concept
tags: [CSS, grid, 布局]
created: 2026-10-17
aliases: [CSS Grid, Grid, 网格布局, grid layout, fr 单位, grid-template-columns]
source: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout · https://drafts.csswg.org/css-grid-1/ · https://drafts.csswg.org/css-grid-2/ · https://drafts.csswg.org/css-align-3/ · https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/grid-template-areas · https://webkit.org/blog/17660/introducing-css-grid-lanes/
---
# Grid 布局

## 一句话定义
Grid 是 CSS 的**二维**布局模型：先在容器上画好「**网格线 + 轨道**」的坐标系（行与列**同时**确定），再把子元素放进由线围出的网格单元 / 区域里。

## 要点

### 与 Flex 的分工（第一落点）
|       | [[Flex 布局]]                 | Grid                               |
| ----- | --------------------------- | ---------------------------------- |
| 维度    | 一维：内容先流动，只调它在轴上的位置          | 二维：先定义整张网，内容再往里放                   |
| 谁决定结构 | 内容（主轴长度靠内容撑）                | **作者**（轨道由 `grid-template-*` 显式定义） |
| 心智模型  | 一条流水线：排队 + 分配剩余空间           | **棋盘**：先画线，后落子                     |
| 擅长    | 一排 / 一列内部怎么摆（导航、标签、toolbar） | 分区版面（整页骨架、卡片墙、表单行列、日历）             |

```css
.layout {
  display: grid;              /* 块级网格容器；inline-grid 为行内级 */
  grid-template-columns: 200px 1fr 1fr;
  grid-template-rows: 60px 1fr;
  gap: 16px;
}
```

### 坐标系：线与轨道交替
- **N 条轨道 ↔ N+1 条线**；轨道是线与线之间的空间，线是轨道的边界
- 线可命名：`grid-template-columns: [sidebar-start] 200px [sidebar-end main-start] 1fr [main-end]`，之后 `grid-column: main-start / main-end` 可直接引用
- `repeat()` 内可放命名线：`repeat(4, [col] 250px [gap])`，同名线随重复展开

### 尺寸：`fr` 与尺寸函数
- **`fr` = 弹性系数**，只存在于 grid，表示分配剩余空间时的份额（`1fr 2fr` 即 1:2）
- ⚠️ 关键细节：`fr` 写在 `minmax()` **之外**时隐含 **`minmax(auto, <flex>)`**，那个 `auto` 下界是 **min-content 尺寸**——所以 `1fr` 轨道可能被内容撑破、不肯收缩；要能缩到 0 需写 `minmax(0, 1fr)`（`1fr` 溢出最常见的原因）
- `minmax(min, max)` 划区间；**max < min 时按 min 处理**
- `fit-content(<长度>)` ≈ `min(max-content, max(auto, 参数))`
- `repeat(auto-fill, minmax(250px, 1fr))`：无断点响应式卡片墙的经典写法

### auto-fill 与 auto-fit 的区别
`auto-fit` 的语义 = 「同 `auto-fill`，但**空的重复轨道会被折叠**」。
- `auto-fill`：空轨道**保留**，且空轨道也参与 `1fr` 伸展分配——所以元素少时不会占满整行
- `auto-fit`：空轨道**折叠为 0 宽**，已存在的轨道独占全部空间——只有一个元素时会铺满整行
- 边界：只有轨道上限带 `1fr`（如 `minmax(250px, 1fr)`）时才会「撑满」；写成 `minmax(250px, 250px)` 则两者都是 250px

### 放置：显式落子、自动填充与隐式网格
- 三种写法：**线号**（`grid-column: 1 / 3`，跨 2 列）、**span**（`grid-column: span 2`）、**区域名**
- `grid-template-areas` 的每个命名区域**必须是矩形**，否则整条声明无效；`.`（一个或多个点）表示空单元
- 整数 **0 非法**；**负整数从显式网格末端倒数**，`-1` = **显式网格**的最后一条线（隐式轨道不计入）
- **隐式网格**：元素落到未显式定义的行 / 列上时，浏览器自动补轨道，尺寸由 `grid-auto-rows` / `grid-auto-columns` 决定（默认 `auto`，按内容）
- `grid-auto-flow` 默认 `row`；`dense` 会**回填空洞**，使视觉顺序偏离 DOM 顺序，有可访问性代价
- 顺序：显式落子的元素先放，其余再自动放置

### 对齐：二维的九宫格
| 属性组 | 作用对象 | inline 轴 | block 轴 | 简写 |
| --- | --- | --- | --- | --- |
| 容器 → 每个 item 在其格内 | 网格单元内对齐 | `justify-items` | `align-items` | `place-items` |
| item → 自己 | 单个 item 偏离默认 | `justify-self` | `align-self` | `place-self` |
| 容器 → 整组轨道 | 轨道整体在容器内的分配 | `justify-content` | `align-content` | `place-content` |

- 经典居中：`place-items: center`（item 在其格内居中）；`place-content: center` 是让**整张网**在容器内居中，两者不通用
- ⚠️ **存疑（默认值）**：规范中 `align-items` 初始值 `normal`、`justify-items` 初始值 `legacy`、`justify-content` / `align-content` 初始值 `normal`（css-align-3 §5–§7）；grid 中 `normal` 行为近似 `stretch`。MDN 的「Box alignment in grid layout」指南正文写初始值为 `stretch`，与规范及 MDN 自己的属性页不一致——**以规范为准**

### 间距与重叠
- `gap` / `row-gap` / `column-gap` 是**容器属性**，作用于**轨道之间**（语义上像一条空轨道），**不是 item 的 margin**；`grid-gap` 为旧别名
- **多个 item 可放进同一网格单元并重叠**（默认 `stretch` 使它们叠在一起）；绘制顺序受 `z-index` 影响，grid item 上 `z-index` ≠ `auto` 会**创建层叠上下文**

### 两个 Level 2 / Level 3 特性
- **`subgrid`**（Level 2）：`grid-template-columns: subgrid` 让子网格继承父网格轨道，解决嵌套网格的列对齐。Firefox 71+、Safari / iOS 16+、Chrome / Edge 117+；MDN 标 Baseline **widely available**（2023-09）
- **Grid lanes（原 masonry）**：⚠️ `grid-template-rows: masonry` 是**旧语法**；CSSWG 2025-12 决议改名，现写法为 **`display: grid-lanes`**（WebKit 博客《Introducing CSS Grid Lanes》2025-12-19）。规范仍为 Editor's / Working Draft，**未进 stable**；仅 Safari 26.4+ / iOS 26.4+ 默认启用，Chrome / Edge 与 Firefox 默认关闭（flag），MDN 标 **Limited availability / Experimental，非 Baseline**——**不要用于生产**

## 我的理解 / 类比
- **Grid 是「先砌墙再摆家具」，Flex 是「先摆家具再调间距」。** 砌墙（定义轨道）时不用想里面放什么；摆家具（flex 流动）时是内容先到场，你再调间距。
- **线是墙，轨道是房间**：房间总比墙少一间——`repeat(3, 1fr)` 是 3 个房间、4 面墙。`1fr` 说的是「房间怎么分空间」，不是「一个家具占一间房」；「一元素占一格」是**自动放置**（默认 `span 1`）在起作用，把 `1fr` 换成 `100px`，落点完全不变。
- **隐式网格是「放不下就加盖」**：你只画了一层的墙，第 4 个元素一来，浏览器自动盖第二层——所以**没写 `grid-template-rows` 也照样能排**。
- **`auto-fill` 是「空房间也留着」，`auto-fit` 是「空房间拆掉」**：家具少时，前者让家具和看不见的空房间一起分空间，后者让剩下的家具把整层楼占满。

## 关联
- [[Flex 布局]] —— 最直接对照：一维 vs 二维；两者的对齐属性（`justify-*` / `align-*`）与九宫格模型同源，Grid 多出 `*-items` / `*-self` / `*-content` 三层分工
- [[CSS 数学函数]] —— 搭档：`repeat(auto-fit, minmax(...))` 里的 `minmax` 与 `min()` / `max()` / `clamp()` 常一起构成无断点流体网格
- [[容器查询]] —— 接缝：该卡的 `inline-size` 包含坑指出「尺寸必须由外部给出（width / flex / grid）」，网格自动列正是可能塌缩的场景之一
- [[移动端适配]] —— 归属：本卡是该框架卡「③ 策略」层里「Flex / Grid」的具体机制
