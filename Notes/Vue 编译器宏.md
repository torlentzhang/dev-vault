---
type: concept
tags: [vue, 编译时, 编译器宏]
aliases: [Vue compiler macros, 编译器宏, compiler macro, Vue 宏]
source: https://vuejs.org/api/sfc-script-setup.html · https://blog.vuejs.org/posts/vue-3-3 · https://blog.vuejs.org/posts/vue-3-4 · https://blog.vuejs.org/posts/vue-3-5 · https://github.com/vuejs/core/blob/v3.5.44/packages/compiler-sfc/src/compileScript.ts
created: 2026-10-15
---
# Vue 编译器宏

## 一句话定义
`<script setup>` 里那些**看起来像函数调用、运行时却根本不存在**的东西——编译器按**名字**识别它们，在编译期改写或整体擦除。所以它们不是函数，而是**写给编译器的语法标记**（compiler macro）。

## 要点

### 本质：判断标准回到「求值时机」
- 形态像函数、能带参数和泛型，但**没有函数体**，运行时不存在
- **不需要 import**（源码层面当前 7 个宏全部无需 import；官方文档只在 `defineProps` / `defineEmits` 一节明说了这句）
- 这正是 [[编译时与运行时]] 里「宏消除」那一行的具体案例：能在编译期处理掉的，就不留到运行时

### 当前 stable（Vue 3.5.x）被编译器识别的宏：共 7 个
依据：`compiler-sfc` 源码中的 `MACROS` 常量数组，处理分支在 `compileScript.ts` 的 `processDefineProps` / `processDefineEmits` / `processDefineOptions` / `processDefineSlots` / `processDefineExpose` / `processDefineModel`。

| 宏               | 作用                                       | 引入 / 变化节点                |
| --------------- | ---------------------------------------- | ------------------------ |
| `defineProps`   | 声明 props（运行时对象/数组，或纯类型）                  | 早期即有；3.3 起支持导入类型         |
| `defineEmits`   | 声明 emits                                 | 早期即有；3.3 新增命名元组类型写法      |
| `defineExpose`  | 显式暴露实例属性给父组件模板 ref                       | 早期即有                     |
| `defineOptions` | 声明组件选项（`inheritAttrs`、自定义选项等）            | **3.3 新增**               |
| `defineSlots`   | 声明插槽名与 slot props 的**类型**提示（纯编译期，无运行时产物） | **3.3 新增**               |
| `defineModel`   | 声明可被父组件 `v-model` 双向绑定的 prop，返回可直接改的 ref | **3.3 实验（默认关闭）→ 3.4 稳定** |
| `withDefaults`  | 为**类型式** `defineProps` 提供默认值             | 3.2 引入；**未废弃**（详见下）      |

> 「7 个」是**源码事实**，不是官方文档的表述——文档是分节介绍各宏，从未给出清单。

### 版本演进：3.3 到底「新收」了什么
这是最容易记混的一段，因为「3.3 的宏」和「3.3 新增的宏」是两个口径。

```text
3.2  withDefaults
3.3  新增 defineOptions、defineSlots ── 正式可用，无需任何开关
     defineModel 以「实验性」引入 ── 需编译选项 defineModel: true，默认关闭
     泛型组件 <script setup generic="T">；defineProps/Emits 支持导入类型
3.4  defineModel 转正稳定，实验开关被移除
3.5  Reactive Props Destructure 稳定并默认开启（解构 props 保持响应性）
3.6  宏集合无任何增删（仍在 RC）
```

- `defineOptions` 与 `defineSlots` 走的是「社区先行试验 → 官方收编」路线（详见 [[Vue Macros]]）
- **3.6 至今无宏变化**：其 `MACROS` 与 3.5 逐项一致，多处宏源文件逐字节未变；3.6 的新东西（Vapor Mode、`defineVaporComponent` 等）**不属于编译器宏**，后者是 `runtime-vapor` 的运行时函数

### 最容易混淆：宏 vs 真正的运行时函数
名字很像、功能还重叠，但**分属编译期与运行期两侧**：

