---
type: moc
tags: [moc, framework]
created: 2026-08-23
---
# Frontend · MOC

> 领域地图：如何构建现代前端应用（框架、组件库与工程实践）的入口。

## 框架（Vue / React）
（响应式原理、组件通信、状态管理 …）

### Vue
- [[MOC-Frontend-Vue]]
（路由 / 工程实践 / 响应式 / 组件通信 …）

## 渲染模式
（CSR / SSR / SSG / ISR、水合、首屏与 SEO …）
- [[服务端渲染]] 渲染链路光谱与 SSR/SPA 取舍，先看这张
- [[水合（Hydration）]] SSR 链路的后半段：接管已有 HTML，而不是重画
- [[SSR 双端构建]] 一份源码为什么编两次、两端产物差在哪

## 工程实践
（脚手架、自动导入、请求层分层 …）
- [[axios 接口层封装]] 实例 + 拦截器由 `utils/axios.ts` 出，service 层只做业务归类

## 交互 / UI 组件库
（滑块、拖拽、图表等前端交互能力库 …）
- [[Swiper]]
