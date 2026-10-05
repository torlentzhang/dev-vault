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
- [[Vue Macros]] vue专属
- [[vite-plugin-pwa]] 产物生成类（Service Worker / Manifest，非钩子适配）
- [[vite-plugin-mock]] 接口模拟类（dev Connect 中间件 / prod mockjs 注入，非钩子适配）

## 插件开发
（resolveId / transform 实战、自定义插件发布 …）
