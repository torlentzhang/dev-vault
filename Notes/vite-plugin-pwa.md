---
type: howto
tags: [vite, pwa, 前端工程化]
aliases: [VitePWA, Vite PWA 插件]
source: https://vite-pwa-org.netlify.app/ · https://github.com/vite-pwa/vite-plugin-pwa
created: 2026-10-02
---
# vite-plugin-pwa

## 适用场景
在 Vite / Vue / React / Svelte 工程里让应用「可离线 + 可安装 + 可更新」，但不想手写、也不想长期维护一份 Service Worker 代码。插件在构建期接管这件事；它和 [[PWA]] 的关系是：**PWA 解决「是什么」，本卡解决「怎么在 Vite 里落地」**。

## 本质
对 `workbox-build` 的一层封装：构建时扫描 `dist` → 生成 `sw.js` 与 `manifest.webmanifest` → 往 HTML 入口注入注册脚本。

## 步骤

### 1. 安装并注册插件
```ts
// vite.config.ts
import { defineConfig } from 'vite'
import { VitePWA } from 'vite-plugin-pwa'

export default defineConfig({
  plugins: [
    VitePWA({
      registerType: 'autoUpdate',
      workbox: {
        globPatterns: ['**/*.{js,css,html}'],
      },
      manifest: {
        name: 'My App',
        short_name: 'MyApp',
        theme_color: '#ffffff',
        display: 'standalone',
        icons: [
          { src: 'pwa-192x192.png', sizes: '192x192', type: 'image/png' },
          { src: 'pwa-512x512.png', sizes: '512x512', type: 'image/png' },
          { src: 'pwa-maskable-512.png', sizes: '512x512', type: 'image/png', purpose: 'maskable' },
        ],
      },
    }),
  ],
})
```
图标须放在 `publicDir`（默认 `public/`）。

### 2. 注册 Service Worker（可选）
默认靠 `injectRegister` 自动注入脚本；要自己控制更新提示，在入口文件用虚拟模块：
```ts
import { registerSW } from 'virtual:pwa-register'

const updateSW = registerSW({
  onNeedRefresh: () => updateSW(true),   // 用户点了「刷新」
  onOfflineReady: () => console.log('离线就绪'),
  onRegisterError: (err) => console.error(err),
})
```
Vue / React 侧另有 `virtual:pwa-register/vue`、`/react` 等响应式封装。

### 3. 选策略
| strategies       | 产出方式                       | 适合                                  |
| ---------------- | -------------------------- | ----------------------------------- |
| `generateSW`（默认） | Workbox 按配置**自动生成** sw.js  | 绝大多数场景：静态资源离线、运行时缓存                 |
| `injectManifest` | 用**你自己的** sw.js，插件只注入预缓存清单 | 需要 push / background sync / 自定义路由判断 |

`injectManifest` 下自定义 SW 至少要有（源码默认位置 `public/sw.js`，可用 `srcDir` + `filename` 改）：
```js
import { precacheAndRoute } from 'workbox-precaching'

precacheAndRoute(self.__WB_MANIFEST)
```
想配合 `autoUpdate`，还得在 SW 里 `self.skipWaiting()` + `clientsClaim()`。

### 4. 用生产构建验证
`dev` 下默认不装 SW，直接看 dev server 会得到误导性结论。跑 `vite build` + `vite preview` 再在 DevTools → Application 里核对。

## 关键选项与默认值
| 选项 | 默认 | 说明 |
| --- | --- | --- |
| `strategies` | `generateSW` | 见上表 |
| `registerType` | `prompt` | `autoUpdate` 会**强制** `skipWaiting` + `clientsClaim` 为 true |
| `injectRegister` | `auto` | `auto` / `inline` / `script` / `script-defer` / `null`；`null` 表示完全手动注册 |
| `workbox.globPatterns` | `['**/*.{js,css,html}']` | **覆盖时必须带上 `html`**，否则入口不在预缓存里直接构建失败 |
| `workbox.maximumFileSizeToCacheInBytes` | 2 MiB | 超出的资源静默不进预缓存 |
| `workbox.cleanupOutdatedCaches` | 开（generateSW） | 自动清掉旧版本缓存 |
| `includeManifestIcons` | `true` | manifest 里声明的图标自动进预缓存 |
| `devOptions.enabled` | `false` | dev 要测 SW 必须显式打开 |
| `selfDestroying` | `false` | 生成「自我注销」的 SW，用于下线 PWA |

