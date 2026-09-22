---
type: concept
tags: [vite, 前端工程化]
aliases: [auto-import, API自动导入]
source: https://github.com/unplugin/unplugin-auto-import · https://api.npmjs.org/downloads/point/last-week/unplugin-auto-import
created: 2026-09-21
---
# unplugin-auto-import

## 一句话定义
unplugin 家族的构建插件：构建期转换代码时自动为常用 API 注入 import 语句（如不写 `import { ref } from 'vue'` 也能直接用 `ref`），并生成 `.d.ts` 保住 TypeScript 体验。

## 要点

### 工作机制
- 构建时拦截目标文件（`.ts` / `.vue` / `.jsx` …），扫描「用到但未导入」的标识符并按需注入 import——不污染全局、tree-shakable
- 生成 `auto-imports.d.ts` 为自动导入的 API 提供类型；还能生成 ESLint / Biome 的 globals 配置，避免 `no-undef` 误报
- 底层引擎是 `unimport`（unjs），本插件是其上层封装：presets、resolvers、目录扫描（`dirs` 把 composables 变成自动导入）等
- 是早年 `vue-global-api` 的继任者：按需转换、无全局污染

### 生态定位
- 一份配置跨 Vite / Webpack / Rspack / Rollup / Rolldown / esbuild / Astro 复用——与 unplugin-vue-components、unplugin-vue-router、unplugin-icons 同属「unplugin」模式
- 框架无关（官方 README 即有 React `useState`、Svelte 示例），但生态重心在 Vue：presets 以 vue / vue-router / @vueuse/core 为主，最常与 `unplugin-vue-components` 成对出现（一个管 API，一个管组件）
- **Nuxt 3+ 已内置 auto-import（同样基于 unimport），Nuxt 项目无需单独安装**；纯 Vite + Vue 项目才需要手动引入

### 现状（2026-09 核实）
- npm 周下载约 93 万，Vite + Vue 生态的事实标准插件之一
- v21.x 活跃维护（antfu / sxzz / hannoeru 等），仓库已归入 unplugin org，Rolldown 等新工具链均已适配

### 社区争议（观点，非事实）
「自动导入 vs 显式 import」争论长期存在：反对者认为损害可读性、提高新人追溯符号来源的成本；插件以生成 d.ts 与 lint 配置缓解。属于选型偏好问题，不影响其高使用率。

## 我的理解 / 类比
像给构建工具请了个「代写 import」的人：源码层面当 API 是全局的来写，产物仍是标准 ESM——便利留在开发期，运行时不付代价。它与 Nuxt 内置 auto-import 的关系是「同一个引擎（unimport），两种外壳」：Nuxt 把能力装进框架，纯 Vite 项目用插件装。

## 关联
- [[vue自动路由方案]] —— 同为 unplugin 家族方案（unplugin-vue-router），Vite 插件层常一起配置
- [[vite+Vue3工程的初始化与启动]] —— 工程初始化后的常用增强插件，属 Vite 工程实践环节
