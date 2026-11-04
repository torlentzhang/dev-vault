---
type: concept
tags: [vue, 组件通信]
aliases: [v-model, 组件 v-model, 双向绑定, defineModel, modelValue]
source: https://vuejs.org/guide/components/v-model.html · https://vuejs.org/api/sfc-script-setup.html · https://blog.vuejs.org/posts/vue-3-4
created: 2026-10-27
---
# Vue 组件 v-model

## 一句话定义
组件上的 `v-model` 是实现**双向绑定**的语法：**3.4+** 推荐的写法是编译宏 `defineModel()`——一行就声明好「一个 model prop + 一个对应的值更新事件」，并返回一个**可以直接改的 ref**，改它，父组件绑定的值跟着变。它把过去「prop 向下 + emit 向上」的两段手写样板收成一个宏。

## 用于解决什么问题
组件要支持父组件 `v-model`，3.4 之前必须自己写两遍：一个 `modelValue` prop 收值、一个 `update:modelValue` 事件回传，中间还得手动把 `:value` / `@input` 接起来。样板长、容易漏一边，具名 `v-model:title` 还要再重复一套。`defineModel` 把这两条线合成一条：**你拿到的是一个可读可写的 ref，不再关心「哪一半忘了写」**。

> 本卡讲的是**组件上的 `v-model`**——它是什么、`defineModel` 怎么用。原生元素上的 `v-model`（如 `<input v-model="x">`）是另一套展开，不在此展开。

## 要点

### 版本前提：3.4 才可用
- 3.3 以**实验特性**引入，需在编译选项里显式打开 `defineModel: true`，默认关闭
- **3.4 转正稳定**，实验开关被移除；3.4 公告同时提到它对 `v-model` **修饰符**的支持变好了
- 3.3 及更早的项目看到 `defineModel is not defined` 之类报错，先查版本，别查语法

### 最小用法与底层展开（所有技巧都从这层长出来）
```vue
<!-- 子：一行搞定 -->
<script setup>
const model = defineModel()          // 父：<Child v-model="countModel" />
</script>
<template>
  <div>父组件绑定的值：{{ model }}</div>   <!-- ref 在模板里自动解包 -->
</template>
```
编译器把它展开成两件事（官方「底层机制」原文）：
- 一个名为 **`modelValue`** 的 prop，本地 ref 的值与之同步
- 一个名为 **`update:modelValue`** 的事件，本地 ref 被改时触发

所以父组件里的 `v-model="foo"` 等价于：
```vue
<Child :modelValue="foo" @update:modelValue="$event => (foo = $event)" />
```
**记住这个展开，后面所有「为什么能这么写」都有答案。**

### 技巧一：包装原生表单元素，一行接上
`model` 就是个 ref，所以可以直接丢给原生元素的 `v-model`：
```vue
<script setup>
const model = defineModel()
</script>
<template>
  <input v-model="model" />
</template>
```
这正是封装基础输入组件的常见写法——**不需要再手写 `:value` + `@input` 转接**。

### 技巧二：具名 model，一个组件挂多个
第一个参数是字符串字面量时，它被用作 prop 名；不传则默认 `modelValue`。
```vue
<!-- 子 -->
<script setup>
const title = defineModel('title')   // 父：<MyComponent v-model:title="bookTitle" />
</script>

<!-- 需要额外 prop 选项时，选项对象放在 model 名之后 -->
const title = defineModel('title', { required: true })
```
一个组件里重复这个模式，就能同时暴露多个双向绑定口，互不干扰。

### 技巧三：prop 选项照旧可用
因为 `defineModel` 声明了一个 prop，底层 prop 的选项可以直接传进去：
```js
const model = defineModel({ required: true })   // 使 v-model 必填
const model = defineModel({ default: 0 })       // 提供默认值
const model = defineModel({ type: String })
```

### ⚠️ 技巧四（最该记住的坑）：`default` 会让父子不同步
官方 WARNING 原文：给 `defineModel` 的 prop 设了 `default`，而父组件**没有**为该 prop 提供任何值，会导致父子之间不同步：

```vue
<!-- Child.vue -->
<script setup>
const model = defineModel({ default: 1 })
</script>

<!-- Parent.vue -->
<script setup>
const myRef = ref()          // 注意：没有初始值
</script>
<template>
  <Child v-model="myRef"></Child>
</template>
```
结果：**父组件的 `myRef` 是 `undefined`，子组件的 `model` 却是 `1`**。两边各持一份值，谁也没同步给谁。

- 根因还是那条展开：`default` 只作用于**子组件这一侧的 prop 解析**，不会反向写回父组件的变量
- 实操口径：**`v-model` 的值应该由父组件提供**。要用默认值就让父组件的 ref 自己带初值（`ref(1)`），而不是塞给 `defineModel`
- 另外，搭配 `defineProps` 的 `withDefaults` 时，**可变引用类型（数组 / 对象）的默认值应在 `defineModel` 中用函数包一层**，避免意外修改和外部副作用

