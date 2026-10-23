---
type: concept
tags: [vue, 组件通信]
aliases: [Vue slots, slot, 插槽, v-slot, 具名插槽, 作用域插槽, scoped slot]
source: https://vuejs.org/guide/components/slots.html · https://vuejs.org/api/built-in-directives.html#v-slot · https://vuejs.org/glossary/#scoped-slot · https://v3-migration.vuejs.org/breaking-changes/slots-unification · https://github.com/vuejs/core/blob/v3.5.44/packages/compiler-core/src/transforms/vSlot.ts
created: 2026-10-17
---
# Vue 插槽

## 一句话定义
把**一段模板片段**从父组件投递到子组件的指定位置渲染（内容分发）；同时子组件能在投递点用 `:` 绑定自己的数据反向交给父组件，由父组件决定「这段内容长什么样」——这个反传就是**作用域插槽**。

## 要点

### 插槽内容与插槽出口
```vue
<!-- 父组件：写在标签之间的即「插槽内容」 -->
<FancyButton>点我</FancyButton>

<!-- 子组件 FancyButton：<slot> 是「插槽出口」 -->
<button class="fancy-btn"><slot></slot></button>
```
- `<slot>` 元素标出了插槽内容**将被渲染在哪**；子组件负责外壳与样式，内部的片段由父组件提供
- 内容不限于文本，可以是任意合法模板内容（多元素、其他组件）
- 出口**之间**写的内容是**默认内容**（fallback），仅在父组件没提供内容时渲染：`<slot>Submit</slot>`

### 渲染作用域：插槽内容属于父组件（本卡最关键的约束）
- 插槽内容**能访问父组件的数据**，因为它在父组件模板里定义
- 插槽内容**无法访问子组件的数据**。官方原文：*"Slot content does **not** have access to the child component's data."*
- 根因是词法作用域：**父模板中的表达式只能访问父作用域，子模板中的表达式只能访问子作用域**
- ⚠️ 推论：子组件的 `<style scoped>` **不会**作用于插槽内容（它不属于子组件的渲染作用域）；要给插槽内容写样式，就在父组件里写，或在子组件里用 `:deep()`

### 作用域插槽：`:` 在出口绑定，`v-slot` 在父组件接收
这就是「通过 `:` 向父组件传数据」的完整形态 —— 在 `<slot>` 上用 `v-bind` 绑定的属性叫 **slot props**：

```vue
<!-- 子组件 TodoList：把内部数据绑到出口上 -->
<ul>
  <li v-for="(todo, i) in todos" :key="todo.id">
    <slot :item="todo" :index="i">{{ todo.text }}</slot>
  </li>
</ul>
```

```vue
<!-- 父组件：用 v-slot 的指令值接收这个 props 对象 -->
<TodoList :todos="todos" v-slot="slotProps">
  <span>{{ slotProps.index }}. {{ slotProps.item.text }}</span>
</TodoList>
```
- `v-slot` 的值就是**你给形参起的名字**，随便叫 `slotProps` / `receivedProps` 都行
- 只有默认插槽有内容、且不需要 `v-slot` 的参数名时，才可以**直接写在组件标签上**；一旦混用具名插槽，就必须对所有插槽统一使用显式 `<template>`，否则编译报错

### 具名插槽
多个出口用 `name` 区分，**没有 `name` 的出口隐式命名为 `default`**：

```vue
<!-- 子组件 BaseLayout -->
<header><slot name="header"></slot></header>
<main><slot></slot></main>

<!-- 父组件：v-slot 的参数就是插槽名，简写为 # -->
<BaseLayout>
  <template #header="headerProps">{{ headerProps.title }}</template>
  <!-- 顶层非 <template> 节点隐式属于默认插槽 -->
  <p>正文</p>
</BaseLayout>
```
- 同时接收默认插槽与具名插槽时，**所有顶层非 `<template>` 节点都被隐式视为默认插槽的内容**
- 插槽名可以动态化：`<template v-slot:[dynamicSlotName]>`
- ⚠️ 细节：`<slot name="header" message="hello" />` 得到的 props 是 `{ message: 'hello' }` —— **`name` 不会进入 slot props**，它是保留字段
- 官方 API 对 `v-slot` 的适用范围写的是 `<template>` **和组件**（用于独占默认插槽带 props 的情况），所以「`#` 只能写在 `<template>` 上」并不准确：**带参数名的写法必须用 `<template>`；无参数写法只在独占默认插槽时能写在组件上**

