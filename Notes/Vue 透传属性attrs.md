---
type: concept
tags: [vue, 组件通信]
aliases: [Vue attrs, $attrs, useAttrs, inheritAttrs, fallthrough attributes, 透传属性, 属性透传, 非 prop 属性]
source: https://vuejs.org/guide/components/attrs.html · https://vuejs.org/api/composition-api-helpers.html · https://vuejs.org/api/sfc-script-setup.html · https://vuejs.org/guide/components/props.html · https://vuejs.org/guide/components/events.html · https://v3-migration.vuejs.org/breaking-changes/listeners-removed.html · https://v3-migration.vuejs.org/breaking-changes/attrs-includes-class-style.html
created: 2026-10-21
---
# Vue 透传属性 attrs

## 一句话定义
`$attrs` 是**父组件传进来、但没被本组件「认领」的那些东西**的集合——包括普通 HTML 属性、`class` / `style`、`v-on` 监听器；组件只有一个根节点时它们被自动贴到根元素上（fallthrough），`inheritAttrs: false` 时改由开发者用 `v-bind="$attrs"` 指定去处。

## 要点

### 一条判据决定谁进 `$attrs`
```
父组件写上 <Child id="a" class="b" @click="c" msg="d" />
                    ↓
      子组件有没有 props / emits 里声明它？
        ├─ 声明了  → 被「消费」，不出现，也不再自动透传
        └─ 没声明  → 留在 $attrs 里，是 fallthrough attribute
```
官方原文把已声明项描述为已被组件 "consumed"（attrs.html "Nested Component Inheritance"）。

- `$attrs` 里包含 `class`、`style`、`id` 以及形如 `onClick` 的**监听器函数**
- 反例提醒：`$attrs` 里**不会有**已声明的 prop，也不会有已声明的 emit 监听器

### 默认行为：单根节点自动透传
```vue
<!-- MyButton 的模板 -->
<button>Click Me</button>

<!-- 父组件 -->
<MyButton class="large" @click="onClick" />

<!-- 最终 DOM -->
<button class="large">Click Me</button>
```
- `class` / `style` **不是覆盖，是合并**：子组件根节点原本 `<button class="btn">`，结果变成 `class="btn large"`
- `v-on` 监听器同理落到根元素；若根元素自己已经有同名 `v-on`，**两个都会触发**
- 子组件的根节点如果又是另一个组件（`<BaseButton />`），`$attrs` 会**继续向下转发**

### 透传也遵守「声明即消费」
转发给内层组件的属性里，**不包含**被本组件声明为 props 或 emits 的那些——它们在上一层就已经被消费掉了。

这也是 [[Vue 组件事件]] 里「声明原生事件名会吃掉原生 `click`」的同一机制：一声明，监听器就离开 `$attrs`，不再落到根元素。

### 关掉自动透传：`inheritAttrs: false`
需要把属性放到**根节点以外**的元素上时用。3.3+ 可以写在 `<script setup>` 里：
```vue
<script setup>
defineOptions({ inheritAttrs: false })
</script>

<template>
  <div class="btn-wrapper">
    <button class="btn" v-bind="$attrs">Click Me</button>
  </div>
</template>
```
- 不带参数的 `v-bind` 会**把一个对象的所有属性**展开绑定到目标元素上
- `inheritAttrs: false` 只关掉「自动贴到根元素」，**不影响** `$attrs` 本身的内容

### 多根节点：不自动透传，且会告警
```vue
<!-- 多根模板 -->
<header>...</header>
<main>...</main>
<footer>...</footer>
```
- 多根组件**没有**自动 fallthrough 行为；`$attrs` 没被显式绑定就直接运行时告警
- 显式绑到某个根节点（如 `<main v-bind="$attrs">`）后告警消失
- 这是 3.x 的既定行为，不是「bug」

