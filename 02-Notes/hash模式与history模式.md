---
type: concept
tags: [vue, vue-router]
created: 2026-09-17
aliases: [hash 模式, history 模式, 路由历史模式, history mode]
source: https://router.vuejs.org/zh/guide/essentials/history-mode.html
---

# hash模式与history模式

## 一句话定义
Vue Router 创建路由器时，通过 `history` 配置选择"URL 怎么变、怎么被监听"的模式；主流是 Hash 与 History 两种，另有 Memory 模式用于非浏览器环境。

## 要点
- **Hash 模式**：`createWebHashHistory()`，URL 形如 `example.com/#/user/1`。`#` 后的内容**不会发给服务器**，所以无需任何服务器配置；缺点是 `#` 不美观、对 SEO 不友好。
- **History 模式（推荐）**：`createWebHistory()`，URL 形如 `example.com/user/1`，干净。但**生产环境必须配置服务器 fallback**——任何不匹配静态资源的路径都回退到 `index.html`，否则用户直接访问 / 刷新就会 404。
  - Nginx：`location / { try_files $uri $uri/ /index.html; }`
  - 原生 Node / Express：用 `connect-history-api-fallback` 中间件或手动回退
  - 注意：fallback 后服务器不再报 404，所以要在应用内用 [[404 catch-all路由]] 兜住未知路径。
- **Memory 模式**：`createMemoryHistory()`，不与 URL 交互、不自动触发初始导航，适合 Node / SSR / 测试；浏览器里用则**没有历史记录**（无法前进后退），需 `app.use(router)` 后手动 `router.push` 首屏。
- 开发环境（Vite）已自动配好 History 的 fallback，所以本地刷新不白屏；**白屏 404 多发生在自部署的生产环境**。

## 我的理解 / 类比
History 模式像"门牌号直接对外"——好看，但物业（服务器）得知道"这家人不在时把人领到大堂（index.html）"；Hash 模式像"信箱里的便利贴（# 后面）"——快递员（服务器）根本不看它，所以不用提前打招呼。

## 关联
- [[前端路由是什么]] —— 路由的本质与 SPA 机制
- [[404 catch-all路由]] —— History 模式下兜底未知路径
