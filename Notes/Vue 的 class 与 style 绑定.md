---
type: concept
tags: [vue, 模板语法]
created: 2026-10-10
aliases: [:class, v-bind:class, 动态 class, class 绑定, Class 与 Style 绑定, :style, v-bind:style, 动态 style]
source: Vue 官方文档 · Class 与 Style 绑定 https://cn.vuejs.org/guide/essentials/class-and-style.html
---
# Vue 的 class 与 style 绑定

## 一句话定义
`:class` / `:style` 是 `v-bind` 对这两个属性的增强：值可以是**字符串、对象、数组**——于是「要不要加某个类」「一次加一串类」都能用表达式声明，不必在代码里手拼字符串。

## 用于解决什么问题
类名随状态变化时，只能在 JS 里拼字符串或直接操作 classList；条件一多就写成 'btn ' + (active ? 'active ' : '') 这种难读的表达式，还得自己保证重渲染时增删不漏。

## 要点

### 三种值类型
`:class`（`:style` 同理）接受三种值：

| 值类型 | 写法 | 语义 |
| --- | --- | --- |
| 字符串 | `:class="someVar"` | 由变量提供整串类名 |
| 对象 | `:class="{ active: isActive }"` | 键是类名，值是布尔条件——真才加 |
| 数组 | `:class="[classA, classB]"` | 元素可为字符串 / 对象 / 嵌套数组 |

- 对象的键含连字符必须加引号：`{ 'text-danger': hasError }`
- 数组元素是**表达式**，字符串元素要带引号——`[flex flex-col]` 是语法错误，应为 `['flex flex-col']`

### 数组可混放：静态类 + 条件类
这是最常见的写法，也是「一串类名看不懂」的来源：

```html
:class="['flex flex-col w-80', { 'items-start': icon }]"
```

拆开看：

| 数组元素 | 类型 | 作用 |
| --- | --- | --- |
| `'flex flex-col w-80'` | 字符串 | **永远**带上 |
| `{ 'items-start': icon }` | 对象 | `icon` 为真时才加 `items-start` |

等价地拆成静态 `class` + 动态 `:class`，Vue 会自动合并：

```html
class="flex flex-col w-80"
:class="{ 'items-start': icon }"
```

### 三元：单条件二选一
对象适合「多个独立开关」；只有**一个条件、二选一**时，三元表达式更直接：

```html
<!-- 加 / 不加 -->
<span :class="icon ? 'items-start' : ''">

<!-- 两个类名互斥 -->
<span :class="icon ? 'items-start' : 'items-center'">
```

三元同样用于动态样式——**动态给 `style` 的经典写法**：

```html
<span :style="{ color: isActive ? '#2563eb' : '#6b7280' }">
```

> 取舍：**对象** = 多个开关各自独立；**三元** = 单条件互斥二选一。二者可混用；条件一多，就收敛回对象或 `computed`。

### 与静态 class 合并
同一元素上 `class` 与 `:class` 共存时，结果是**并集**，不互相覆盖。

### 组件上的行为
`:class` / `:style` 属于「透传 attribute（fallthrough attrs）」：
- **单根组件**：自动透传到根元素并合并
- **多根（Fragment）**：不自动透传，需组件自己决定挂到哪个元素，否则 Vue 给出告警

### `:style` 同族差异
- 值是对象时，键可用 camelCase 或 kebab-case：`{ fontSize: '14px' }` / `{ 'font-size': '14px' }`
- 值是数组时，多个对象**按顺序合并**，后者覆盖前者：`[baseStyles, overridingStyles]`
- 需要时 Vue 会自动补浏览器前缀

## 我的理解 / 类比
把 `:class` 想成一张清单：
- **字符串** = 写死的一项
- **对象** = 一排开关，每个开关管一个类名
- **数组** = 那张清单本身，可以同时放「写死的项」和「开关」
- **三元** = 一个二选一的岔路口

## 关联
- [[声明式与命令式编程]] —— 用表达式声明「这个类要不要」，而不手动增删 DOM，正是声明式写法
- [[原子化CSS的样式复用]] —— `:class` 数组里那串 `flex flex-col w-80` 正是原子化 CSS 的产物，那张卡讲为什么类名会变长
- [[Flex 布局]] —— 示例里的 `items-start` 对应 `align-items: flex-start`
