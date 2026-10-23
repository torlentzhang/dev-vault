---
type: concept
tags: [vue, vite, 前端工程化]
aliases: [vue-macros, Vue宏, vue 宏]
source: https://vue-macros.dev/guide/getting-started.html · https://vue-macros.dev/guide/bundler-integration.html · https://blog.vuejs.org/posts/vue-3-3 · https://blog.vuejs.org/posts/vue-3-4 · https://blog.vuejs.org/posts/vue-3-5 · https://vuejs.org/guide/extras/reactivity-transform.html
created: 2026-10-01
---
# Vue Macros

## 一句话定义
Vue 非官方语法扩展的编译期实现库（Kevin Deng 主导）：把 Vue 官方没有（或已移除）的**编译器宏**做成构建插件——宏看似函数调用，构建期被 AST 转换改写/消除，运行时不存在。

## 要点

### 生态位：Vue 与 Vite 之间的「扩展层」
- **对 Vue**：扩展的是**编译管线**而非运行时代码。宏的转换必须先于官方 SFC 编译发生，产出再交给官方插件完成编译——所以 Vite 配置里必须「包裹」而非「并列」：`VueMacros({ plugins: { vue: Vue() } })`（一个 `.vue` 文件的 transform 链只能有一个 owner，并列会产生顺序冲突）
- **对 Vite**：基于 [[unplugin]] 实现跨构建工具（`vue-macros/vite`、`/webpack`、`/rollup`…），Vite 只是宿主之一——这与官方宏绑死在 `@vitejs/plugin-vue` / `vue-loader` 形成对照
- **隐性第四方 Volar**：构建能过不代表 IDE 认识这些宏，还需 `tsconfig.json` 的 `vueCompilerOptions.plugins` 配 `vue-macros/volar`，否则类型层一片红；这也是它对 vue-tsc / Volar 版本敏感的原因

### 与 Vue 核心的关系演进：双向通道
```text
社区新想法 ─→ Vue Macros 试验 ─→ 验证成功 → 收编进核心
                                ↘ 官方弃用 → 回收到 Vue Macros 继续维护
```
- **收编案例**：defineOptions、defineSlots 由 Vue Macros 先行试验，Vue 3.3（2023-05）进入核心，其文档直接标注 "Implemented by Vue 3.3"
- **回收案例**：Reactivity Transform（`$ref` / `$()` / `$$()`）在 Vue 3.3 废弃、3.4（2023-12）从核心移除，官方博客明确「想继续用请迁移到 Vue Macros」→ 该特性现在 vue-macros 中以 `stable` 状态维护（独立包 `@vue-macros/reactivity-transform`）
- **并行演进**：defineModel 在 Vue 3.3 实验、3.4 稳定；Reactive Props Destructure 在 3.5（2024-09）稳定为默认行为——官方每收编一块，Vue Macros 相应模块就降级为兼容层

### 与官方宏的分工边界
- 官方宏（`defineProps` / `defineEmits` / `defineModel` / `defineOptions` / `defineSlots`…）由 Vue 编译器内置，只覆盖「已验证的共识能力」
- Vue Macros 覆盖的是共识之外的地带：未进入核心的想法（definePropsRefs、setupSFC、chainCall…）、被官方放弃但社区仍在用的能力——**两者是「核心 ↔ 试验场/收容所」的关系，不是竞争关系**

### 现状（2026-10）
- v3+ 要求 Node ≥ 20.19、Vue ≥ 2.7（部分特性需 ≥ 3.2.25），已放弃 Vue 2（旧版支持）/ webpack 4 / Node 16-18
- Vite 与 Rollup 完整支持，其余 bundler 有限支持；Nuxt 走 `@vue-macros/nuxt` 模块

## 我的理解 / 类比
Vue Macros 之于 Vue 核心，像「预科班 + 退休公寓」的合体：新语法先进来试运行，被官方看中的「毕业」进核心（defineOptions / defineSlots）；被官方淘汰但仍有用户的「入住」继续被维护（Reactivity Transform）。它不碰 Vue 运行时的本体，只是在构建期（Vite 等 bundler 的 transform 阶段）给编译管线加一道前置工序。

## 关联
- [[unplugin]] —— 抽象与实现：Vue Macros 的跨 bundler 能力建立在 unplugin 的统一插件接口上
- [[unplugin-auto-import]] / [[unplugin-vue-components]] —— 同属「构建期转换增强 Vue 开发体验」的插件层，常同项目共存
- [[vite+Vue3工程的初始化与启动]] —— Vue Macros 属于 Vite + Vue3 工程初始化后的进阶语法增强选项
