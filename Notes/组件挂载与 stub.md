---
type: concept
tags: [测试, 单元测试, Vue, VTU, stub]
created: 2026-11-09
aliases: [mount, shallowMount, Vue Test Utils, VTU, 浅挂载, 组件挂载, stubs]
source: https://test-utils.vuejs.org/api/ · https://test-utils.vuejs.org/guide/advanced/stubs-shallow-mount · https://test-utils.vuejs.org/migration/
---
# 组件挂载与 stub

## 一句话定义
`mount` 把组件连同**整棵子树**真实挂载；`shallowMount` 把子组件全部换成空壳再挂载。二者不是两个函数，是**同一个挂载动作的两个档位**——官方原话：*shallowMount(Component) is an alias of mount(Component, { shallow: true })*。

> 归属：`mount` / `shallowMount` 是 **Vue Test Utils（VTU）** 的 API，不是 Vitest 的。Vitest 负责组织用例与断言（`describe` / `it` / `expect`），挂载由 VTU 负责。

## 用于解决什么问题
组件测试要验的是「本组件的行为」，但本组件一渲染就把整棵子树拖进来：子组件发请求、要 router、要 store，测试还没开始断言就先被环境问题绊住。stub 提供的是一个**隔离旋钮**：调多少，测试就有多接近真实。

## 要点

### 两者的真实差别只有一件事：子组件是否被替换
| | 子组件 | 渲染层级 |
| --- | --- | --- |
| `mount` | 真实渲染 | 整棵组件树 |
| `shallowMount`（= `mount` + `shallow: true`） | 全部 stub | 只有本组件 |

### 官方给的是权衡，不是硬规则
官方在 "mount, shallow and stubs: which one and when?" 里的表述：

- *the more your tests resemble the way your software is used, the more confidence they can give you*
- `mount`：*render the entire component hierarchy, which is closer to what the user will experience in a real browser*
- `shallow`：*focused on a specific component … useful for testing advanced components in complete isolation*
- 关键的取舍建议：*If you just have one or two components that are not relevant to your tests, consider using mount in combination with stubs instead of shallow. The more you stub, the less production-like your test becomes.*

### 实际操作怎么选
| 场景 | 选择 |
| --- | --- |
| 只测本组件的渲染输出、props、emit | `shallowMount`，快且不受子树干扰 |
| 要验证父子协作（子组件触发 → 父组件响应） | `mount` |
| 只有一两个无关子组件 | `mount` + `stubs` 精确排除，不要用 shallow 一刀切 |
| 子组件很重（发请求、依赖 router / store） | `shallowMount`，再按需 `emit` 打桩 |
| 要断言子树里的真实 DOM / 文案 | 必须 `mount`（见坑 ①） |

### 三个必须先知道的坑

**① shallow 下插槽根本不渲染**
官方原文：*any `<slot>` won't get rendered when using shallow … you won't be able to verify the correct text is rendered!*
所以「用 shallowMount 断言按钮上的文案」必然拿不到——文案在子组件的 slot 里，而子组件被换成了空壳。
变通是 `config.renderStubDefaultSlot`，但官方明确限制：*cannot be extended to slots other than the default one*（只管默认插槽）。

**② v2 改了行为，从 v1 迁来的测试会挂**
迁移页原话：*This behavior is corrected in v2 - the slot content for a stubbed component is not rendered.*
VTU v1 会渲染 stub 组件的默认插槽内容，**v2 不渲染**。这是跨版本迁移最容易踩空的一处。

**③ `trigger` 造不出被 stub 子组件的事件**
官方原文：*trigger only dispatches DOM events, so it cannot simulate a custom event emitted by a child component, for example one stubbed by shallowMount. Use `emit` instead.*

```js
// 子组件已被 stub，trigger 打不动它，改用 emit
await wrapper.findComponent(DataGrid).emit('ready', payload)
```

### 更细的控制：`stubs`
`shallow: true` 是一刀切，`stubs` 是按名指定、还能给自定义实现：

```js
// 全 stub，但放行一个
mount(ComplexComponent, {
  shallow: true,
  global: { stubs: { ComplexA: false } }
})
```

- 三种写法：`['Foo']` / `{ Foo: true }` / `{ Foo: CustomStub }`
- 作用于**整棵渲染树**，所以放在 `global` 里；可用 `config.global.stubs` 对整套测试生效
- 另一处易忽略：`global.stubs` **默认就会 stub `Transition` / `TransitionGroup`**，与是否 shallow 无关

### 不是废弃 API
官方 TIP 原话：*That method is still available, too - it's the same as writing `shallow: true`.* `shallowMount` 在 v2 仍是合法写法，只是多了一层不必要的新 API 面积。

## 我的理解 / 类比
`mount` 是**把整栋楼真实盖起来验收**：看得见邻居、看得见管道串通，最接近住户体验，但任何一处施工问题都会让你误以为是自己这层的问题。
`shallowMount` 是**只盖你负责的那一层，邻居全做成灰色占位块**：干净、快，但「隔壁墙上的字」你是看不见的——这正是插槽不渲染那条坑的庐山真面目。

## 关联
- [[单元测试框架]] —— 上游：Vitest 负责组织与运行用例，VTU 负责挂载
- [[前端测试分类]] —— 本卡属于分层里最细那一层的写法细节
