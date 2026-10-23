---
type: concept
tags: [vue, 组件通信]
aliases: [Vue 组件通信, 组件通信方式, 父传子, 子传父, 跨组件传值, 组件传参, props drilling]
source: https://vuejs.org/guide/components/props.html · https://vuejs.org/guide/components/events.html · https://vuejs.org/guide/components/slots.html · https://vuejs.org/guide/components/provide-inject.html · https://vuejs.org/guide/essentials/template-refs.html · https://v3-migration.vuejs.org/breaking-changes/events-api.html
created: 2026-10-21
---
# Vue 组件通信方式

## 一句话定义
Vue 组件之间传递**数据与意图**的手段清单：按「关系远近 × 数据流向」分成三组——父传子、子传父、跨层级 / 全局；这些工具解决的不是同一件事，选型先问「数据在谁手里、谁负责渲染、中间隔了几层」。

## 要点

### 总图：四条通道
```
父 ──props / 插槽内容 / $attrs / v-model(prop)──▶ 子
父 ◀──emit / v-model(update 事件) / slot props / 暴露的方法── 子
祖先 ──provide──▶ 任意深度后代 ──inject──▶
任意组件 ◀──▶ Pinia / 路由参数 / 事件总线（mitt）
```

### 决策表：什么关系用什么
| 场景 | 首选手段 | 关键理由 |
| --- | --- | --- |
| 父 → 子 传**值** | `props` | 声明式契约、单向、子只读 |
| 父 → 子 传**结构** | 插槽 | 传的是模板片段，不是数据 |
| 父 → 子 **命令式**操作 | 模板 ref + `defineExpose` | 官方建议优先用 props / emit，ref 是最后手段 |
| 子 → 父 传**意图** | `emit` | 子只负责说「想改」，改由父执行 |
| 子 ↔ 父 **双向** | `v-model` / `defineModel` | 本质是 prop 向下 + `update:xxx` 事件向上 |
| 子 → 父 交**数据**、父决定渲染 | 作用域插槽 | 数据在子、渲染在父 |
| 祖先 → 深后代 | `provide` / `inject` | 解决 prop drilling |
| 兄弟 | 提升到共同父组件 | 官方明确推荐：siblings 走父组件 |
| 任意组件 / 跨页面 | Pinia | 显式、可调试、不受组件树限制 |
| 跨页面且要可分享 / 刷新保留 | 路由 `params` / `query` | URL 就是状态载体 |

### 父传子

**① `props` —— 主力通道**
声明契约、单向流动、子只读。写法、校验、响应式解构、`$attrs` 边界都在 [[Vue 组件 props]]，此卡不重复。

**② 插槽 —— 传「模板」而不是「值」**
父组件把自己模板里的一段片段投递到子组件的 `<slot>` 出口。子组件负责外壳，父组件负责内容。机制见 [[Vue 插槽]]。
- 插槽内容属于**父组件的作用域**，因此能用父的数据、用不了子的数据
- 子组件的 `scoped` 样式不作用于插槽内容

**③ `$attrs` —— 透传**
- **已声明的 props / emits 会被「消费」并从 `$attrs` 摘除**，未声明的才留在里面
- 单根节点组件自动透传到根元素；多根节点要显式 `v-bind="$attrs"`
- Vue 3 中 `$listeners` 已并入 `$attrs`（迁移指南专章 `$listeners merged into $attrs`），所以透传监听器也走同一条 `v-bind="$attrs"`
- 不想自动继承用 `inheritAttrs: false`

**④ 模板 ref —— 父组件直接抓手柄**
```vue
<!-- 父 -->
<script setup>
import { useTemplateRef, onMounted } from 'vue'   // 3.5+
const child = useTemplateRef('child')
onMounted(() => child.value.someMethod())
</script>
<template><Child ref="child" /></template>
```
- 3.5 之前要写 `const child = ref(null)`，且**变量名必须与模板 ref 值同名**
- ⚠️ **`<script setup>` 组件默认私有**：父组件拿到的实例上什么都访问不到，子组件必须 `defineExpose({...})` 显式开放
- `defineExpose` **必须在任何 `await` 之前调用**，否则 await 之后暴露的属性访问不到
- 暴露出的 ref 会被**自动解包**（拿到的直接是值，不是 ref）
- `ref` 用在 `v-for` 里得到的是**数组**（3.5+）
- 官方态度很明确：能访问子组件全部属性 / 方法会让父子**紧耦合**，"in most cases, you should try to implement parent / child interactions using the standard props and emit interfaces first"

### 子传父

**① `emit` —— 主力通道**
子组件 `emit('事件名', 载荷)`，父组件 `@事件名` 监听。**组件事件不冒泡**，只能监听直接子组件 —— 这一条正是「隔层就得换方案」的根因。完整机制见 [[Vue 组件事件]]。

**② `v-model` / `defineModel` —— 双向绑定是「两条线合成一条」**
不是独立机制，就是「prop 向下 + `update:modelValue` 事件向上」。3.4 起 `defineModel` 把这套收成一个宏。

**③ 作用域插槽 —— 子交数据、父做渲染**
`<slot :item="todo" />` 在出口绑定，父组件用 `v-slot="slotProps"` 接收。**方向是子→父**，但和 emit 的语义完全不同：emit 是「请求父组件改变状态」，slot props 是「把数据递给父组件，让父决定这段 UI 长什么样」。

**④ 父组件持 ref 调子组件方法**
与 `defineExpose` 配套，是**反方向**的控制通道：命令式「让子组件做件事」，而不是「子组件通知父组件」。耦合度最高，能不用就不用。

