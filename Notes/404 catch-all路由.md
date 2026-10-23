---
type: concept
tags: [vue, vue-router]
created: 2026-09-17
aliases: [catch-all 路由, 404 路由, 通配路由, pathMatch, NotFound]
source: https://router.vuejs.org/zh/guide/essentials/dynamic-matching.html#%E6%8D%95%E8%8E%B7%E6%89%80%E6%9C%89%E8%B7%AF%E7%94%B1%E6%88%96-404-Not-found-%E8%B7%AF%E7%94%B1
---

# 404 catch-all路由

## 一句话定义
用一个匹配"任意路径"的路由记录来兜底所有未定义的 URL，渲染 404 页面；History 模式下尤其必需。

## 用于解决什么问题
用户从书签或外部链接进来的失效地址无法预先穷举；没有它就得手写 redirect 逐条堵漏，漏掉的路径连「页面不存在」都提示不了。

## 要点
- 写法：`{ path: '/:pathMatch(.*)*', name: 'NotFound', component: NotFound }`——`(.*)*` 是正则"任意字符、可重复"，把整段路径存进 `route.params.pathMatch`（数组）。
- 不带 `*` 的 `(.*)` 也能用；带 `*` 的优势是 `pathMatch` 为数组，能 `router.push({ name: 'NotFound', params: { pathMatch: route.path.substring(1).split('/') } })` 保留原路径分段。
- ⚠️ **History 模式下必须有它**：因为服务器 fallback 让所有路径都返回 `index.html`，若应用内不兜 404，未知路径会显示首页而非 404（见 [[hash模式与history模式]]）。
- 常与 [[路由重定向与别名]] 配合：把某些旧路径 `redirect` 到新结构，剩余未知交给 catch-all。
- 正则 / 可选 / 可重复等更细语法见官方 route-matching-syntax 文档页。

## 我的理解 / 类比
像商场的"服务台"：任何找不到的店铺（未知路径），导购都把你领到服务台（404 页），而不是让你在空地里发呆（白屏）。

## 关联
- [[hash模式与history模式]] —— History 模式必须配 404 兜底
- [[路由重定向与别名]] —— 旧路径重定向 + 未知路径交 catch-all
