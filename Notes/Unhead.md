---
type: concept
tags: [vue, unhead, ssr]
created: 2026-10-23
aliases: [Unhead, @unhead/vue, @vueuse/head, useHead, 文档头管理]
source: https://unhead.unjs.io/docs/vue/head/guides/get-started/installation · https://unhead.unjs.io/docs/vue/head/guides/get-started/migrate-from-vueuse-head · https://vueuse.org/add-ons
---
# Unhead

## 一句话定义
Vue 的文档头（document head）管理器：把 `<head>` 里的 title / meta / link / script，以及 `<html>`、`<body>` 的属性纳入响应式管理，并让 SSR 与 CSR 共用同一套声明。它接替了已 Sunset 的 `@vueuse/head`。

## 要点

### 先分清：它不是 VueUse 的一部分
⚠️ `@vueuse/` 是 npm **scope（组织命名空间）**，不代表「属于 `@vueuse/core`」。同 scope 下是一堆各自独立的包：`@vueuse/core`、`@vueuse/head`、`@vueuse/router`、`@vueuse/math` …

- VueUse 官方对 core 的定位是 **lightweight and dependency-free**；包装第三方库的能力一律放 **Add-ons**，head 管理进不了 core 正是因为它会引入依赖。官方文档导航也把 Core Functions 与 Add-ons 分成两栏。
- **`@vueuse/head` 本身就是 unhead 的一层壳**：npm 上 `@vueuse/head@2.0.0` 的 dependencies 为 `@unhead/dom` / `@unhead/ssr` / `@unhead/vue` / `@unhead/schema`（均 `^1.7.0`）。它不是竞品，是 unhead v1 的包装。
- **`@vueuse/head` 已 Sunset**：仓库描述写着 `Powered by Unhead. - 🌇 Sunset`，官方迁移文档开篇即导向 Unhead；npm 上末版停留在 2023-09。新项目直接装 `@unhead/vue`（当前 latest `3.4.3`），等于跳过中间商。
- VueUse **本体**里 head 相关只有 `useTitle`、`useFavicon`（Core 的 Browser 分类）——零依赖单点操作（给 `document.title` 套 ref），不管 meta/link/script、不管 SSR、不做合并去重。`useMeta` 不在本体中；`@vueuse/head` 对外提供的是 `useHead` / `useHeadSafe` / `useSeoMeta` 与 `<Head>` 组件。

### 怎么用：入口装一次 + 组件里声明
```ts
// 入口（客户端）
import { createHead } from '@unhead/vue/client'
const head = createHead()
app.use(head)
```
```ts
// 组件内
useHead({
  title: 'Page Title',
  meta: [{ name: 'description', content: 'Page description' }],
})
```
字段 schema：`title`、`titleTemplate`、`templateParams`、`base`、`link[]`、`meta[]`、`style[]`、`script[]`、`noscript[]`、`htmlAttrs`、`bodyAttrs`。

### 响应式是它相对「手改 document.title」的核心价值
- 支持传 `ref`、`computed`、getter 函数，adapter 在 `watchEffect` 中解析，依赖变化时打补丁到同一条 entry；另有 `patch()` / `dispose()` 手动更新。
- **传 `.value` 是官方点名的 Common Mistake**——值传进去，响应性直接丢失。
```ts
const title = ref('首页')
useHead({ title })               // ✅ 传 ref
useHead({ title: title.value })  // ❌ 丢响应性
```

### 「SSR ready」的真实边界（最影响决策）
- 它解决的是「JS 应用需要管理 app 挂载点之外的标签」——`<head>` 标签、`<html>`/`<body>` 属性、body 开闭位置的内容，并统一 CSR 与 SSR。
- **但 SPA 下客户端的更新不改变服务器 / CDN 返回的 HTML**，社交预览类爬虫只读那份响应 HTML。公开页面应在 `index.html` 放**站点级默认值**并对公开路由**预渲染**；私有后台通常模板默认值就够。别把「用了 useHead」等同于「SEO 友好」。

### 从 @vueuse/head 迁移
```bash
npm remove @vueuse/head
npm install @unhead/vue
```
- `createHead` 改从 `@unhead/vue/client`（SPA）/ `@unhead/vue/server`（SSR）导入
- composable 从 `@unhead/vue` 导入，`<Head>` 组件从 `@unhead/vue/components` 导入
- auto-import 换成 `unheadVueComposablesImports`
- 装最新版会从 Unhead v1 直接跨到 v3；**Vue 2 必须锁 `@unhead/vue@^1`**（v2+ 要求 Vue 3）

### 常见坑
- **异步之后调用会丢上下文**：报 `useHead() was called without provide context.`（它靠 Vue provide/inject 取实例）。应在 setup 内先建 entry，之后用 `patch()`。
- **别在 `watch` 里调用**：每次触发都新建 entry；应建一次、让 ref 自己更新。
- **多次调用是「合并」不是「覆盖」**：多个组件的 `useHead()` 会被 dedupe/merge 后统一解析，最终结果要等渲染完成才知道。
- **实例管理**：客户端一个应用一个 head 实例；**SSR 必须每个请求新建 server head**，不能复用。
- **它不是消毒器**：别喂不可信输入，需要就用 `useSeoMeta()` / `useHeadSafe()`。
- **KeepAlive**：组件停用/启用时其 head entry 同步停用/启用。
- SSR 下默认注入 `<meta charset>`、viewport、`<html lang="en">`，可用 `disableDefaults` 关闭；`onRendered` 仅在客户端生效，且不作为渲染屏障。

### 与 Markdown frontmatter 的接点
`unplugin-vue-markdown` 的 `headEnabled: true` 让 `.md` frontmatter 控制文档头，底层正是把白名单字段（`title`、`meta`、`link`、`base`、`style`、`script`、`htmlAttrs`、`bodyAttrs`）**编译成一句注入的 `useHead()` 调用**——frontmatter 是编译期数据，最终 head 由运行时的合并规则决定。注意该插件依赖的也是 `@unhead/vue`（填 `"vueuse"` 会直接抛错终止构建）。详见 [[unplugin-vue-markdown]]。

## 我的理解 / 类比
把 `<head>` 想成整栋楼的门牌、公告栏和名片（标题、描述、分享预览图），而 Vue 组件只负责装修自己的房间——组件根本够不着门牌。Unhead 就是那套「门牌管理处」：任何房间（组件）都能提交自己的门牌信息，它统一收单、去重合并、再挂到楼外（`<head>`），而且**服务端预渲染时和浏览器接管后用的是同一套申请表**——这正是它相对「在组件里手改 `document.title`」的全部价值。

至于 `@vueuse/` 前缀，像同一个写字楼里的不同公司：门牌号（scope）相同不代表是同一家公司，`@vueuse/core` 是房东自营的轻量业务，head 管理则是外包给 unhead 做的专柜，后来专柜直接搬去 unhead 自己的门店了。

## 关联
- [[unplugin-vue-markdown]] —— `headEnabled` 把 md frontmatter 编译成注入的 `useHead()`，是「编译期数据 → 运行时调用」的实例
- [[Vue Hooks（组合式函数）]] —— `useHead` 是标准 composable：靠 provide/inject 取实例，所以异步后调用会丢上下文
- [[编译时与运行时]] —— frontmatter 在构建期被翻译成调用，head 最终形态在运行期才由合并规则决定
- [[路由元信息meta]] —— 另一条路线：标题存 `route.meta.title`，在守卫里读取，不依赖 head 库
