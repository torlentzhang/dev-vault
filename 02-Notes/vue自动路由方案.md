---
type: concept
tags: [vue, vue-router]
aliases: [Vue文件路由方案, Vue自动路由对比]
source: https://github.com/hannoeru/vite-plugin-pages · https://github.com/posva/unplugin-vue-router · https://router.vuejs.org/
created: 2026-09-17
---

# Vue 自动路由方案

## 一句话定义
Vue 生态里「以文件系统（目录/文件名）自动生成路由表」的一类方案，省去手写 routes 数组的样板；主流有三种，按推荐度递进。

## 三种方案对比

| 方案 | 作者 | 框架支持 | 类型推导 | 现状 |
|------|------|----------|----------|------|
| vite-plugin-pages | hannoeru | Vue3 / React / Solid（框架无关） | 弱 | 仍活跃但发版放缓（v0.33.3 / 2026-02）；**Vue 方向官方 README 已劝退**，建议改用 unplugin-vue-router |
| unplugin-vue-router（独立包） | @posva（vue-router 作者） | 仅 Vue | **强**：自动生成 typed-router.d.ts，useRoute/useRouter 带路径与参数类型 | 从 vue-router/auto 导入，与 vue-router 深度集成；Vue 赛道的「官方继任者」 |
| Vue Router 5（内置） | Vue 官方 | 仅 Vue | 强（继承自 unplugin） | 2026-03 把 unplugin-vue-router 合并进核心；文件路由从 vue-router 自身导入、无需独立依赖；对 v4 无破坏性变更；同步带来实验性 Data Loaders、Volar 支持、route JSON schema 校验；是通往 Vue Router 6（纯 ESM）的过渡版 |

## 选型建议（Vue 项目）
- 新项目首选 **Vue Router 5**（依赖链最简、跟官方主线）；若担心 5.0 太新，退而用 **Vue Router 4 + unplugin-vue-router 独立包**，体验几乎一致，未来升 5 只改 import。
- vite-plugin-pages 在 **Vue 方向不推荐**（官方已劝退、类型推导弱）；它仅适合 React / Solid 用户。

## 我的理解 / 类比
文件路由就是「目录即路由表」：把 src/pages 下的文件结构直接映射成 URL，开发者只管建文件，路由表自动长出来。三种方案本质是「谁来做这份映射 + 能不能顺手给你类型」的演进——vite-plugin-pages 是通用初代，unplugin-vue-router 是 Vue 专属加强版，Vue Router 5 把它收编为正统。

## 关联
- [[前端路由是什么]]
- [[vite+Vue3工程的初始化与启动]]
