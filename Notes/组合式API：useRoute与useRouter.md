---
type: concept
tags: [vue, vue-router]
created: 2026-09-17
aliases: [useRoute, useRouter, 路由组合式 API, composition api]
source: https://router.vuejs.org/zh/guide/advanced/composition-api.html
---

# 组合式API：useRoute与useRouter

## 一句话定义
在 `<script setup>` 里用 `useRoute()` 拿当前路由对象、`useRouter()` 拿路由器实例，替代 Options API 里的 `this.$route` / `this.$router`。

## 用于解决什么问题
逻辑一抽成 composable 或放进 Pinia store，`this.$route` 就不存在了；只能把路由对象一路当参数透传，层级一深，组件就被迫接收自己用不到的 props。

## 要点
- `useRoute()` 返回**响应式**的当前路由（`route.params` / `route.query` / `route.meta` 会变），模板自动刷新。
- `useRouter()` 返回路由器实例，用于 `router.push` / `replace` / `go` 等导航操作。
- setup 中没有 `this`，所以必须用这两个函数；但**模板里仍可直接用 `$route` / `$router`**，只在模板用时不必 `useRoute`。
- ⚠️ 避免监听整个 `route` 对象，应只 `watch(() => route.params.id, ...)` 这类具体属性。
- 组合式守卫：`onBeforeRouteUpdate` / `onBeforeRouteLeave`（见 [[导航守卫]]、[[动态路由的组件复用问题]]），可在任意被 `<router-view>` 渲染的组件里用。
- 在路由外部（如 Pinia store、全局守卫）用 router，应通过参数传入或 `useRouter()` 在当前组件上下文调用。

## 我的理解 / 类比
`useRoute` 是"当前这封信的内容"（会随收件人变），`useRouter` 是"邮局柜台"（用来寄新信 / 改地址）。

## 关联
- [[动态路由匹配]] —— `useRoute().params` 取参数
- [[导航守卫]] —— 组合式守卫 `onBeforeRouteUpdate` / `Leave`
- [[路由组件传参props]] —— 不取 props 时用 `useRoute` 的替代
