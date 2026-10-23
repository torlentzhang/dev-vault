---
type: concept
tags: [vue, vue-router]
created: 2026-09-17
aliases: [params 与 query, 路径参数与查询参数, route params vs query]
source: https://router.vuejs.org/zh/guide/essentials/navigation.html
---

# 路由params与query的区别

## 一句话定义
`params` 是 URL 路径里的动态段（如 `/user/123` 的 `123`），`query` 是 `?` 后的键值对（如 `/user/123?tab=info` 的 `tab=info`）；两者用途与写法都不同。

## 用于解决什么问题
分不清两者，就会把页码、筛选项存进组件内部状态，用户一刷新或把链接发给同事，看到的是默认列表而不是当前结果。

## 要点
- **params**：属于"路径的一部分"，必须在路由表里用 `:id` 声明才能解析出来；组件里读 `route.params.id`。它标识"这是哪一个资源"。
- **query**：与路径无关，`?` 后任意 `key=value`，组件里读 `route.query.tab`；常用于筛选、分页、临时 UI 状态，**不需要在路由表声明**。
- 写法差异（编程式导航）：
  - `params` 必须配 `name`：`router.push({ name: 'user', params: { id: '123' } })` → `/user/123`
  - `query` 可配 `path`：`router.push({ path: '/user/123', query: { tab: 'info' } })` → `/user/123?tab=info`
- 选型原则：**标识"哪一个"资源用 params；控制"怎么看 / 筛什么 / 第几页"用 query**。例如用户详情页 subTab 切换用 query，用户 id 用 params。

## 我的理解 / 类比
params 是"收件人地址"（决定包裹送到哪），query 是"配送备注"（怎么送、要不要挂号）。地址错了送错人，备注错了只是送达方式不同。

## 关联
- [[动态路由匹配]] —— params 来自 `:id` 动态段
- [[编程式导航router.push]] —— 携带 params / query 的写法与坑