### 在 JS 里读：`useAttrs()`
```vue
<script setup>
import { useAttrs } from 'vue'
const attrs = useAttrs()
</script>
```
| 位置 | 拿法 |
| --- | --- |
| `<template>` | `$attrs` 直接用 |
| `<script setup>` | `useAttrs()` |
| 普通 `setup(props, ctx)` | `ctx.attrs` |
| Options API | `this.$attrs` |

- ⚠️ **`attrs` 对象永远是最新值，但本身不是响应式的**（出于性能考虑）；**不能**用 `watch` 观察它，需要响应式就用 prop，或者用 `onUpdated()` 在每次更新时读最新值
- ⚠️ **大小写原样保留**（这点和 props 相反）：`foo-bar` 要用 `attrs['foo-bar']` 取，不能写 `attrs.fooBar`；`@click` 暴露为 `attrs.onClick` 函数
- `useAttrs()` 与 `useSlots()` 是**普通的运行时函数**，不是编译器宏，需要 import（对照 [[Vue 编译器宏]]）

### Vue 2 → 3 的两处语义变化
| 变化 | 2.x | 3.x |
| --- | --- | --- |
| `class` / `style` 是否在 `$attrs` 里 | **不在**（vDOM 特殊处理） | **在**（迁移指南） |
| `$listeners` | 独立对象 | **已移除**，监听器并入 `$attrs`（迁移指南） |

- 迁移陷阱：用 `inheritAttrs: false` 的组件里，以前 `class` / `style` 会**残留**在根元素上（因为它们不在 `$attrs` 里）；3.x 之后它们随 `$attrs` 一起被移走，**样式可能突然失效**（这正是迁移指南点名的场景）
- 2.x 的 `v-bind="$attrs" v-on="$listeners"` 写法，3.x 里**只留 `v-bind="$attrs"`** 即可

### 两个常见误判
- **`$attrs` 不是「所有属性」**，是「所有**未被认领**的属性」。想拿到已声明的 prop，用 `props`，不是 `$attrs`
- **`inheritAttrs: false` 不等于「禁用透传」**，只等于「别自动贴，我自己贴」。只要模板里写了 `v-bind="$attrs"`，透传照旧

## 我的理解 / 类比
`$attrs` 像**快递柜里没人签收的包裹**：

- 组件在 `props` / `emits` 里**声明了某个名字**，等于在签收单上签了字，包裹被前台领走（不再出现）
- 没签收的，就留在柜子里（`$attrs`），最后**默认被丢到门口**（根元素）
- 多根节点就是**没有唯一的门口**——快递员不知道该放哪，于是报警（告警），除非你明确说「放这儿」（`v-bind="$attrs"`）
- `inheritAttrs: false` 是「**别放门口，我自取**」，不是「拒收」

最容易记错的一条：`class` / `style` 在 3.x 里**也进这个柜子**了。2.x 时代它们被特殊照顾、不进 `$attrs`，所以老代码开 `inheritAttrs: false` 时常有「样式怎么没了」的怪事——不是 bug，是它们终于跟着队伍一起走了。

另一条值得记的：**`$attrs` 永远是新的，但你追不到它的变化**。它像一块实时刷新但没装通知的公告屏——你去看总是最新的，但没有任何机制会在它变的时候叫你。

## 关联
- [[Vue 组件 props]] —— 同一判据的两面：props 是「已认领」的契约，`$attrs` 是「未认领」的剩余；已声明项从 `$attrs` 摘除的规则在那张卡也提过
- [[Vue 组件事件]] —— 机制交汇：已声明的 emit 监听器同样从 `$attrs` 摘除，声明原生 `click` 会「吃掉」原生点击，根因就在这里
- [[Vue 组件通信方式]] —— 上位选型卡：`$attrs` 在「父传子」一节里被列为透传通道，本卡是它的细节展开
- [[Vue 编译器宏]] —— 对照关系：`defineOptions({ inheritAttrs: false })` 是宏，`useAttrs()` 是运行时函数，两者的编译期 / 运行时边界看那张卡
- [[Vue 插槽]] —— 边界相邻：插槽决定「内容片段」放哪渲染，`$attrs` 决定「属性」落到哪个元素，是两条独立的穿透路径