### 技巧五：读修饰符——解构返回值
想响应 `v-model.trim` 这类修饰符，把返回值解构开：
```js
const [modelValue, modelModifiers] = defineModel()

// 对应 v-model.trim
if (modelModifiers.trim) {
  // ...
}
```

### 技巧六：转换器 `get` / `set`——修饰符真正生效的地方
修饰符给到了，还得在读写时真正改变值。用 `get` / `set` 变换：
```js
const [modelValue, modelModifiers] = defineModel({
  // 这里不需要 get()，省略
  set(value) {
    if (modelModifiers.trim) {
      return value.trim()   // 用了 .trim 修饰符就回传去空格后的值
    }
    return value
  }
})
```
- `set`：**写回父组件前**做变换（如 trim、转数字）
- `get`：**从父组件读值时**做变换（如反序列化）
- 官方措辞是「model ref 的**值转换选项**」，与 prop 选项共处同一个对象

### 技巧七：TypeScript 下的类型速查
```ts
const modelValue = defineModel<string>()
//    ^? Ref<string | undefined>

// 带选项的默认 model，required 会移除可能的 undefined
const modelValue = defineModel<string>({ required: true })
//    ^? Ref<string>

const [modelValue, modifiers] = defineModel<string, 'trim' | 'uppercase'>()
//                 ^? Record<'trim' | 'uppercase', true | undefined>
```

| 写法 | 得到的类型 |
| --- | --- |
| `defineModel<string>()` | `Ref<string \| undefined>` |
| `defineModel<string>({ required: true })` | `Ref<string>` |
| 解构第二项的 `defineModel<string, 'trim' \| 'uppercase'>()` | `Record<'trim' \| 'uppercase', true \| undefined>` |

- **第二个类型参数专门描述修饰符**，`Record` 的键就是允许的修饰符名、值为 `true | undefined`
- 只要把 `required: true` 传进去，`undefined` 就被从类型里摘掉——这是 TS 体验最直接的一处收益

### 工具链提示（类型提示不生效时先查这里）
3.4 公告明确列出了配套版本：升级 3.4 时**必须**同时升 Volar / `vue-tsc@^1.8.27`，Vite 项目建议 `@vitejs/plugin-vue@^5.0.0`。`defineModel` 的类型提示异常，第一诊断是工具链落后，不是写法问题。

### 什么时候不该用它
`defineModel` 只管「双向绑定」这一件事。如果子组件只是**通知父组件一个意图、并不持有这个值**（比如 `submit`、`cancel`、`delete`），那是 [[Vue 组件事件]] 的地盘——不要为了省事把一切都做成 `v-model`。

## 我的理解 / 类比
`defineModel` 像给父子之间接了**一条双向专线**，而不是像过去那样自己拉两根线（一根往下送值、一根往上回传），还要确认每根都接对了。拉好专线之后，你在子组件这头对着话筒说话（`model.value = x`），父组件那头就直接听到。

「`default` 会造成不同步」这条坑，本质是**两边各印了一本账**：子组件按 `default: 1` 在自己的账本上记了「有 1」，可父组件的变量还是空的。专线只负责**传你主动说出口的话**，不负责替对方补默认值。所以默认值该记在父组件那本账上，而不是子组件自己先填一个。

「修饰符 + 转换器」这一对，则像**贴在对讲机上的便签与按便签行事的习惯**：`v-model.trim` 是父组件贴过来的一张「记得去空格」便签（`modelModifiers.trim`），而 `set` 里那句 `return value.trim()` 才是你真正照便签做了事。**只有便签、没有动作，等于没贴。**

## 关联
- [[Vue 组件事件]] —— 底层关系：`defineModel` 展开后就是一个 prop 加一个 `update:xxx` 事件；那张卡有「3.4 之前的手写版」对照，看完就懂这个宏省掉了什么
- [[Vue 组件 props]] —— 同一套机制：model prop 也是 prop，`default` / `null` 的触发规则、只读语义都归那张卡；本卡只讲它作为「双向」的那一面
- [[Vue 编译器宏]] —— 上位关系：`defineModel` 是那 7 个宏之一，「3.3 实验 → 3.4 稳定」的版本演进与宏 vs 运行时函数的边界看那张卡
- [[Vue 组件通信方式]] —— 定位关系：双向绑定在通信选型表里的位置，以及「什么时候该用 emit 而不是 v-model」
- [[编译时与运行时]] —— 原理支撑：宏编译后消失、只留下 prop 与事件，是「编译期处理、运行时不存在」的又一处例证