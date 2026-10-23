---
type: concept
tags: [vue, vue-router]
created: 2026-09-17
aliases: [路由元信息, route meta, 路由自定义字段]
source: https://router.vuejs.org/zh/guide/advanced/meta.html
---

# 路由元信息meta

## 一句话定义
在路由记录上加 `meta` 字段存自定义数据（如是否需要登录、页面标题、权限角色），在导航守卫里统一读取。

## 用于解决什么问题
没有 meta，就只能在每个页面组件里各写一遍登录态 / 角色判断，规则重复还容易漏；改成全局拦截时，守卫里又没有统一的地方可读，只能另维护一份路由名单。

## 要点
- 定义：路由记录里写 `meta: { requiresAuth: true }`，可写在父记录、子记录上。
- 读取：`to.meta` 是**从父到子非递归合并**所有 `meta` 的结果（等价于遍历 `to.matched` 取 meta 的合集）；所以父设了 `requiresAuth`，子自动继承。
- 典型用途：鉴权（`to.meta.requiresAuth && !isLoggedIn()` 则重定向登录）、页面标题、角色权限。
- 配合守卫（见 [[导航守卫]]）：
  ```ts
  router.beforeEach((to, from) => {
    if (to.meta.requiresAuth && !auth.isLoggedIn())
      return { path: '/login', query: { redirect: to.fullPath } }
  })
  ```
- TypeScript：通过 `declare module 'vue-router'` 扩展 `RouteMeta` 接口，给 meta 字段加类型。

## 我的理解 / 类比
像快递单上的"备注栏"：包裹（路由）自己带着"需签收 / 易碎"等标签，分拣员（守卫）扫一眼标签决定怎么处理，不用拆包看内容。

## 关联
- [[导航守卫]] —— 在 `beforeEach` 里读 `to.meta` 做拦截
