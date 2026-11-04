---
type: concept
tags: [vue, 模板, 双向绑定]
created: 2026-10-29
aliases: [v-model, 双向绑定, v-model 本质, modelValue, update:modelValue]
source: https://vuejs.org/guide/components/v-model.html · https://vuejs.org/guide/essentials/forms.html · https://vuejs.org/api/built-in-directives.html
---
# v-model

## 一句话定义
`v-model` 是**双向绑定**的语法糖，本质是**「用 `v-bind` 绑一个 prop 向下传值 + 监听一个事件把新值交回给值的主人去改」**——组件场景监听 `update:xxx`（自定义事件），原生元素场景监听 DOM 的 `input` / `change` 事件。

## 用于解决什么问题
Vue 规定 prop 是**只读**的、父级更新才向下流动（单向数据流），子组件想让父组件的值变，只能「喊一声」再由父组件自己改。这对「输入框 ↔ 变量」这种最高频的场景意味着每个输入都要手写一段「绑值 + 监听回写」，父组件那侧同样要手写一段监听。`v-model` 把这两端各压缩成一个词，**在遵守单向数据流的前提下**少写两段样板。

## 要点

### 本质：一个 prop 向下、一个事件向上
官方原文（「Under the Hood」）：

```vue
<!-- 父组件写 -->
<Child v-model="foo" />

<!-- 编译后等价于 -->
<Child :modelValue="foo" @update:modelValue="$event => (foo = $event)" />
```

- **它不是魔法，也没有绕过单向数据流**：子组件从头到尾没碰过 prop，改的是**父组件自己的变量** `foo`——改的人本来就是值的主人。
- 所谓「双向」，是**两次单向叠加**出来的观感：

```text
① 子 emit('update:modelValue', newVal)   ← 子只是喊了一声
② 父的监听执行 foo = newVal              ← 父改自己的状态
③ foo 变了 → 作为新 prop 流下去          ← 单向数据流照常
```

- ⚠️ `update:modelValue` 是**普通的组件自定义事件**（只是约定成「`update:` + prop 名」），不是 Vue 内置的特殊事件类型。**子组件不 emit，父组件那半句监听就永远不触发。**

### 两套展开，别混着套
`v-model` 展开成什么，**取决于用在哪**：

| 用在哪儿 | prop | 事件 | 载荷 |
| --- | --- | --- | --- |
| 组件（默认） | `modelValue` | `@update:modelValue` | **值本身** |
| 组件（带参）`v-model:title` | `title` | `@update:title` | 值本身 |
| `<input>`（text 类）/ `<textarea>` | `value` | `@input` | DOM 事件对象 |
| `<input type="checkbox">` / `radio` | `checked` | `@change` | DOM 事件对象 |
| `<select>` | `value` | `@change` | DOM 事件对象 |

**原生元素上根本没有 `@update:xxx`**——那是组件自定义事件的命名约定。且载荷不同：组件事件的载荷是值，原生事件的载荷是事件对象，所以原生展开要写 `$event.target.value`。

```vue
<input v-model="searchText" />
<!-- 等价于 -->
<input :value="searchText" @input="searchText = $event.target.value" />
```

> 一句话：**组件用 `update:xxx`，原生用 DOM 事件**。把「v-model = v-bind + @update」套到原生元素上是错的。

### 组件场景的进阶形态
- **带参数**：`v-model:title="foo"` 改用 `title` prop + `update:title` 事件；官方有「v-model Arguments」专节。
- **多个同时用**：一个组件可挂多个口，每个同步不同 prop，组件侧无需额外选项——

```vue
<UserName v-model:first-name="first" v-model:last-name="last" />
```

  这也印证了「v-model 只是语法糖」：它本来就允许任意多个，因为它就是「一个 prop + 一个事件」的组合。

### 修饰符：原生元素与组件的分工不同
**原生元素**的三个修饰符由框架内置实现：

| 修饰符 | 作用 |
| --- | --- |
| `.lazy` | 同步时机从 `input` 改为 `change` 事件 |
| `.number` | 把输入串转成数字 |
| `.trim` | 去掉首尾空白 |

⚠️ `.number` 有官方限定：`parseFloat()` 解析失败时**保留原字符串**；输入被清空时得到**空字符串**；元素带 `type="number"` 时会**自动应用**该修饰符。

**组件**上的修饰符（如 `<MyComp v-model.trim="text" />`）框架不会替你转换，只负责**把修饰符传进来**，名字随写法而变：

| 写法 | 修饰符从哪拿 |
| --- | --- |
| 旧写法（`defineProps`） | `modelModifiers` prop；带参时是 `arg + "Modifiers"`（如 `titleModifiers`） |
| 3.4+（`defineModel()`） | 解构返回值第二项 |

```js
const [modelValue, modelModifiers] = defineModel()
// modelModifiers.trim 对应父组件写的 v-model.trim
```

**关键坑**：拿到修饰符 ≠ 生效。还得自己在 `set` 里按修饰符改值——**只有便签、没有动作，等于没贴**。具体实现见 [[Vue 组件 v-model]]。

### 什么时候不该用
`v-model` 表达的是「这个值由父子共同持有」。如果子组件只是**通知一个意图**（`submit` / `cancel` / `delete`），值并不在父子之间往返，那是普通 emit 的事——见 [[Vue 组件事件]]。

## 我的理解 / 类比
`v-model` 像父子之间的一根**双向话线**，但拆开看是两根单向线：父组件把值递下去（`v-bind`），子组件有话要说就按一下呼叫键（emit），**父组件听到后自己改**。

它不是「让子组件获得改写权」，而是「让子组件有个正经的汇报通道」。正因为改的始终是父组件自己，prop 只读那条规则一次都没破。

原生元素与组件的差别，则像**直接对着原始设备说话** vs **通过一层遥控器**：原生 `v-model` 接的是浏览器给的 `input` / `change` 信号（所以拿到的是事件对象，要自己 `.target.value`），组件 `v-model` 接的是你自己约定的 `update:xxx` 频道（拿到的是干净的值）。

## 关联
- [[Vue 组件 v-model]] —— 组件侧专章：`defineModel()` 的用法、prop 选项、`default` 不同步的坑、修饰符转换器与 TS 类型；本卡只给总纲，不重复其细节
- [[Vue 组件事件]] —— 底层通道：`update:modelValue` 就是一个普通自定义事件，那条规则（未声明不被监听、不冒泡）同样适用于它
- [[Vue 组件 props]] —— 配套规则：prop 只读、单向数据流，正是 `v-model` 必须绕成「父改父的值」的原因
- [[Vue 组件通信方式]] —— 定位关系：双向绑定在通信选型表里的位置
