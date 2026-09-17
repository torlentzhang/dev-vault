---
type: concept
tags: [vue, vue-router]
created: 2026-09-17
aliases: [router.push, router.replace, 编程式导航, programmatic navigation]
source: https://router.vuejs.org/zh/guide/essentials/navigation.html
---

# 编程式导航router.push

## 一句话定义
在 JS 里通过路由器实例方法（而非 `<router-link>`）控制跳转；核心是 `router.push`，另有 `replace` / `go` 家族。

## 要点
- `router.push` 参数：字符串路径 `router.push('/users/eduardo')`、带 path 的对象 `{ path: '/users/eduardo' }`、带 name+params 的对象 `{ name: 'user', params: { username } }`、可带 `query` / `hash`。
- ⚠️ **如果提供了 `path`，`params` 会被忽略**——必须用 `name` 才能携带 params；否则写成完整 path 字符串（注意自己 `encodeURIComponent`）。
- `router.replace(...)`：与 `push` 类似，但不向 history 栈压新记录（替换当前条目）；也可 `router.push({ path, replace: true })`。
- `router.go(n)` / `router.back()` / `router.forward()`：在 history 栈中前进 / 后退 n 步（类似 `window.history.go`），步数不够时**静默失败**。
- `push` / `replace` / `go` 都是 `window.history` 对应 API 的翻版，无论用哪种 history 模式都能正常工作。
- 导航方法都返回 **Promise**，可 `await` 判断成功 / 失败（重复导航会返回 NavigationFailure）。

## 我的理解 / 类比
`push` 像"新开一页浏览记录"，`replace` 像"覆盖当前这页"，`go(-1)` 像"按浏览器返回键"。

## 关联
- [[命名路由]] —— `name + params` 才带得上参数
- [[路由params与query的区别]] —— params 配 name、query 可配 path
