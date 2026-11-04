---
type: concept
tags: [vue, 浮层]
aliases: [Teleport, 传送, 传送门, vue teleport]
source: https://vuejs.org/guide/built-ins/teleport.html · https://vuejs.org/api/built-in-components.html · https://vuejs.org/guide/scaling-up/ssr.html
created: 2026-10-28
---
# Vue Teleport

## 一句话定义
`<Teleport>` 是 Vue 3 的**内置组件**（无需注册），它只改变内容**渲染到哪个 DOM 节点**，不改变内容在**组件树里的位置**——props、事件、provide / inject 全部仍按原来的父子关系走。

## 用于解决什么问题
弹窗写在某个卡片组件内部，而那张卡片有 `overflow: hidden`，或它的祖先带 `transform`，`position: fixed` 的弹窗就会被裁掉、定位基准跑偏、或被后面的内容盖住。这时把 `z-index` 调到 9999 也没用——**这不是层级数字不够大，是它生在了错误的 DOM 位置**。

`<Teleport>` 就是把「挂到哪儿」这件事**声明化**的手段：模板照旧写在原组件里，逻辑一行不改，只让最终产物落到别的地方。

## 要点

### 基本用法：`to` 收两种值
```vue
<Teleport to="#modals">      <!-- CSS 选择器字符串 -->
<Teleport :to="targetEl">    <!-- 实际 DOM 元素，常配合模板 ref -->
```
官方原文：`to` 期望一个 **CSS 选择器字符串**或一个**实际的 DOM 节点**。

### 关键性质：DOM 变了，逻辑没变
这是理解 Teleport 的地基，其余所有行为都从它长出来。

| 维度 | 是否变化 |
| --- | --- |
| 在 DOM 里的位置 | **变** —— 内容渲染进 `to` 指定的容器 |
| 在组件树里的位置 | **不变** —— 仍是原父组件的逻辑子节点 |
| props / 事件 | 不变，按原父子关系传递与派发 |
| `provide` / `inject` | 不变，**按原组件树解析**，与目标容器所在位置无关 |
| Vue Devtools 中的嵌套 | 不变，仍显示在原父组件之下 |

官方措辞是 "only alters the rendered DOM structure — it does not affect the logical hierarchy of the components"，并明确补了一句「父组件的 injection 照常生效」。

> 因为走的是**组件树**而不是 DOM 树，所以把内容传送出去**不会**丢掉祖先的注入。想不通「注入为什么还活着」时，回到这张表。

### `disabled`：留在原地，不是「关掉」
```vue
<Teleport to="body" :disabled="isMobile">
```
`disabled` 为 `true` 时，内容**留在原始位置**——既不移进目标容器，也**不隐藏、不销毁**。

典型用法就是上面这一行：桌面端渲染成浮层，移动端渲染成行内组件，一套模板两种形态。

注意别拿它当开关用：要隐藏弹窗请用 `v-if`，`disabled` 只管「搬不搬」。

### 多个 Teleport 指向同一目标：顺次追加，不覆盖
按挂载顺序往后排。A、B 两个实例都指向 `#modals`，得到的是：

```html
<div id="modals"><div>A</div><div>B</div></div>
```

谁也不会盖掉谁。实务含义：**后挂载的自然在上面**——多个弹窗叠加时你想要的顺序，它已经替你保证了。

### 目标必须挂载时就存在（最常见的坑）
`to` 指向的元素，在 `<Teleport>` 挂载的那一刻**必须已经在 DOM 里**。如果目标本身也是 Vue 渲染的，就得保证它**先于** Teleport 挂载。

**Vue 3.5+** 提供了 `defer` prop，把目标解析推迟到应用其它部分挂载之后（语义类似 `mounted` 钩子）。但它不是万能药：目标仍需与 Teleport 在**同一个挂载 / 更新周期**内出现，晚一步照样失败。

排查顺序：先确认那个元素此刻**在不在** DOM 里（`document.querySelector('#modal-root')` 一眼就知道）。在，是时序问题，考虑 `defer`；不在，才轮到拼错或没渲染。

### SSR 下要避开 `to="body"`
官方在 SSR 章节专门给 Teleport 写了一节，三点：

1. **被传送的内容不会出现在服务端渲染出的字符串里**
2. 必须水合它时，内容要放进 SSR context 的 `ctx.teleports`（形如 `{ '#teleported': '...' }`），再自行注入到最终页面 HTML 的正确位置
3. **SSR 下避免 `to="body"`**：`<body>` 通常已经装了别的服务端渲染内容，Teleport 无法确定可靠的水合起点。改用专属容器，如 `<div id="teleported"></div>`

### 与 `<Transition>` 组合：官方没有规定嵌套顺序
官方只说明二者可以组合做出带过渡动画的模态框，**从未规定谁包谁**。官方 modal 示例里的写法是 `<Teleport to="body">` 在外、`<Transition>` 在该模态组件自己的模板内部——那是**示例惯例，不是规范**。

所以「必须 Teleport 在 Transition 外层」这类说法没有官方出处，照示例写可以，但别把它当成规则去纠正别人。

### 什么时候不该用它
- **只是想盖住别人**：先查父级是不是有 `overflow` / `transform` / 层叠上下文。父级不裁你，就不必传送——Teleport 不是 `z-index` 的替代品。
- **内容需要参与原父级的布局**（比如 `flex` 里的一项）：传送出去它就不在原布局里了。
- **需要浏览器原生浮层能力时**：`<dialog>`、popover 各有自己的语义与能力，Teleport 不替代它们。

## 我的理解 / 类比

像**编制和工位分家**：你的人事关系还挂在原部门（组件树位置），但工位搬到了别层楼（DOM 位置）。报销、审批、汇报都还走原部门那条线——这就是 props、事件、注入照常生效的原因；只有「坐哪儿」变了。

被裁切、被盖住的那些麻烦，全都是**工位环境**的问题，跟编制毫无关系。所以调 `z-index` 是在错的地方使劲：**`z-index` 管前后，`overflow` 管里外，而 Teleport 换的是里外**。

「目标必须先存在」这条，则像**搬家前得先有新房**：钥匙拿到了、箱子封好了，可新地址还没盖好，车就没地方卸货——`defer` 只是让你晚点发车，新房不会因为你多等就自动出现。

## 关联
- [[Vue 透传属性attrs]] —— 同族对照：两卡都在讲「产物落在哪儿」，`attrs` 管**属性**落到哪个元素、Teleport 管**节点**落到哪棵子树
- [[Vue 插槽]] —— 常配合使用：浮层内容通常经插槽传入，Teleport 搬的正是同一段模板片段
- [[1px 边框]] —— 成因前提：`transform` 会创建层叠上下文并成为 `position: fixed` 后代的**包含块**，正是弹窗错位的源头之一
- [[声明式与命令式编程]] —— 范式对照：手动 `appendChild` 是命令式地指定挂载点，Teleport 把同一件事声明化
- [[浮层的分类与命名]] —— 前置入口：先按行为确认要做的是哪一类浮层，再谈它的挂载位置
- [[Vue 组件通信方式]] —— 边界关系：Teleport 改变的是渲染位置而非数据通道，不属于那张选型表里的任何一种通信方式
