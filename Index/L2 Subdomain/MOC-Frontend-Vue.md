---
type: moc
tags: [moc, framework, vue]
created: 2026-09-15
---
# Vue · MOC

> 子领域地图：Vue 框架本体与实践的入口（路由 / 响应式 / 组件通信 / 状态管理 …）。

## 模板与绑定
（插值、指令、`v-bind`、class / style 绑定、`v-model` …）
- [[v-model]] 本质是「prop 向下 + 事件向上」，原生与组件两套展开
- [[Vue 的 class 与 style 绑定]]

## 组合式 API 与响应式
（编写、规则、与 React Hooks 区别 …）
- [[Vue 编译器宏]]
- [[Vue Hooks（组合式函数）]]

## 组件通信
（选型总览、props 向下传参、emit 向上通知、插槽、`v-model` 双向绑定、provide / inject、跨层级与全局 …）
- [[Vue 组件通信方式]] 总览与选型入口，先看这张
- [[Vue 组件 props]]
- [[Vue 组件事件]]
- [[Vue 组件 v-model]] 组件上的双向绑定，3.4+ 用 `defineModel` 宏
- [[Vue 插槽]]
- [[Vue 透传属性attrs]]

## 内置组件
（无需注册即可用：`<Teleport>`、`<Transition>`、`<KeepAlive>`、`<Suspense>` …）
- [[Vue Teleport]] 把内容渲染到指定 DOM 节点，逻辑位置不变

## 工程实践
（Vue3 项目脚手架、初始化与启动 …）
- [[vite+Vue3工程的初始化与启动]]
- [[unplugin-auto-import]]
- [[unplugin-vue-components]]
- [[unplugin-icons]]
- [[Vue Macros]]
- [[px 转 vw 的 PostCSS 插件]] 移动端模板（vue3-vant-mobile / Vant）默认用它
- [[axios 接口层封装]] 框架无关的请求层分层，Vue 项目里最常落在这里

## 路由（Vue Router）
- [[MOC-Frontend-Vue-Router]]
（自动路由 / 原理 / 配置实战 / 进阶 …）

## 组合式工具集（VueUse）
- [[MOC-Frontend-Vue-VueUse]]
（状态 / 响应式进阶 / 元素与鼠标 / 浏览器 / 网络 …）

## 状态管理（Pinia）
- [[Pinia 选项式与组合式]]
