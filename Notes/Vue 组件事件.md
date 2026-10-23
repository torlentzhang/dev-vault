---
type: concept
tags: [vue, 组件通信]
aliases: [Vue 组件事件, defineEmits, emits, $emit, 自定义事件, 组件自定义事件]
source: https://vuejs.org/guide/components/events.html · https://vuejs.org/api/sfc-script-setup.html · https://vuejs.org/api/options-state.html · https://vuejs.org/guide/components/attrs.html · https://blog.vuejs.org/posts/vue-3-3
created: 2026-10-21
---
# Vue 组件事件

## 一句话定义
子组件向父组件**向上通知**的通道：子组件用 `$emit` / `defineEmits` 返回的 `emit` 抛事件，父组件用 `v-on`（`@xxx`）监听。它是单向数据流的另一半——props 向下传、事件向上抛，数据本身始终由父组件改。

## 要点

### 为什么需要它：和 props 是一对
```
父 ──props──▶ 子      数据向下（子组件只读）
父 ◀──emit── 子      意图向上（子说「想改」，父自己动手改）
```
props 是子组件的**收货单 + 验收标准**，emits 是子组件的**对外广播清单**。对称关系见 [[Vue 组件 props]]。

### 最核心的一步：`$emit` 在 `<script setup>` 里拿不到
| 代码位置 | 怎么拿到 emit |
| --- | --- |
| `<template>` | `$emit('submit')` 直接可用 |
| `<script setup>` | **没有 `$emit`**，只能用 `defineEmits()` 的返回值 |
| 普通 `setup()` | `setup(props, { emit })`，配 `emits: ['submit']` 选项，可安全解构 |
| Options API | `this.$emit('submit')` |

官方原文：`$emit` 在 `<script setup>` 中不可访问，但 `defineEmits()` 返回一个等价函数（events.html）。

```vue
<script setup>
const emit = defineEmits(['submit'])
function onClick() {
  emit('submit', { id: 1 })   // 等价于模板里的 $emit('submit', { id: 1 })
}
</script>
```

### 三种声明写法：选一条，不能混
```vue
<script setup>
// ① 数组：只列名字，不做校验
const emit = defineEmits(['inFocus', 'submit'])

// ② 对象：带运行时校验（返回值 true / false；null 表示声明但不校验）
const emit = defineEmits({
  submit: ({ email, password }) => {
    if (email && password) return true
    console.warn('Invalid submit event payload!')
    return false
  },
  click: null,
})

// ③ 纯类型（TS）：编译期约束参数签名
const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
}>()

// ③' 3.3+ 具名元组写法，更简洁
const emit = defineEmits<{
  change: [id: number]
  update: [value: string]
}>()
</script>
```

- `defineEmits` 接收的值**与 `emits` 选项完全相同**（官方原文）
- **运行时声明与类型声明互斥**：同时用 → 编译报错 `cannot accept both type and non-type arguments`
- 具名元组语法是 **3.3+** 新增，旧的调用签名写法仍然支持（Vue 3.3 发布博客 "More Ergonomic `defineEmits`"）

### 编译期发生了什么
和 `defineProps` 同属 [[Vue 编译器宏]]：按名字识别、编译后消失、不需要 import。

```js
// 大致等价产物（非逐字）
export default {
  emits: ['submit'],              // 声明被 hoist 到模块作用域
  setup(__props, { emit }) { /* ... */ }
}
```

由此带出两条硬约束：

| 约束 | 后果 |
| --- | --- |
| 选项被 hoist 到模块作用域 | 数组 / 对象里**不能引用 setup 局部变量**（编译错误），**可以**引用 import 进来的绑定 |
| 不能在函数内调用 | 官方原文：must be placed directly within `<script setup>`，**不能**写在函数里面 |

### 声明了到底有什么用：三件「Vue 才知道」的事
很多人只写数组、不知为何要写。声明不是形式主义，它实际改变运行时行为：

**① 已声明事件的监听器会从 `$attrs` 中摘除**
父组件传的 `@submit` 若被子组件声明，则视为「已消费」，**不再作为 fallthrough attribute 透传到根元素**；未声明的会被透传（单根节点自动加在根元素上）。官方原文用词是 "removed from the component's `$attrs` object"。这与 props 卡里「已声明的 prop / emit 不进 `$attrs`」同一条机制。

