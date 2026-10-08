---
type: concept
tags: [vue, composable]
created: 2026-10-08
aliases: [组合式函数, composables, use 函数, vue hooks]
source: https://cn.vuejs.org/guide/reusability/composables.html
---

# Vue Hooks（组合式函数）

## 一句话定义
以 `use` 开头、封装可复用**有状态逻辑**的函数——Vue 对应 React Hooks 的复用方案，用来替代 mixin。

## 要点
- **命名约定**：`use` 开头。这是社区约定，方便人眼与团队规范识别；自动导入工具并不靠前缀匹配（见 [[unplugin-auto-import]]）。
- **本质是普通函数**：状态由响应式系统承载、不靠调用顺序，调用位置灵活；但 `onMounted` / `inject` 等**依赖组件实例的 API 只能在 `setup` 中同步调用**。
- **每次调用 = 独立状态**：函数体里的 `ref()` / `reactive()` 随调用重建，多组件互不影响。
- **返回响应式值**：返回 `ref` 模板自动解包；解构 `ref` 会失响应式，返回 `reactive` 对象时配 `toRefs()`。
- **副作用清理**用 `onScopeDispose()`（组件外的 `effectScope` 也有效），`onUnmounted` 是组件内特化版。
- 与 React Hooks 对比：普通函数 vs 调用顺序链表；细粒度响应式，无需 `useMemo` / `useCallback`。

```js
// useMouse.js
import { ref, onMounted, onUnmounted } from 'vue'

export function useMouse() {
  const x = ref(0)
  const y = ref(0)
  function update(e) {
    x.value = e.pageX
    y.value = e.pageY
  }
  onMounted(() => window.addEventListener('mousemove', update))
  onUnmounted(() => window.removeEventListener('mousemove', update))
  return { x, y }
}
```

## 我的理解 / 类比
mixin 像把整页笔记复印贴进每份报告——来源不清、还可能撞名；composable 像按需引用的函数，要哪段取哪段，每次调用独立取材。

## 关联
- [[组合式API：useRoute与useRouter]] —— 最典型的 composable 实例
