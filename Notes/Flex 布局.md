---
type: concept
tags: [css, flex, 布局]
created: 2026-10-09
aliases: [Flexbox, 弹性布局, 弹性盒子]
source: MDN · Flexbox https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_flexible_box_layout
---

# Flex 布局

## 一句话定义
Flex 是 CSS 的一维布局模型：以**主轴**与**交叉轴**两根轴组织子元素，所有对齐问题都能归进"轴 × 位置"的九宫格。

## 要点
- **两根轴**：主轴方向由 `flex-direction` 决定（默认 `row`，可 `column`，均可加 `-reverse`），交叉轴永远垂直于主轴
- **三个位置**：每根轴都有 `start / center / end`（flex 中写作 `flex-start / center / flex-end`），两轴组合成 3×3 九宫格——Flex 对齐的"格子"都在这张表里
- **两个核心属性**：
  - `justify-content` → 主轴对齐：`flex-start / center / flex-end / space-between / space-around / space-evenly`
  - `align-items` → 交叉轴对齐：`flex-start / center / flex-end / baseline / stretch`（默认 `stretch`）
- **经典居中**：水平垂直居中 = 九宫格的 (center, center) 格——`justify-content: center` + `align-items: center`
- `align-self` 让单个子元素脱离 `align-items`，单独占一个交叉轴位置
- `flex-start / flex-end` 是**相对轴向**而非屏幕方向：`row-reverse` 时 `flex-start` 在右侧
- `order` 可视觉重排子元素（注意会改变阅读顺序，有可访问性隐患）
- 其余属性都在为九宫格服务：`flex-wrap` 换行、`gap` 留缝、`align-content` 多行时交叉轴整体分布
- **空间分配**：`flex-grow / shrink / basis`（简写 `flex`）管主轴剩余空间；默认 `flex: 0 1 auto`——不 grow、可 shrink、基准 auto，且子项默认 `min-width: auto`（row 轴），不会缩到比内容更小

**九宫格示意**（默认 `row`，左上角为原点）：

```text
              flex-start       center       flex-end
              ┌────────────────┬────────────────┬────────────────┐
交叉轴 start  │ start, start   │ center, start  │ end, start     │
              ├────────────────┼────────────────┼────────────────┤
交叉轴 center │ start, center  │ center, center │ end, center    │
              ├────────────────┼────────────────┼────────────────┤
交叉轴 end    │ start, end     │ center, end    │ end, end       │
              └────────────────┴────────────────┴────────────────┘
```

- 每个格子 =（主轴位置， 交叉轴位置）；原点 = 左上角的 (start, start)
- 默认 row 下：主轴 start→end 即左→右，交叉轴 start→end 即上→下；`column` 时两轴旋转 90°
- 居中格 = (center, center)：`justify-content: center` + `align-items: center`

## 我的理解 / 类比
先定轴的方向（横排还是竖排），再在九宫格里选格子：`justify-content` 管"沿主轴站哪"，`align-items` 管"沿交叉轴站哪"。`space-*` 是主轴两端对齐的变体，`flex` 三件套决定主轴空间不够分时谁多谁少。

## 关联
- [[normalize与reset的区别]] —— 同属 CSS 基础与基线
- [[UnoCSS与Tailwind的定位差异]] —— 原子化 CSS 里 `justify-center`、`items-center` 等工具类直接映射九宫格的两个核心属性
