---
type: concept
tags: [vite, vue, markdown, 前端工程化]
aliases: [vite-plugin-vue-markdown, Markdown 转 Vue 组件]
source: https://github.com/unplugin/unplugin-vue-markdown · https://www.npmjs.com/package/unplugin-vue-markdown
created: 2026-10-21
---
# unplugin-vue-markdown

## 一句话定义
把 `.md` 文件编译成**真正的 Vue SFC（单文件组件）**的构建插件——编译产物是组件，不是 HTML 字符串，因此 md 文件可以直接 `import` 进来当组件用，md 正文里也能写 Vue 组件和 `{{ }}` 插值。原名 `vite-plugin-vue-markdown`，2023 年随 antfu 的 `vite-plugin-*` → `unplugin-*` 收编而改名。

## 要点

### 解决什么问题
- Vite 不认 `.md`：`import x from './a.md'` 会报未知扩展名；`.md?raw` 只能拿到**字符串**，拿不到组件
- 本插件在 `transform` 钩子上把 md 原文改写成一份 SFC 源码，再交回 `@vitejs/plugin-vue` 编译
- 双向能力（README 原话）：**Use Markdown as Vue components** / **Use Vue components in Markdown**——md 里可写 `<Counter :init="5"/>`、`<script setup>` 局部注册组件

### 使用：两个插件缺一不可（最常见的踩坑）
```ts
// vite.config.ts
import Vue from '@vitejs/plugin-vue'
import Markdown from 'unplugin-vue-markdown/vite'

export default defineConfig({
  plugins: [
    Vue({ include: [/\.vue$/, /\.md$/] }),   // ← 让 Vue 插件也接管 .md
    Markdown({ /* options */ }),
  ],
})
```
只装 Markdown 不配 `include`，`.md` 生成的 SFC 不会被 Vue 插件接管 → 编译不出组件。这是它「本质是源码改写器、不是编译器」的直接后果。

### 处理链路
```text
.md 原文
  → markdown-exit 渲染（默认 html/linkify/typographer 全开）
  → @mdit-vue/plugin-frontmatter 拆 YAML 头进 env
  → @mdit-vue/plugin-component 解析 md 里的 Vue 组件标签
  → 拼成 SFC 源码：<template> … </template> + <script> … </script>
  → @vitejs/plugin-vue 编译成组件
```

### 关键选项与默认值（据发布版类型声明与源码）
| 选项 | 默认 | 说明 |
| --- | --- | --- |
| `wrapperDiv` | `true` | 渲染结果外包一层 `<div>` |
| `wrapperClasses` | `'markdown-body'` | 该 div 的 class；`wrapperDiv: false` 时忽略 |
| `wrapperComponent` | `null` | 换自定义组件包裹内容，frontmatter 作为 prop 传入 |
| `frontmatter` | `true` | 解析 YAML 头，注入组件实例数据 `frontmatter` |
| `exposeFrontmatter` | `true` | `defineExpose` 暴露，模板里可直接 `{{ frontmatter.name }}` |
| `escapeCodeTagInterpolation` | `true` | 给 `<code>` 加 `v-pre`，防代码块里的 `{{ }}` 被当插值 |
| `headEnabled` | `false`（源码） | 配 `@unhead/vue`，用 frontmatter 的 `title`/`meta` 控制文档头 |
| `markdownOptions` / `markdownUses` / `markdownSetup` | — | 新名；旧名 `markdownItOptions` / `markdownItUses` / `markdownItSetup` 仍可用但 deprecated |
| `transforms.before` / `.after` / `.extraScripts` | — | 渲染前后的自定义改写钩子 |
| `vueVersion` | 自动探测 | 源码含 `isVue2` 分支，Vue 2 亦可 |

- 底层解析器是 **`markdown-exit`**（markdown-it 的 TypeScript 重写版，带异步渲染），**兼容 markdown-it 插件**，所以 `markdown-it-anchor`、`prism` 等可直接 `md.use()`。README 里仍有「markdown-it」的旧措辞，实际依赖已换。
- 默认 `markdownOptions` 在源码里被设成 `{ html: true, linkify: true, typographer: true }`，不是 markdown-it 的默认值。
- `headEnabled`：类型声明注释写 `@default true`，`resolveOptions()` 实际返回 `false` —— **以实现为准，默认关闭**。

### 现状（2026-10）
- `unplugin-vue-markdown`：`32.1.1`（2026-08-25），仓库 `unplugin/unplugin-vue-markdown` 未归档、末次 push 同日，约 626 stars，**活跃**。
- peer 依赖 `vite ^2 || ^3 || ^4 || ^5 || ^6 || ^7 || ^8`，**支持 Vite 8**（当前 Vite latest 8.3.5）。
- 周下载 125,341（2026-10-12~18），6 月以来日下载约从 1.1 万涨到 3.3 万。
- 旧包 `vite-plugin-vue-markdown`：末版 `0.23.8`（2023-08-14），**全部 23 个版本都带 npm `deprecated` 字段**，文案原文：`` `vite-plugin-vue-markdown` is renamed to `unplugin-vue-markdown`. For usages in Vite, you also need to change the import path to `unplugin-vue-markdown/vite`. ``；旧仓库 `mdit-vue/vite-plugin-vue-markdown` 已 301 重定向到新仓库；周下载 4,518。
- 迁移成本只有 import 路径一行：
  ```diff
  - import Markdown from 'vite-plugin-vue-markdown'
  + import Markdown from 'unplugin-vue-markdown/vite'
  ```
- README 声称「The same transformation as VitePress」；但 VitePress（1.6.4）自身并不依赖本包，二者是**转换结果对齐**，不是共用一份实现。

## 我的理解 / 类比
它像一台**把 Markdown 原稿改装成 .vue 文件的预处理器**，而不是一台「Markdown 渲染机」。关键在于它只把 md **翻译成另一种源文件**，后续编译仍由 Vue 编译器负责——所以它必须和 `@vitejs/plugin-vue` 同时在场：一个负责「改写成 SFC」，一个负责「把 SFC 编成组件」，缺后者就像把译稿交给不会排版的人。

## 关联
- [[unplugin]] —— 机制底座：本插件的 `/vite`、`/webpack`、`/rspack` 多入口正是由 unplugin 适配出来的，且它是 `vite-plugin-*` → `unplugin-*` 改名潮的一员
- [[编译时与运行时]] —— `.md` 的处理发生在构建期，属于「能在编译期确定就别拖到运行时」的典型
- [[unplugin-vue-components]] —— 常见搭配：其 README 要求本插件在前、`Components()` 在后，并放开 `extensions: ['vue', 'md']`
