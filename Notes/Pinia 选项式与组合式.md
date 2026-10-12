---
type: concept
tags: [vue, pinia]
created: 2026-10-12
aliases: [选项式 store, 组合式 store, option store, setup store, defineStore 写法]
source: https://pinia.vuejs.org/core-concepts/
---

# Pinia 选项式与组合式

## 一句话定义
`defineStore()` 的两种写法：选项式用 `state / getters / actions` 三段配置对象，组合式（setup store）直接写一个 setup 函数——两者能力完全等价，只是组织代码的方式不同，底层转换成同一套 store 结构。

## 要点
- **核心映射规则**：setup store 中返回的 `ref` / `reactive` → state，`computed` → getter，普通函数 → action。Pinia 自动对 `ref` **解包（unwrap）**，所以 `store.count` 直接取值，不写 `.value`（与组件模板中的解包是同一机制）。
- **共享同一套实例 API**：`$patch`、`$subscribe`、`$onAction`、`$state`、`$id` 在两种写法下行为一致，组件中的使用方式零区别。
- **`$reset()` 是选项式的内置方法**（内部重新调用 `state()` 生成初始状态）。组合式 store 上调用它会直接抛错，需自己实现并 return 回去：
  ```ts
  function $reset() { count.value = 0 }
  return { count, $reset }
  ```
- **选项式 `state()` 必须声明全部初始值**，不能事后新增属性（同组件 `data`）；初始值为空数组 / `null` 时需 `as` 辅助类型推断，组合式写法天然完整推断。
- **选型判断**：数据增删改为主 → 选项式够用；需要复用 composable、复杂派生 / 异步逻辑、重度依赖 TS → 组合式。两者可在同一项目共存，不同 store 各选各的。
- 两种写法可混用于同一项目，但**单个 store 不能混写**。

```ts
// 选项式
defineStore('cart', {
  state: () => ({ items: [] }),
  getters: { total: (state) => state.items.length },
  actions: { add(item) { this.items.push(item) } },
})

// 组合式：同构关系一目了然
defineStore('cart', () => {
  const items = ref<Item[]>([])
  const total = computed(() => items.value.length)
  function add(item: Item) { items.value.push(item) }
  return { items, total, add }
})
```

## 我的理解 / 类比
选项式像填表格——固定栏目，照着填即可；组合式像写一个普通函数，把成果 `return` 回去——**交什么就变成什么**：`ref` 变数据、`computed` 变派生值、函数变方法，Pinia 负责统一拆封。

## 关联
- [[Vue Hooks（组合式函数）]] —— 组合式 store 本质就是把 composable 的写法用于定义 store
