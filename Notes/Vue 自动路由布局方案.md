---
type: concept
tags: [vue, vue-router]
created: 2026-09-24
aliases: [自动路由布局, Vue 布局方案, layouts 插件]
source: https://router.vuejs.org/file-based-routing/ · https://github.com/loicduong/vite-plugin-vue-layouts-next
---

# Vue 自动路由布局方案

## 一句话定义
文件自动路由只解决「URL → 哪个页面」，不解决「页面外面套什么壳」；布局在 Vue Router 里本质是嵌套路由，而自动路由把 routes 变成生成物后，那层嵌套就没地方手写了——本卡梳理四种落地方式的取舍。

## 用于解决什么问题
同一个后台外壳要让几十个页面共用，又不希望每个页面各自复制一份；文件路由生成 routes 之后没有现成写法可抄，只能在几种补法里做取舍。

## 要点

### 1. 布局的本质：嵌套路由

Vue Router 里「布局」不是独立概念，它就是嵌套路由：父路由渲染布局组件（内含 `<router-view />`），子路由渲染页面。

```js
{ path: '/admin', component: AdminLayout,   // 父路由 = 布局壳
  children: [{ path: 'users', component: UsersPage }] }
```

原理见 [[嵌套路由]]。

### 2. 自动路由带来的冲突

`vue-router/vite` 扫描 `src/pages/**` 自动生成 routes 数组 → **routes 成了生成物**，你无法再手写 `{ component: AdminLayout, children: [...] }` 那层嵌套。

> 结论：布局插件不是「提供布局能力」，而是在生成结果之上把那层嵌套**补回去**。

### 3. 四种方案对比

| 方案                         | 布局放哪           | 与路径解耦 | 跨路径复用 | 类型安全     | 需装依赖   |
| -------------------------- | -------------- | ----- | ----- | -------- | ------ |
| 同名壳文件                      | `pages/` 内     | ❌ 强绑定 | ❌     | ✅        | 无      |
| Route Groups `(group)/`    | `pages/` 内     | 半解耦   | ⚠️ 部分 | ✅        | 无      |
| layouts 插件 + `meta.layout` | `src/layouts/` | ✅ 彻底  | ✅     | ⚠️ 布局层丢失 | 需装     |
| 手写 routes 数组               | 任意             | ✅     | ✅     | ✅        | 放弃自动路由 |

**① 同名壳文件（零插件）**
官方规则：同级同时存在 `users.vue` 与 `users/` 目录时，`users/index.vue` 会渲染在 `users.vue` 的 `<router-view>` 里——`users.vue` 天然就是布局壳。

```text
src/pages/
├── users.vue        ← 布局壳
└── users/
    ├── index.vue
    └── [id].vue
```

局限：壳只能服务 `/users*` 前缀，无法让 `/dashboard`、`/orders` 共用同一个壳。

**② Route Groups**
括号目录不影响 URL，可把共享布局的页面归到一组（官方明说这是给共享布局用的）：

```text
src/pages/(admin)/dashboard.vue  → /dashboard
src/pages/(admin)/settings.vue   → /settings
```

局限：仍靠目录（物理位置）表达布局归属，换布局要搬文件。

**③ layouts 插件**：布局抽到 `src/layouts/`，页面用 `meta.layout` **声明式**指定，布局与路径彻底解耦，任意页面可复用任意布局。
**④ 手写 routes**：放弃自动路由，换回完全自由，代价是失去类型自动生成。

> 补充：`users.create.vue`（点号命名）可让 `/users/create` 只嵌套 URL、不嵌套 UI。

### 4. layouts 插件怎么工作

- 约定目录 `src/layouts/*.vue`，每个布局是普通组件 + `<router-view />`。
- 页面声明：`definePage({ meta: { layout: 'admin' } })`；未声明则用 `default.vue`。
- `setupLayouts()` 把每条路由替换成「布局包页面」的嵌套结构（布局复用同一实例，切页不重建）。

```ts
// vite.config.ts
import VueRouter from 'vue-router/vite'
import Layouts from 'vite-plugin-vue-layouts-next'   // 注意必须带 -next
export default defineConfig({ plugins: [VueRouter(), Vue(), Layouts()] })
```

```ts
// main.ts
import { routes } from 'vue-router/auto-routes'      // 不是 ~pages
import { setupLayouts } from 'virtual:generated-layouts'
const router = createRouter({ history: createWebHistory(), routes: setupLayouts(routes) })
```

布局名规范化（Nuxt 兼容）：`src/layouts/someLayout.vue` → `some-layout`；`src/layouts/desktop/default.vue` → `desktop-default`。

### 5. Vue Router 5 代际注意点（关键）

- **必须用 `vite-plugin-vue-layouts-next`**：原版 `vite-plugin-vue-layouts` 停在 `0.11.0`（2023-12），面向 `vite-plugin-pages` / 早期 `unplugin-vue-router`；装到 Vue Router 5 自动路由项目会因 `~pages` 缺失而报错。
- **`inheritDefaultLayout`**：Vue Router 5 auto routes 生成的是**嵌套结构**（含 `children`），而 `vite-plugin-pages` 生成扁平结构；嵌套时父子都套布局会「双包裹」（页面出现两层侧边栏）。该选项默认 `true`，设为 `false` 可让子路由自带布局时父级不套默认布局。**只对 auto routes 生效**（对 vite-plugin-pages 无效）。
- **代价（官方原话）**：
  - `setupLayouts(routes)` 是**运行时**改路由 → 官方："these changes will not be reflected in types"——**布局这层的类型提示会失效**（`route.name` / `params` 推不出来）。要保住类型得改用 build-time 方案：`extendRoute` / `definePage` / `<route>` 块。
  - 官方在 Extending Routes 页警告："Routes added at runtime require special handling for HMR... using vite-plugin-vue-layouts which can only be used this way"——它是唯一被点名的运行时改路由场景，且需自行处理 HMR。

### 6. 什么时候真的需要它

- 布局需求简单（一个后台壳 + 一个无壳登录页）→ **可以不用**：`pages/admin.vue` 当壳 + `pages/(auth)/login.vue` 足够，还保住类型。
- 多布局 + 需跨路径复用（admin / mobile / blank 三套，页面散落各处）→ 插件的价值才真正兑现。

## 我的理解 / 类比
路径决定你「进哪个房间」（页面），`meta.layout` 决定房间外面套什么「装修」（外壳）。同名壳文件 = 装修焊死在门上；Route Groups = 按走廊统一装修；layouts 插件 = 装修独立成「套装」，任何房间都能挑一套穿上，跟房间在哪条走廊无关。

## 关联
- [[嵌套路由]] —— 布局的本质就是嵌套路由在「页面外壳」上的应用（原理 ↔ 实践）
- [[Vue 自动路由方案]] —— 同一主题域，本卡是其「布局」维度的延伸
- [[路由元信息meta]] —— `meta.layout` 依赖路由 meta 机制