### 解构、重命名与默认值
`v-slot` 的值按**函数形参位置**解析，因此 JavaScript 形参能写的东西它都能写：

```vue
<ChildComponent v-slot="{ text, count }">...</ChildComponent>
<ChildComponent v-slot="{ item: thing }">...</ChildComponent>        <!-- 重命名 -->
<ChildComponent v-slot="{ item = '默认值' }">...</ChildComponent>    <!-- 解构默认值 -->
```
- 解构有官方文档与示例
- 解构**默认值**：Vue 3 文档只显式演示了解构，未演示默认值；依据是官方 Vue 2 文档给过默认值示例 `v-slot="{ user = { firstName: 'Guest' } }"`，且官方源码把 `v-slot` 的表达式按函数形参解析（`ExpParseMode.Params`）

### 机制：插槽内容就是一个函数
官方原话：*"You can think of a scoped slot as a function being passed into the child component. The child component then calls it, passing props as arguments."*

```js
// 父组件 <TodoList v-slot="slotProps">…</TodoList> 的插槽内容，编译成：
{ default: (slotProps) => [ /* 插槽里的模板 */ ] }

// 子组件 <slot :item="todo"> 编译成：调用它并传参
slots.default({ item: todo })
```
- 源码佐证：`compiler-core` 的 `transforms/vSlot.ts` 里 `buildClientSlotFn` 用 `createFunctionExpression(props, children, false, /* isSlot */ true)` 生成插槽函数
- 这就解释了为什么 `v-slot` 能解构、能写默认值 —— 它本来就在形参位置
- **Vue 3 中「作用域插槽」不再是独立机制**：官方术语表写明 Vue 3 把所有插槽统一成「可接收 props 的插槽函数」，`scoped slot` 这个词仅作为「带 props 的插槽」这一描述性术语保留。默认插槽、具名插槽**同样可以传 props**
- 因此 `$slots` 上每个插槽都是**函数**：`this.$slots.default()`；`$scopedSlots` 已移除并合并进 `$slots`（迁移指南标为 BREAKING）
- 条件插槽可结合 `$slots` 与 `v-if` 判断内容是否存在

### 边界与版本
- **Vue 2 的 `slot-scope` / `scope` 属性在 Vue 3 已彻底移除**，只支持 `v-slot`（官方 Vue 2 文档明说其最终会在 Vue 3 移除；3.5.x 的 compiler-core / runtime-core 源码中已无 `slot-scope` / `scopedSlots` 兼容处理）
- 子组件若**没有** `<slot>` 出口，父组件写在标签之间的内容会被直接丢弃
- TS 下声明插槽名与 slot props 的类型用 `defineSlots`（编译器宏，见 [[Vue 编译器宏]]）

## 我的理解 / 类比
插槽像**点外卖时自带的餐盒**：子组件提供厨房和摆盘（外壳、样式），你（父组件）带内容来放。但一开始餐盒是**不透明**的 —— 你放进去的菜用的是你自己厨房的食材（父作用域），看不到后厨有什么。

`v-slot` 就是子组件在餐盒上开的一个**小窗口 + 一张标签**：子组件在窗口递出「这一份是什么」（`slot props`），你在标签上写个名字（`slotProps`）才能接住它。不写标签名，递出来的东西就落不到你手上 —— 因为你的模板只会去**你自己的作用域**里找那个名字。

理解「插槽内容本质是函数」插槽片段其实是**一个待调用的函数**，子组件的 `<slot :item="todo">` 就是**调用它并把 `todo` 作为实参**。参数名在形参位置定，所以能解构、能带默认值。

## 关联
- [[Vue 组件 props]] —— 相邻且互补：props 是「父向子传数据」（数据在父、消费在子），插槽是「子向父交数据 + 父决定渲染」（数据在子、渲染在父）；两者共同构成组件通信的向下与反传两条线，本卡不重复 props 机制
- [[Vue 编译器宏]] —— 支撑关系：`defineSlots` 是声明插槽名与 slot props 类型的宏，`useSlots` 是运行时取值函数；宏的编译期/运行时边界看那张卡
- [[编译时与运行时]] —— 原理支撑：`v-slot` 能解构与默认值，靠的是编译期按函数形参解析，而非运行时特判
- [[Vue 透传属性attrs]] —— 边界相邻：插槽决定「内容片段」放哪渲染，`$attrs` 决定「属性」落到哪个元素，是插槽内容之外「属性如何穿透组件」的另一条独立路径
