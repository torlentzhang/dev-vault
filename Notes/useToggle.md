---
type: concept
tags: [vue, vueuse, composable]
created: 2026-10-13
aliases: [useToggle]
source: https://vueuse.org/core/useToggle/
---
# useToggle

## 一句话定义
VueUse 提供的布尔状态开关：把「一个布尔 ref + 一个切换函数」封装成组合式函数，省去手写样板。

## 用于解决什么问题
每处需要一个布尔开关，都要先 `ref(false)` 再手写一个取反函数，组件多起来全是重复代码；状态已经存在别处时，补一个切换器还得把取反逻辑再抄一份。

## 要点
- **两种返回形态，由入参是否为 ref 决定**（源码里就是 `isRef(initialValue)` 一个判断）：
  - `useToggle()` / `useToggle(true)` → 返回 `[ref, toggle]`；传普通值也走这条分支
  - `useToggle(someRef)` → 只返回 `toggle` 函数本身，状态复用传入的 ref
- **toggle 有参即 set，无参才 toggle**，且返回切换后的新值：
  ```ts
  const [value, toggle] = useToggle()
  toggle()       // true → 切换
  toggle(true)   // 直接设值
  ```
- **传 ref 的经典搭配**：`const isDark = useDark(); const toggleDark = useToggle(isDark)` —— 状态已有，只补一个切换器。
- **可自定义 truthy/falsy 值**（`MaybeRefOrGetter`，内部用 `toValue()` 解析，可传 ref，每次切换重新取值）：
  ```ts
  const [value, toggle] = useToggle('on', { truthyValue: 'on', falsyValue: 'off' })
  ```
- **模板陷阱（官方文档 Caution）**：`@click="toggle"` 不加括号时，MouseEvent 会被当作第一个参数传入，toggle 变成 set，切换逻辑被绕过。必须写 `@click="toggle()"`。
- **多次调用完全独立**：每次调用重建自己的 `shallowRef` 与闭包，多个 useToggle 互不影响（见 [[Vue Hooks（组合式函数）]] 的"每次调用 = 独立状态"）。
- 内部用 `shallowRef` 实现——布尔值不需要深度响应式。

## 我的理解 / 类比
手写布尔开关像每次自己拧灯绳；useToggle 是把灯绳和灯装成一体的开关面板——还送你一个"传入已有灯也能只买开关"的用法。而 `@click="toggle"` 不加括号，就像把递过来的信封（$event）硬塞进开关槽位，开关直接失灵。

## 关联
- [[Vue Hooks（组合式函数）]] —— useToggle 是它的一个标准实例：每次调用独立状态、返回响应式值
