---
type: concept
tags: [vite, 前端工程化]
aliases: [unplugin 家族, 统一插件系统]
source: https://github.com/unjs/unplugin · https://unplugin.unjs.io/
created: 2026-09-22
---
# unplugin

## 一句话定义
unjs 生态（Nuxt Contrib）维护的插件接口抽象："Unified plugin system for build tools"——按 Rollup 风格写一份插件，自动适配 Vite / Webpack / Rspack / esbuild 等约十种构建工具；家族插件共用的 `unplugin-` 前缀即宣告这套机制。

## 要点

### 解决什么问题
- 各构建工具的插件 API 互不兼容（Rollup、Webpack、esbuild 自成一套），同一能力历史上要分别维护 `vite-plugin-*` / `webpack-plugin-*`
- unplugin 统一为一套钩子接口（Rollup 风格 `resolveId` / `load` / `transform`），作者写一份实现，由适配层转译给各工具——插件以 `unplugin-xxx/vite`、`/webpack` 等子路径暴露入口
- 官方 README 声明支持：Vite、Rollup、Webpack、esbuild、Rspack、Rsbuild、Rolldown、Farm、Bun，以及「构建在其上的每个框架」

### 通用性的边界（关键）
- unplugin 只解决**构建工具轴**；一个插件能否跨**前端框架**，由插件自身决定——这是两个独立的问题
- 家族示例（库内均有卡）：[[unplugin-auto-import]] 框架无关（标识符注入是语言层转换）；[[unplugin-icons]] 框架无关但需 `compiler` 选项指定产出框架；[[unplugin-vue-components]] **仅 Vue**（挂钩 SFC 模板编译，React 侧不存在「按名注册组件」这个环节）

### 现状（2026-09 核实）
- 仓库在 unjs org 下（© 2021–present，Nuxt Contrib），文档站 unplugin.unjs.io
- v3.0.0+ 要求 Node 20+（约束会传导到依赖它的插件，如 unplugin-icons v24+）

## 我的理解 / 类比
unplugin 是「万用电源转换头标准」：各国插座（bundler）形状不同，插件按这个标准做插头，配一套转换头到哪都能用。至于电器本身是通用电压（框架无关）还是专用电器（Vue-only），unplugin 不管——`unplugin-` 前缀只证明它兼容接口标准，不证明它跨框架。

## 关联
- [[unplugin-auto-import]] / [[unplugin-vue-components]] / [[unplugin-icons]] —— 抽象与实现：构建于其上的家族成员
- [[Vue 自动路由方案]] —— 家族另一成员（unplugin-vue-router）在路由领域的应用
