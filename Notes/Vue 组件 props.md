---
type: concept
tags: [vue, 组件通信]
aliases: [Vue props, props, 组件 props, defineProps]
source: https://vuejs.org/guide/components/props.html · https://vuejs.org/api/sfc-script-setup.html · https://vuejs.org/guide/components/attrs.html · https://vuejs.org/guide/typescript/composition-api.html · https://github.com/vuejs/core/blob/v3.5.44/packages/compiler-sfc/src/script/defineProps.ts
created: 2026-10-16
---
# Vue 组件 props

## 一句话定义
父组件向子组件传数据的**声明式入口**：在 `<script setup>` 里用 `defineProps` 声明「本组件接收什么」，编译期生成对应的 `props` 选项；数据**单向向下**流动，子组件只读。

## 用于解决什么问题
子组件要用父组件的数据，只能反向去读父实例或全局变量，组件无法独立复用；若改用普通 HTML 属性传值，又没有类型、必填与默认值约束，传错要到运行时才暴露。

## 要点

### 三种声明写法：选一条路线，不能混
```vue
<script setup>
// 1. 数组语法：只列名字，不做校验
defineProps(['title', 'count'])

// 2. 对象语法：带运行时校验
defineProps({ title: { type: String, required: true } })

// 3. TS 类型语法：只写类型，编译期生成运行时声明
defineProps<{ title: string; count?: number }>()
defineProps<Props>()   // 引用接口
</script>
```

- **运行时声明与类型声明互斥**。同一次调用里既给对象又给泛型 → `defineProps() cannot accept both type and non-type arguments`；分两次调用 → `duplicate defineProps() call`（两者都是编译错误）
- `defineProps` 的选项会被 **hoist 到模块作用域**，因此**不能引用 `<script setup>` 里的局部变量**（编译错误），但**可以引用 import 进来的绑定**

### 运行时校验选项（对象语法专用）
| 选项 | 说明 |
| --- | --- |
| `type` | `String / Number / Boolean / Array / Object / Date / Function / Symbol / Error`，或自定义类 / 构造函数（走 `instanceof` 检查） |
| `required` | 必填 |
| `default` | 默认值 |
| `validator` | 自定义校验，返回 `false` 时告警 |

细节：
- **所有 props 默认 optional**，除非 `required: true`
- **对象 / 数组的 `default` 必须是工厂函数** `() => ({})`，保证每个实例各自持有一份；`Function` 的 `default` 则**不是**工厂函数，它本身就是默认值
- `validator(value, props)` 的第二个参数（完整 props）是 **3.4+** 才有
- 校验发生在**组件实例创建之前**，所以 `default` / `validator` 里拿不到 `data`、`computed` 等实例属性

### Boolean 的特殊 casting
- 只声明为 `Boolean` 的 prop，父组件**不传时会被 cast 成 `false`**（模仿原生布尔属性）；想让它表现得像非 Boolean prop，显式写 `default: undefined`
- 类型是数组时，Boolean 规则只在 **Boolean 排在前面**时生效：`[Boolean, String]` 有 casting，`[String, Boolean]` 没有

### 单向数据流：props 只读
- 子组件直接改 prop → **开发环境告警**（`Props are readonly.`）；生产环境不做只读包装，也就不告警
- ⚠️ 但**对象 / 数组的嵌套属性仍然改得动** —— Vue 明说拦截这类修改「代价过高」。这会让子组件以不明显的方式影响父组件状态，官方建议避免；需要改就 emit 事件让父组件来改

### 与 `$attrs` 的边界
- **已声明为 prop 的属性不会出现在 `$attrs` 里**（已被这个组件「消费」），同理已声明的 `emit` 事件
- **未声明**的属性 / `v-on` 监听器是 fallthrough attribute，单根节点时**自动透传到根节点**
- 多根节点组件**不自动透传**，需显式 `v-bind="$attrs"`，否则有运行时警告
- 不想自动继承时用 `inheritAttrs: false`（3.3+ 可写在 `<script setup>` 里：`defineOptions({ inheritAttrs: false })`）

### props 命名与传值
- 声明用 **camelCase**（便于在模板里直接当标识符引用），父组件模板传入用 **kebab-case**（对齐 HTML 属性惯例）；in-DOM 模板下必须 kebab-case
- 非字符串值要靠 `v-bind` 才按 JS 表达式解析：`:likes="42"`；`<Comp is-published />` 等价于传 `true`

### 类型式声明下的默认值：两条路径
| 版本 | 做法 |
| --- | --- |
| 3.4 及以下 | `withDefaults(defineProps<Props>(), { msg: 'hello', labels: () => ['one','two'] })` |
| 3.5+ | 原生解构默认值 `const { msg = 'hello', labels = ['one','two'] } = defineProps<Props>()` |

