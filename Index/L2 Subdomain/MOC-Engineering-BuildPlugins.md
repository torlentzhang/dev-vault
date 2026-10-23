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

## 常用插件
- [[unplugin-auto-import]]
- [[unplugin-vue-components]] vue专属
- [[unplugin-icons]]
- [[unplugin-vue-markdown]] vue专属（md → Vue SFC，旧名 vite-plugin-vue-markdown）
- [[Vue Macros]] vue专属
- [[vite-plugin-pwa]] 产物生成类（Service Worker / Manifest，非钩子适配）
- [[vite-plugin-mock]] 接口模拟类（dev Connect 中间件 / prod mockjs 注入，非钩子适配）
- [[px 转 vw 的 PostCSS 插件]] 单位转换类（移动端 vw 适配，非钩子适配）

## 插件开发
（resolveId / transform 实战、自定义插件发布 …）

## Markdown 与内容处理
- [[markdown-it]] 解析器本体（token 流 + renderer，md → HTML）
