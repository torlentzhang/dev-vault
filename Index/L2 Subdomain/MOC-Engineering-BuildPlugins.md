---
type: moc
tags: [moc, engineering]
created: 2026-09-22
---
# 构建插件 · MOC

> 子领域地图：构建工具插件体系的入口——机制、具体插件、插件开发。

## 插件机制
- [[unplugin]]
（构建工具钩子模型、插件开发指南 …）

## unplugin 系列
（同一套 unplugin 适配层下的插件，多构建工具入口来自它）
- [[unplugin-auto-import]]
- [[unplugin-vue-components]] vue专属
- [[unplugin-icons]]
- [[unplugin-vue-markdown]] vue专属（md → Vue SFC，旧名 vite-plugin-vue-markdown）

## 其他常用插件
- [[Vue Macros]] vue专属
- [[vite-plugin-pwa]] 产物生成类（Service Worker / Manifest，非钩子适配）
- [[vite-plugin-mock]] 接口模拟类（dev Connect 中间件 / prod mockjs 注入，非钩子适配）
- [[px 转 vw 的 PostCSS 插件]] 单位转换类（移动端 vw 适配，非钩子适配）

## 插件开发
（resolveId / transform 实战、自定义插件发布 …）

## Markdown 与内容处理
- [[markdown-it]] 解析器本体（token 流 + renderer，md → HTML）
- [[unplugin-vue-markdown]] md → Vue SFC 编译器，vue专属（旧名 vite-plugin-vue-markdown）
- [[Markdown 代码高亮]] 代码块着色层（markdown-it-prism，样式表需自备）
- [[Unhead]] 内容站的文档头（frontmatter + `headEnabled`）
