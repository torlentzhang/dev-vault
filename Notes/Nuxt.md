---
type: concept
tags: [vue, nuxt, ssr]
created: 2026-11-20
aliases: [Nuxt, Nuxt.js, Nuxt 4]
source: https://nuxt.com/docs/getting-started/introduction · https://nuxt.com/docs/guide/concepts/rendering · https://nuxt.com/docs/getting-started/data-fetching · https://nuxt.com/docs/guide/directory-structure/pages · https://nuxt.com/docs/guide/concepts/auto-imports · https://nuxt.com/docs/guide/directory-structure/server · https://nuxt.com/docs/getting-started/deployment · https://nuxt.com/docs/guide/best-practices/hydration
---
# Nuxt

## 一句话定义
Nuxt 是建立在 Vue 之上的应用框架：它在 Vue 之上装好路由、构建、SSR 双端渲染与 Node 服务端引擎（Nitro），用一套目录约定让同一份组件代码同时产出服务端渲染的 HTML、可交互的 SPA 和同源后端接口。

## 用于解决什么问题
纯 Vue + Vite 项目里，文件路由、SSR 双端构建、数据在服务端预取后传给客户端、后端接口、多目标部署，每一环都要自己搭；Nuxt 把这些默认装好，把「页面在哪、路由怎么来、数据怎么取、接口写在哪」变成目录约定。

## 要点

### 它是什么，不是什么
Nuxt 不是另一个前端框架，是 Vue 的上层应用框架（底层 Vue + Vue Router + Vite + Nitro）。默认形态是「首屏由服务端生成、站内跳转不刷新」的 SPA。

### 目录即约定（Nuxt 4）
Nuxt 4 默认源码根目录为 `app/`：

- `app/pages/index.vue` → `/`，`app/pages/users/[id].vue` → `/users/:id`；`pages/` 是可选目录，有 pages 时 `app.vue` 需含 `<NuxtPage />`
- `app/components/`、`app/composables/`、`app/utils/` 自动导入；Vue 的 `ref`/`computed` 与内置 composable 也无需 import
- `server/api/hello.ts` 写后端接口：`defineEventHandler` 导出，对应 `/api/hello`
- 根目录 `nuxt.config.ts` 是唯一配置入口，`modules` 数组启用模块
- `public/` 原样输出静态资源

网上的 Nuxt 3 教程写的是根目录 `pages/`，与 Nuxt 4 的 `app/pages/` 不同。

### 数据获取：三个 API
- `$fetch`：最朴素的网络请求，服务端与客户端都能用
- `useFetch`：`$fetch` 的封装，在 universal 渲染下只取一次——服务端取到的数据随 payload 传给客户端，水合时不重复请求，并且结果参与首屏 HTML
- `useAsyncData`：类似 `useFetch`，控制更细（自定义 key、handler）

`useFetch` / `useAsyncData` 应在 setup、插件、路由中间件中调用；表单提交、按钮点击等纯客户端场景用 `$fetch`。

### 渲染模式
默认 universal rendering（SSR + 客户端水合），按需切换：

- `ssr: false` → 纯 CSR；Nitro 仍在、接口仍可用，只是首屏为空壳
- `nuxt generate` → 预渲染成静态站，产物无服务端，`server/api` 失效
- `routeRules` → 混合渲染，按路由分配 SSG / ISR / SWR / CSR

### 部署
同一套代码可构建到 Node 服务器、静态托管、Serverless / Edge；用 `nitro.preset` 或环境变量切换，未指定时自动检测，Node 为默认输出。

### 适用边界
- 适合：需要逐 URL 的 HTML（SEO、分享卡片）、首屏内容、前后端同源一个工程、按路由混合渲染
- 不适合：纯后台 / 内部工具（纯 CSR 就够）、已有成熟复杂且不需要 SSR 的 Vite 工程

### 版本
当前稳定版 Nuxt 4.x，最低 Node.js 22。

## 我的理解 / 类比
纯 Vue + Vite 像买好食材和厨房，菜谱得自己写；Nuxt 是把整套后厨班底配齐——你只写菜品（Vue 组件），出餐口（HTML）、传菜（水合）、点单台（API）都按固定动线摆好。代价是得遵守它的动线约定，不能想怎么摆就怎么摆。

## 常见坑
- SSR 下没有 `window` / `document` / `localStorage`：setup 顶层直接访问会抛错或水合错配。规避：`import.meta.client` 判断、`<ClientOnly>` 包裹、挪进 `onMounted`；持久化状态用 `useCookie` / `useState`
- `<ClientOnly>` 的内容会被 tree-shake 出服务端构建，CSS 可能不内联进首屏，不写 `placeholder` 首屏即空白
- `nuxt generate` 产物没有服务端，`server/api` 不可用
- 自动导入让函数来源不明显，查 API 文档比翻 import 快

## 关联
- [[服务端渲染]] —— Nuxt 默认的 universal rendering 就是这张卡里的 SSR + 水合，routeRules 配置也在那里
- [[SSR 双端构建]] —— Nuxt 把「一份源码编两份产物」的工程化过程内置为默认能力
- [[Vue 自动路由方案]] —— Nuxt 的 pages 文件路由是同一思路的框架内置版本，不必再装插件
- [[静态站点生成器]] —— `nuxt generate` 属于这类工具，适用条件相同
