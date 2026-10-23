---
type: concept
tags: [vite, 前端工程化, 图标]
aliases: [unplugin-icon, vite-plugin-icons, IconsResolver, 图标自动导入]
source: https://github.com/unplugin/unplugin-icons
created: 2026-09-22
---
# unplugin-icons

## 一句话定义
unplugin 家族的构建插件（Anthony Fu）：把 Iconify 图标在构建期编译成按需的框架组件——代码里写 `~icons/mdi/account`（或配合自动导入直接写 `<i-mdi-account />`），只有实际用到的图标进入 bundle。

## 用于解决什么问题
模板里要摆图标时，没有它就得逐个 import 注册组件，或先把 SVG 下载进仓库再当静态资源引用；图标一多，引用路径散落各处，换图标集只能全局搜替换。

## 要点

### 工作机制
- 以虚拟模块 `~icons/{collection}/{icon}` 寻址，构建时从本地安装的 Iconify 数据（`@iconify/json` 全集或 `@iconify-json/*` 单集包）取出 SVG，按 `compiler` 选项编译成目标框架组件
- 框架无关：Vue 3 / Vue Vapor / React（jsx）/ Preact / Svelte / Solid / Astro / Qwik / Ember；构建工具跨 Vite / Webpack / Rspack / Rollup / esbuild / Nuxt
- 按需打包：装全集数据（约 120MB）也只把用到的图标编译进产物；SSR / SSG 友好，图标随页面下发、无 FOUC；无运行时远程请求，可离线
- 图标可样式化：颜色默认走 `currentColor`、尺寸随 `font-size`（默认 scale 1.2 即 1.2em），像文本一样用 CSS 控制

### 与 Iconify 的分工
- Iconify 是数据层（统一 IconifyJSON 格式，官方称约 150 个流行图标集、20 万+ 图标），本插件是构建期消费层的一种实现——消费方式会变，数据层不变
- 与「组件包」路线（如 lucide-react 显式 import）对照：组件包赢在类型安全与来源可追溯，unplugin-icons 赢在跨图标集统一写法 + 全集按需，两条路线并存

### 自动导入组合
- Vue 3：配 `unplugin-vue-components`——`Components({ resolvers: [IconsResolver()] }) + Icons()`，模板直接写 `<i-carbon-accessibility />`；命名规则 `{prefix}-{collection}-{icon}`，默认前缀 `i`
- React / Solid：配 `unplugin-auto-import`——`IconsResolver({ prefix: 'Icon', extension: 'jsx' })`
- 支持自定义图标集（`customCollections` / `FileSystemIconLoader`，本地 SVG 目录可组件化并支持 HMR）、`?raw` 导入原始 SVG 字符串、`autoInstall` 缺集时自动安装（实验性）

### 工程注意事项
- ESM-only 包；v24+ 要求 Node 20+（跟随 unplugin v3）
- 前身是 `vite-plugin-icons`，旧前缀 `virtual:vite-icons/...` 已废弃，统一用 `~icons/`
- TS 支持靠在 `tsconfig.json` 的 `compilerOptions.types` 加 `unplugin-icons/types/vue`（或 react / svelte 等）
- 图标名可在线检索：icones.js.org

## 我的理解 / 类比
Iconify 是「图标的 USB 标准」，unplugin-icons 就是把「插上的设备」当场编译出来的那头：开发期按名字取图标，构建期只把用到的那几个锻造成组件，20 万个图标的仓库留在 node_modules 里不进包。它与 auto-import（管 API）、vue-components（管组件）凑齐 unplugin 三件套的「管图标」一角。

## 关联
- [[Iconify]] —— 上下游关系：Iconify 是数据层，本插件是构建期消费层的实现之一
- [[unplugin-vue-components]] —— 标准组合：IconsResolver 挂进其 resolvers，实现 Vue 模板图标自动导入
- [[unplugin-auto-import]] —— React / Solid 侧的图标自动导入走它的 resolvers 机制