## 注意事项 / 踩坑

- **SPA 的 `navigateFallback` 是默认就开着的。** `generateSW` 会自动注册一条 `NavigationRoute` 把导航请求兜底到 `index.html`，所以 vue-router 直接刷新子路由本来就能离线打开，**不用也不该**照网上教程手写 `navigateFallback: 'index.html'`。
  - 反过来，SSG 站点（VitePress / Valaxy 等）每路由各有真 HTML，这条兜底会把文章页顶成首页 HTML，导致水合错乱、布局塌陷 → 要显式 `workbox.navigateFallback: null` 关掉，让未命中的请求回源。
  - 服务端 history fallback 与这个兜底是同一件事的两面，见 [[hash模式与history模式]]。
- **更新行为要二选一**：`autoUpdate` 装完就接管并重载页面（开发者省心、用户无感）；`prompt` 装完停在 waiting，等你调用 `updateServiceWorker()` 才生效（适合需要弹窗提示的版本）。二者混着写会出现「提示了但页面不更新」。
- **移除 PWA 不能只删配置。** 已经装到用户浏览器里的 SW 不会因为配置消失而注销 —— 要先加 `selfDestroying: true` 构建部署一次（文件名、路径**不能改**，否则老 SW 认不出替换者），确认生效后再移除集成。
- **auth 站点会拿不到 manifest。** 浏览器请求 manifest 时不带凭据，站点若整体鉴权会返回 401；开 `useCredentials: true` 给 `<link rel="manifest">` 加 `crossorigin="use-credentials"`。
- **SSR / SSG 下别在模块顶层 import 虚拟模块**，它在构建期会于服务端被执行而撞 `navigator is undefined`。改成动态 import，或等 `router.isReady()` 之后再注册。
- **`injectManifest` 自定义 SW 若根本不用预缓存**，要把注入点关掉（`injectionPoint: undefined`），否则构建会因找不到 `self.__WB_MANIFEST` 报错。
- **dev 模式策略有差别**：`generateSW` 强制产出 classic 格式；`injectManifest` 可用 `type: 'module'`，SW 里能直接写 `import`。dev 下只预缓存入口，避免拦截全部资源把 HMR 搞坏。
- CDN / 第三方字体这类跨源资源不是构建产物，得走 `workbox.runtimeCaching` 单独配，且 HTML 里引它们要带 `crossorigin="anonymous"`，否则拿不到缓存响应。

## 我的理解 / 类比
`vite-plugin-pwa` 不是让你少写一个文件，而是让你**不再承担维护 Service Worker 的生命周期**：你只描述「哪些文件预缓存、什么请求走什么缓存策略、更新时是自动重载还是弹窗问用户」，构建期插件把这些翻译成一份带 revision 哈希的 sw.js。和 [[unplugin]] 那类「抽象构建工具接口」的插件不同，它是一条**业务能力管线**，不是钩子适配层。

## 关联
- [[PWA]] —— 能力本体（HTTPS / Manifest / Service Worker）与本卡是「原理 → 落地」的关系，先读前者再读本卡更顺
- [[hash模式与history模式]] —— 导航兜底到底该由服务端还是 SW 承接，取决于路由方案
- [[unplugin]] / [[MOC-Engineering-BuildPlugins]] —— 同为 Vite 侧插件，但 unplugin 抽象的是构建钩子，本卡抽象的是产物生成
- [[MOC-Web]]
