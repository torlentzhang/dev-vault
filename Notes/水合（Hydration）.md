---
type: concept
tags: [渲染, ssr]
created: 2026-11-17
aliases: [hydration, 水合, 激活, 客户端激活]
source: https://vite.dev/guide/ssr · https://react.dev/reference/react-dom/client/hydrateRoot · https://vuejs.org/guide/scaling-up/ssr
---
# 水合（Hydration）

## 一句话定义
水合是客户端接管服务端渲染产物的过程：在**已有的静态 HTML** 上重新跑一遍应用代码，把组件逻辑、事件监听与状态挂到已存在的 DOM 节点上，使其变为可交互应用。

## 用于解决什么问题
服务端渲染产出的是静态 HTML，看得见但点不动。水合补上「可交互」这一段，同时不重建 DOM，避免内容闪烁与滚动位置、输入焦点的丢失。

## 要点

### 认领已有节点，不是重新创建
水合的前提是容器里**已经有服务端渲染的 HTML**；它做的是匹配，不是新建。（Vue 文档表述：「将每个组件与它应该控制的 DOM 节点相匹配，并添加 DOM 事件监听器」；React 文档表述：「attach React to existing HTML」—「take over managing the DOM inside it」。）

因此「服务端渲染 + 水合」与「客户端渲染」在 DOM 层面的差别是：前者复用节点，后者创建节点。

### 入口必须区分「接管」与「新建」
各框架都为这两件事提供不同入口，用错不报错，只是把服务端成果丢掉：

| 框架 | 接管已有 HTML | 新建 DOM |
| --- | --- | --- |
| Vue | `createSSRApp()` | `createApp()` |
| React | `hydrateRoot()` | `createRoot()` |

React 文档对边界写得更明确：服务端渲染的应用「using `createRoot()` is not supported. Use `hydrateRoot()` instead」；而 `createRoot()` 的首次 `root.render`「will clear all the existing HTML content inside the React root before rendering」。误用的后果也点明了——更慢，且**重置焦点与滚动位置、丢失用户输入**。

Vue 侧对应的是 `createSSRApp()` 与 `createApp()`：前者假定 HTML 已预渲染并执行激活，后者按常规挂载处理。

### 前提：两端输出必须一致
水合是逐节点比对，服务端与客户端必须产出相同结构、相同文本。不一致即**水合不匹配（hydration mismatch）**。

两个框架列出的成因不同，合起来更完整：

| 成因 | 出处 |
| --- | --- |
| HTML 结构不合法被浏览器解析纠正（如 `<p><div>` 被拆开） | Vue |
| 渲染数据含随机值 | Vue |
| 两端时区不同 | Vue |
| 根节点内 HTML 周围的空白 / 换行 | React |
| 渲染逻辑里用 `typeof window !== 'undefined'` 这类判断 | React |
| 渲染逻辑里用浏览器专有 API | React |
| 两端渲染了不同数据 | React |

### 不匹配要当 bug 修，不当配置问题绕
两个框架的态度有差异，但方向一致：

- Vue：会自动尝试恢复并调整 DOM 以匹配客户端状态，代价是丢弃节点、渲染新的，性能有损失
- React：明确要求「you should treat mismatches as bugs and fix them」；最坏情况是**事件监听器挂到了错误的元素上**

两者都提供按元素静音的逃生门（Vue 3.5+ 的 `data-allow-mismatch`、React 的 `suppressHydrationWarning`），定位是例外手段而非常规解法。

### 事件监听为何只能在这一步挂上
服务端只有 HTML，没有可执行逻辑。事件处理器是函数，不是能写进 HTML 属性的值（HTML 属性只能是字符串），所以服务端产物里不存在监听器；页面此时「看得见、点不动」。

水合阶段客户端代码重新执行，才有机会把监听器绑到已存在的节点上。这也是「SSR 让首屏更快，但可交互时间不变」的直接原因：HTML 一到即可显示，监听器要等 JS 下载、执行、水合完成。

## 我的理解 / 类比
服务端渲染像印刷厂印出一张成品海报，字是死的；水合像装修工拿着同一份图纸进场，在海报对应位置把按钮接上电线。工人没有重画海报——所以不闪、不丢焦点。

## 关联
- [[服务端渲染]] —— 水合是 SSR 链路的后半段，本卡展开其中「核心是水合」那一句
- [[Vue 组件事件]] —— 被水合补上的监听器，来源就是模板里声明的事件
- [[Vue Teleport]] —— SSR 下避免 `to="body"`，因为 `<body>` 已装载服务端内容、无法确定水合的起始位置