- 类型语法**本身无法直接声明默认值**，这是它最主要的短板
- `withDefaults` **只能包类型式 `defineProps`**（包运行时对象声明报错）、**第二参数必填**，且**未被废弃**——它仍有解构路径给不了的能力：**在返回类型上移除 optional 标记**
- 用 `withDefaults` 时，可变引用类型的默认值同样要写成工厂函数；解构默认值则**不需要**，编译器会自己包工厂
- ⚠️ **两者不可混用**：编译器会告警 `withDefaults() is unnecessary when using destructure...`，并**让响应式解构静默失效**（详见下）

### 返回值、解构与响应式（本卡最值得记住的机制）
- 声明的 props **自动暴露到模板**，模板里直接写 `{{ title }}`；`defineProps` **返回一个包含全部已传 props 的对象**，JS 里要通过它访问：`const props = defineProps(['title']); console.log(props.title)`
- 这个对象是**响应式的**，但**普通解构会丢响应性**（`const { title } = props` 之后即定死），要用 `toRefs()` / `toRef()`
- **响应式 props 解构**：3.3 作为独立特性从 Reactivity Transform 中拆出（当时实验性、非默认）→ **3.5 稳定并默认开启**
- 机制（这层才是关键）：**它不是运行时机制，而是编译期变换**。编译器把「解构语句」和「脚本里对解构变量的引用」一起改写：

  | | 编译产物 | 模板解析 |
  | --- | --- | --- |
  | 只用解构 | 解构语句**整个消失**，绑定标记为 `props` | 编译成直读 `$props.msg` → 永远是最新值 |
  | 混入 `withDefaults` | **多出一行真实 JS 解构**，绑定降级为 `setup-const` | 编译成读 `$setup.msg`（一次性常量）→ 后续变化不再相关 |

  实测绑定元数据对照：`{"msg":"props"}` vs `{"msg":"setup-const"}`。这也正是告警里 `Reactive destructure will be disabled` 的字面含义

### 类型式声明的静态分析限制
- 类型 → 运行时的转换是 **AST 静态分析**，不是真正的类型推导
- 3.2 及以下：只能类型字面量或本地接口引用
- **3.3 起**：支持 imported types 和有限复杂类型
- 需要真实类型分析的（如条件类型）不支持：**可用于单个 prop 的类型，不能用于整个 props 对象**

### 宏层面的约束（体系细节见 [[Vue 编译器宏]]）
- 只能在 `<script setup>` 中使用；**不需要 import**（显式 import 会被移除并告警）
- **一次性调用**；**不能别名**，但两种失败方式不同：

  | 写法 | 结果 |
  | --- | --- |
  | `import { defineProps as dp } from 'vue'` | **编译期硬报错**（编译器扫 import 说明符时就发现宏被改名） |
  | `const p = defineProps`（未调用） | **编译期静默通过**，`p` 原样留在产物里；运行时 `defineProps` 并不存在 → **`ReferenceError`** |

  根因：编译器只按「顶层 + 是否为 `defineProps(...)` 调用」接管。没有调用就不处理，而它本来就不是运行时函数

## 我的理解 / 类比
props 像子组件的**收货单 + 验收标准**：单子上写明收什么、什么类型、缺货怎么办（默认值），但**签收方无权改单子上的字**（只读）。想改内容得打电话回总部（emit 给父组件），而不是自己拿笔改。

响应式解构那段另记一个比喻：**它是编译器替你贴的「直通标签」**——你在代码里叫 `msg`，编译器把它直接接到 `$props.msg` 上，所以永远读得到最新的。一旦 `withDefaults` 出现，标签被撕掉，退回成「抄一份单子塞进抽屉」——之后单子变了，抽屉里那份不会跟着变。理解了「响应式靠的是编译期改写」，就自然明白为什么混用会失效。

`const p = defineProps` 这条我印象最深：它和一个真正的函数长得一模一样，但你把它当值传递时，才发现**运行时根本没有这个东西**——不是编译器拦你，是它本来就不存在。

## 关联
- [[Vue 编译器宏]] —— 上位关系：`defineProps` 是那 7 个编译器宏之一；宏的本质、版本演进、宏 vs 运行时函数看那张卡，本卡不重复
- [[路由组件传参props]] —— 应用关系：路由把 `params` 映射成组件 props，从而解耦组件与 `$route`
- [[编译时与运行时]] —— 原理支撑：`defineProps` 是「编译期处理、运行时不存在」的标准例证
- [[Vue Hooks（组合式函数）]] —— 相邻话题：同在 setup 上下文，且同样存在「解构丢响应式」的问题