### 跨层级 / 全局

**① `provide` / `inject` —— 解决 prop drilling**
```vue
<!-- 祖先 -->
<script setup>
import { provide, ref, readonly } from 'vue'
const location = ref('North Pole')
function updateLocation() { location.value = 'South Pole' }
provide('location', { location, updateLocation })
</script>

<!-- 任意深度后代 -->
<script setup>
const { location, updateLocation } = inject('location')
</script>
```
- 中间隔多少层都行；**同 key 多个祖先时，最近的那个胜出**
- 注入 key 可以是 **字符串或 `Symbol`**；一个组件可以多次 `provide` 不同 key
- **注入的 ref 原样注入、不会自动解包** —— 这是刻意设计，为了保住与 provider 的响应式连接
- 官方推荐：**响应式状态的修改尽量留在 provider 内部**；需要后代改，就**顺带 provide 一个修改函数**
- 想彻底禁止后代改，用 `readonly(count)` 包一层
- 传普通值（非 ref / reactive / computed）**不会自动变响应式**；Options API 的 `provide()` 函数同理，要靠 `computed(() => this.message)` 才有响应性
- 可选依赖用默认值 `inject('key', '默认值')`；默认值需要新建实例时用 `inject('key', () => new ExpensiveClass(), true)`，**第三个参数 `true` 表示把第二个参数当工厂函数**
- `app.provide()` 提供应用级依赖，插件里常用
- 非 `<script setup>` 时，`provide()` / `inject()` **必须在 `setup()` 内同步调用**
- 官方列举的用途还包括「**组件与自己的插槽内容之间通信**」，适合总是配套出现的紧耦合组件（Vue 3 迁移指南）

**② Pinia —— 任意组件、跨页面**
显式的全局状态，可调试、可持久化，不受组件树位置限制。见 [[Pinia 选项式与组合式]]。

**③ 事件总线 —— Vue 3 已不是内置能力**
- Vue 3 **移除了 `$on` / `$off` / `$once`**，实例不再是事件发射器，这个用途**没有迁移路径**
- 替代方案是 `mitt` / `tiny-emitter` 这类外部 emitter
- 官方态度：短期的确最省事，但 "几乎总会变成长期的维护负担"，并明确建议 props / events 优先、兄弟走父组件

**④ 路由参数 —— 跨页面的状态传递**
`params` / `query` 天然跨组件、跨页面，还能被刷新与分享保留；配合路由 `props` 可把参数解耦成组件 props。见 [[路由组件传参props]]。

### 四组最容易混的边界
| 对比 | 差别 |
| --- | --- |
| `props` vs `$attrs` | 前者是**声明的契约**，后者是**剩下的杂物**；声明即从 `$attrs` 摘除 |
| `emit` vs 作用域插槽 | 都是子→父：emit 传**意图**（父改状态），slot props 传**数据**（父渲染） |
| `provide`/`inject` vs Pinia | 前者是**隐式依赖树**（谁在上面谁说了算，适合组件库 / 紧耦合组件），后者是**显式全局**（可调试、可跨页面） |
| 模板 ref vs emit | 前者**命令式**（父主动喊子做事），后者**声明式**（子主动通知父）；前者耦合度显著更高 |

### 选型四问
1. **数据在谁手里、谁负责渲染？** → 决定方向（父持有=props，子持有=插槽 / emit）
2. **隔了几层？** → 1 层用 props / emit；深链用 provide/inject；不在同一棵树用 Pinia / 路由
3. **是否「总是配套使用」的组件对？** → 可以考虑 provide/inject 或插槽，而不是层层传 props
4. **要不要被调试、持久化、跨路由？** → 要就用 Pinia，别用事件总线硬凑

## 我的理解 / 类比
组件通信像一栋办公楼里的信息传递：

- **props** 是正式发文：格式固定、单向下发，收文方只能看不能改
- **emit** 是回执单：现场有情况往上报，但**只能报给自己的直接上级**（不冒泡），越级就送不到
- **插槽** 是「我给你留个工位」：外壳我搭，坐进来的人和带来的东西都归你管（父作用域）
- **作用域插槽** 是工位上递出来的**样品**：东西是我的，怎么摆由你决定
- **provide / inject** 是全楼广播的暗号：「本层有饮水机」，只要在这栋楼里、知道暗号就能接；但**离你最近的那台优先**
- **Pinia** 是公司公告板：谁都能看、按规矩改，跨楼层跨时间都在

一句口诀：**往下传用「声明」，往上传用「通知」，隔层用「暗号」，跨页面用「公告板」**。

最容易踩的坑不是「不会用」，而是**用错方向**：拿 ref 去指挥子组件（耦合拉满）、拿事件总线替代状态管理（调试噩梦）、或者为了省事把该 provide 的东西一层层 props 透传下去（prop drilling）。

## 关联
- [[Vue 组件 props]] —— 父传子的主力通道；本卡只讲选型与定位，props 的声明 / 校验 / 解构机制看那张卡
- [[Vue 组件事件]] —— 子传父的主力通道；「组件事件不冒泡」正是本卡「隔层 / 兄弟要换方案」的直接原因
- [[Vue 插槽]] —— 补齐父子之间的第三条线：父传子传结构、子向父反传 slot props
- [[Pinia 选项式与组合式]] —— 跨层级与跨页面的首选方案
- [[路由组件传参props]] —— 路由参数作为跨页面通道，并把 params 解耦成组件 props
- [[Vue 编译器宏]] —— `defineModel` / `defineExpose` 都是编译器宏，编译期产物与版本演进看那张卡
