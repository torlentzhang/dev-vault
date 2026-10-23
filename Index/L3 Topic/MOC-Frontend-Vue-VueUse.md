---
type: moc
tags:
  - moc
  - vue
  - vueuse
created: 2026-10-13
source: https://vueuse.org/guide/
---
# VueUse · MOC

> 主题地图：VueUse 组合式工具集的入口（状态 / 响应式进阶 / 元素与鼠标 / 浏览器 / 网络 / 工具函数 …）。
> 卡片积累中，分类结构会随卡片增长调整。

## 状态
- [[useToggle]]
（布尔 / 数字 / 存储类状态工具，如 useLocalStorage …）

## 响应式进阶
（watch / ref 增强类工具，如 useDebounceFn、watchDebounced …）

## 元素与鼠标
（DOM 元素测量、鼠标/滚动交互，如 useElementBounding、useMouse …）

## 浏览器
（浏览器 API 封装，如 useClipboard、useWindowSize、useDark …）

## 网络
（请求 / 在线状态，如 useFetch、useOnline …）

## 工具函数
（与 Vue 响应式无关的通用工具增强 …）

## 文档头（head 管理）
- [[Unhead]] —— ⚠️ 不属于 VueUse 本体：`@vueuse/head` 只是 unhead v1 的一层外包壳（已 Sunset），真实接棒者是 `@unhead/vue`。此处仅作历史入口导流
（Core 本体内的 head 相关工具只有 `useTitle` / `useFavicon`，见「浏览器」小节待补）