**② 声明原生事件名会「吃掉」原生事件**
> 官方原文：If a native event (e.g., `click`) is defined in the `emits` option, the listener will now only listen to component-emitted `click` events and no longer respond to native `click` events.

⚠️ **别随手声明 `click` 这类原生事件名**。一声明，父组件的 `@click` 就再也收不到根元素的原生点击，这是很难查的静默失效。

**③ 规避第三方代码手动 dispatch DOM 事件造成的边界情况**
官方给出的推荐理由是：声明全部事件既让组件契约更清楚，也让 Vue 能把已知监听器从 fallthrough attributes 中排除，避免 3rd party 代码手动 dispatch DOM 事件引发的 edge case。所以「尽量声明全」是有运行时收益的，不是纯风格问题。

### 事件名大小写：JS 抛 camelCase，模板听 kebab-case
`emit('someEvent')` 对应父组件模板里的 `@some-event="callback"`，**会自动转换**；与 props 一样，官方**推荐模板里用 kebab-case 的监听名**。

### 传递参数与监听侧细节
- **所有额外参数原样透传**：`$emit('foo', 1, 2, 3)` → 监听函数收到三个参数
- 监听侧可用**内联箭头函数**接参：`@increase-by="(n) => count += n"`
- 支持 `.once` 修饰符：`<MyComponent @some-event.once="callback" />`
- 监听函数是方法时，第一个参数即 emit 附带的值

### 组件事件**不冒泡**
只能监听**直接子组件**抛出的事件，不像原生 DOM 事件那样向上冒泡。兄弟 / 深层嵌套组件通信要用状态管理（如 [[Pinia 选项式与组合式]]）或事件总线（events.html 的 TIP）。

### 与 `v-model` / `defineModel` 的关系（打通一条线）
`v-model` 不是魔法，就是「prop 向下 + `update:propName` 事件向上」：

```vue
<!-- 3.3 之前的手写版 -->
<script setup>
const props = defineProps(['modelValue'])
const emit = defineEmits(['update:modelValue'])
function onInput(e) {
  emit('update:modelValue', e.target.value)
}
</script>
```

`defineModel`（**3.4+**）把这个模式收成一个宏：底层「声明一个 model prop 和一个对应的值更新事件」（sfc-script-setup 原文 + 3.3 发布博客的 BEFORE / AFTER 对照）。看懂这一条，`defineEmits` 就通了一半。

## 我的理解 / 类比
props 与 emits 是一对**对讲机的两端**：props 是总部（父组件）发给现场（子组件）的指令，现场只能听、不能改指令本身；现场遇到需要总部决策的事，就按对讲机上的**频道按钮**（声明好的事件名）呼叫回去——**按钮必须在清单上登记过**，否则总部根本不知道你在喊什么（未声明就被当成透传给根元素的普通监听器）。

「声明原生事件名会吃掉原生事件」这条我印象最深：给对讲机登记一个叫 `click` 的频道，结果**原本能听见的现场环境音（原生点击）被这个频道顶掉了**——不是报错，是静默地听不见了。所以事件名要起得有语义（`submit`、`change`），别去抢 DOM 的通用词。

「已声明 → 从 `$attrs` 摘除」这条，则是**快递签收**的比喻：清单上写了的东西，前台（组件）会直接签收并转交内部，不会再原样丢到门口（根元素）让路人看见。

## 关联
- [[Vue 组件 props]] —— 对称关系：props 向下、emit 向上，共同构成单向数据流；两者都是「已声明即从 `$attrs` 摘除」的同一机制
- [[Vue 编译器宏]] —— 上位关系：`defineEmits` 是那 7 个编译器宏之一；宏的本质、版本演进、宏 vs 运行时函数看那张卡，本卡不重复
- [[编译时与运行时]] —— 原理支撑：`defineEmits` 编译后消失、只留下 `emits` 选项，是「编译期处理、运行时不存在」的例证
- [[Vue 插槽]] —— 互补关系：组件通信的第三条线（父传子渲染片段），与 props / emit 共同构成父子通信手段
