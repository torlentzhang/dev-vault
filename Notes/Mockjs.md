---
type: concept
tags: [mock, 前端工程化]
created: 2026-10-04
aliases: [Mock.js, mockjs]
source: nuysoft/Mock（GitHub）；现状依据多方检索
---

# Mockjs

## 一句话定义
Mock.js（nuysoft/Mock）是**前端开发期的数据模拟库**：用模板语法生成随机数据，并劫持 `XMLHttpRequest` 拦截匹配 URL 的请求、在内存中直接返回，让前端在后端接口未就绪时也能联调页面。

## 用于解决什么问题
接口还没就绪，前端要么干等，要么把假数据硬编码进组件、在每个请求处手写一层 if；真接口上线后这些临时代码要一处一处翻出来删，漏一个就带上生产。

## 要点
- **三大能力**：随机数据生成、XHR 请求拦截、数据校验（`Mock.valid` 比对真实返回值与模板）
- **DTD 数据模板定义**：`'list|10'` 生成 10 个元素；`'list|1-10'` 随机 1~10 个；`'id|+1': 1` 自增 ID
- **DPD 数据占位符**：`@cname`（中文名）、`@email`、`@integer(1,100)`、`@date`、`@image` 等
- **拦截机制**：重写 `XMLHttpRequest.prototype.open/send`，匹配 URL 后在内存模拟响应
  - ✅ 拦 **XHR**（axios 默认走 XHR adapter，因此能被拦到）
  - ❌ **不拦原生 `fetch`**（旧版）
  - ⚠️ 响应不发真实网络请求，**DevTools 的 Network 面板不可见**
- **现状**：约 2019 年后停止积极维护，停留在 `1.0.1-beta3`；对 Vite / fetch / TypeScript 兼容性一般，**新项目不建议选用**
- **现代替代**：`MSW`（Service Worker 拦真实网络层，fetch/XHR 通吃、Network 可见）、`@faker-js/faker`（纯造数）、`vite-plugin-mock`（Vite 构建期中间件）、`json-server`（独立 REST 服务）

## 我的理解 / 类比
像在后厨门口安了个**「替身厨师」**：你点菜（发请求）时，单子不真送进餐厅后厨（真实服务器），替身直接在门口给你炒一份假的端上来。因为根本没走真实网络，所以 DevTools 的 Network 面板里看不到这次"点单"——这既是它上手简单的原因，也是调试体验差的 root cause（MSW 改用 Service Worker 拦在真实网络层，正好补上这点）。

## 关联
- [[vite-plugin-mock]] —— 现代 Vite 工程化实现；Mockjs 是其思想来源与 prod 模式可注入的库，二者是「库 → 插件」的上下游关系