| 名字 | 真相 |
| --- | --- |
| `useSlots` / `useAttrs` | **真正的运行时函数**，必须 `import { useSlots, useAttrs } from 'vue'`。文档原文用词是 "actual runtime functions" |
| `useModel` | 运行时 helper，是 `defineModel` 的**编译目标**，不需要自己写 |
| `useTemplateRef` / `useId` / `onWatcherCleanup` | 3.5 新增的**运行时 API**，需 import，与宏无关 |

最有意思的一处对照：`defineSlots` 是**宏**、编译期被擦除、自身不产生运行时代码，但它的编译产物是去**调用 `useSlots()`**——`defineSlots` 负责编译期的类型契约，`useSlots` 负责运行期的取值。

### `withDefaults` 的准确定位（易错重灾区）
- **没有被废弃**：文档无 deprecated 字样，源码无 `@deprecated`，它仍是 `MACROS` 正式成员
- 官方口径是「3.4 及以下**需要**它」（"the `withDefaults` compiler macro is **needed**"），3.5+ 推荐改用解构默认值 `const { foo = 1 } = defineProps(...)`
- **不能单独使用**：必须包住一个类型式 `defineProps` 调用，第二参数必填
- ⚠️ **不要与解构默认值混用**：编译器会告警，且**会让 reactive props destructure 静默失效**（解构退化为一次性取值，失去响应性）
- 它仍有一个解构路径不具备的能力：在**返回类型上移除 optional 标记**

### 限制：区分「有官方依据」与「没有」
| 限制 | 依据 | 状态 |
| --- | --- | --- |
| 宏不需要 import | 文档只点名 `defineProps`/`defineEmits`；源码对 7 个宏都做移除处理 | 源码级成立；文档措辞**未**推广到全部宏 |
| 宏**不能被别名** | 3.4 起硬编译错误（3.3 只在别名时静默移除、不报错） | 有源码依据，文档未写 |
| 宏选项被 hoist 到模块作用域，**不能引用 setup 局部变量** | 官方文档 + 源码 `checkInvalidScopeReference` | 有官方依据 |
| `defineProps`/`defineEmits` **不能 runtime 与 type 混用** | 官方文档 + 源码 | 有官方依据 |
| 宏不能作为值传递/引用 | ⚠️ **无官方依据**，且编译期**不报错**；`const f = defineProps` 能编译通过，问题在运行时 `ReferenceError` | **存疑**，不可写成「编译期限制」 |
| 宏必须写在 `<script setup>` 顶层 | ⚠️ **无官方依据**，且编译器**不校验**；写在 `if` / 函数体内会静默失效（宏调用原样残留、`props` 未生成） | **存疑**，实践上成立但官方未文档化 |

## 我的理解 / 类比
编译器宏像**写给施工队的暗号，而不是给住户的开关**。你在图纸（源码）上写「这里要留个门」（`defineProps`），施工队（编译器）一看就懂，直接在成品墙里把门开好——交房后你在屋里**找不到「留门」这个工具本身**，因为它只是图纸上的记号。

`withDefaults` 与解构默认值的关系，则像**旧式窗帘盒 vs 直接在窗框上装窗帘**：新房子（3.5+）直接在窗框上装就好；旧房子（3.4 及以下）必须靠窗帘盒。窗帘盒没被拆掉、也还能用，只是**别再和「直接装在窗框上」同时用**——一混用，窗帘盒会把你新装的那套悄悄顶掉。

「宏不能别名、不能当值用」这类说法，我记得最清楚的一条是：**它不是一个值，所以你不能把它传给别人，也不能给它改名**——这不是编译器专门拦你，而是它本来就不存在于运行时。

## 关联
- [[Vue Macros]] —— 对照关系：官方宏（核心内置）vs 社区宏扩展库（核心之外的试验场/收容所）；`defineOptions`、`defineSlots` 正是「社区试验 → 官方收编」的案例
- [[编译时与运行时]] —— 原理支撑：宏是「编译期处理、运行时不存在」的典型例证
- [[Vue Hooks（组合式函数）]] —— 相邻话题：同在 `<script setup>` 的 setup 上下文中，讲组合式复用与调用时机约束
- [[路由组件传参props]] —— 实践落点：其中 `defineProps({ id: String })` 是 vault 里少数实际用法示例
