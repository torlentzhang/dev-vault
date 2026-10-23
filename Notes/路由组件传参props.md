---
type: concept
tags: [vue, vue-router]
created: 2026-09-17
aliases: [路由 props, 路由组件传参, passing props, route props]
source: https://router.vuejs.org/zh/guide/essentials/passing-props.html
---

# 路由组件传参props

## 一句话定义
在路由配置里开启 `props`，把路由参数 / 查询等作为组件 `props` 传入，从而解耦组件与 `$route`、提升复用与可测试性。

## 用于解决什么问题
不给 props，组件就只能自己从 $route 取——params 全是字符串和 `string | string[]`，每个用到 id 的地方都要手动 Number() 和断言，默认值与校验也无处声明。

## 要点
- 布尔模式：`props: true` → 把 `route.params` 映射为组件 props；组件里 `defineProps({ id: String })` 直接用 `id`，不再依赖 `useRoute()` / `$route`。
- 对象模式：`props: { newsletterPopup: false }` → 把静态值原样传入，适合常量。
- 函数模式：`props: route => ({ query: route.query.q })` → 可把 params / query / hash 任意组合后传给组件，最灵活；尽量保持无状态。
- 命名视图：`components: { default, sidebar }` 时，每个命名视图单独配 `props: { default: true, sidebar: false }`。
- 价值：组件不再"只知道某个特定 URL"，可在任意地方复用、也更易单测。

## 我的理解 / 类比
像餐厅把"你要的菜"直接端到桌上（props），而不是让你自己跑进厨房看订单（`$route`）——服务员（路由）帮你取好递过来，你只管吃。

## 关联
- [[动态路由匹配]] —— `params` 通过 `props: true` 变成组件 prop
- [[组合式API：useRoute与useRouter]] —— 不传 props 时用 `useRoute` 的替代取法
