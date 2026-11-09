---
type: concept
tags: [测试, 单元测试, Vue, VTU, v-model]
created: 2026-11-09
aliases: [测试 v-model, v-model 测试, setValue, onUpdate:modelValue, 测试双向绑定]
source: https://test-utils.vuejs.org/guide/advanced/v-model · https://test-utils.vuejs.org/guide/essentials/forms · https://test-utils.vuejs.org/guide/advanced/async-suspense · https://test-utils.vuejs.org/migration/ · https://vuejs.org/guide/components/v-model.html
---
# 测试 v-model

## 一句话定义
v-model 是「一个 prop 向下 + 一个事件向上」的组合，所以测它要**两端都接上**：用 `setValue` 模拟用户输入（造上游），再自己挂 `onUpdate:modelValue` 监听器把值回写（补下游），最后断言 `wrapper.props('modelValue')` 确实变了。

## 用于解决什么问题
只断言 `emitted('update:modelValue')` 只能证明「子组件喊了一声」，证明不了「双向绑定闭环成立」——父组件那半段回写没人验。而 v-model 出问题，恰恰常出在回写没接上、或值变了但 DOM 没更新。

## 要点

### 原生表单元素：统一用 `setValue`，没有 `setChecked`
VTU v2 里 text / textarea / select / checkbox / radio **全部用 `setValue`**：

```js
await wrapper.find('input[type=email]').setValue('name@mail.com')
await wrapper.find('textarea').setValue('Lorem ipsum')
await wrapper.find('select').setValue('moscow')
await wrapper.find('input[type=checkbox]').setValue()        // 不传参 = 勾上
await wrapper.find('input[type=radio][value=monthly]').setValue()
```

官方原话：*setValue is a very versatile method. It can work with all types of form elements.*
另有 TIP：*If you don't pass a parameter to `setValue` for `OPTION`, `CHECKBOX` or `RADIO` inputs, they will set as `checked`.*

⚠️ **`setChecked()` / `setSelected()` 是 v1 的 API**，官方迁移表写的是 *removed. Now part of `setValue`*。网上大量 `setChecked` 示例照抄到 v2 会直接失败。

提交、点击这类操作用 `trigger`：`await wrapper.find('form').trigger('submit.prevent')`。

### 自定义组件的 v-model：官方写法是自己接上回写
官方专门有 [Testing v-model](https://test-utils.vuejs.org/guide/advanced/v-model) 页，核心示例：

```js
test('modelValue should be updated', async () => {
  const wrapper = mount(Editor, {
    props: {
      modelValue: 'initialText',
      'onUpdate:modelValue': async e => {
        // 若 modelValue 在 setup 中更新，需要这个 await nextTick
        await nextTick()
        await wrapper.setProps({ modelValue: e })
      }
    }
  })

  await wrapper.find('input').setValue('test')
  expect(wrapper.props('modelValue')).toBe('test')
})
```

这段做了三件事：

1. 传 `modelValue` 模拟父组件给的初值
2. 挂 `onUpdate:modelValue` 监听器，**扮演父组件**收到事件后自己改值
3. `setValue` 模拟用户输入，断言 `wrapper.props('modelValue')` 已更新

**关键**：官方这一页通篇没有 `expect(wrapper.emitted('update:modelValue'))` 的写法。因为那样只验了「子组件发出去了」，没验「闭环成立」。这个模式验的是完整链路。

> `emitted('update:modelValue')` 仍然可用，只是**不是**官方在 v-model 页推荐的写法；两者不冲突，需要单独验「事件确实发了」时可以加。

### 必须 await——但只在要读 DOM 时
官方异步页原话：*Methods that cause the DOM to update, such as `trigger` and `setValue` return `nextTick`, so you can just `await` those directly.*

```js
await wrapper.find('input').setValue('x')   // 返回 nextTick
expect(wrapper.find('.preview').text()).toBe('x')
```

**细节**：官方自己的示例证明，**只断言 `emitted()`、不读 DOM 时可以不 await**（event-handling 页连调两次 `trigger('click')` 不 await 就直接断言 emitted，仍然通过）。await 的必要条件是「后面要读 DOM」。

### 多个 v-model：每个各挂一条监听
```js
const wrapper = mount(MoneyEditor, {
  props: {
    modelValue: 'initialText',
    'onUpdate:modelValue': e => wrapper.setProps({ modelValue: e }),
    currency: '$',
    'onUpdate:currency': e => wrapper.setProps({ currency: e })
  }
})
```
官方有独立段落「Multiple `v-model`」，每个具名 model 各接一条。

### 自定义输入组件：也可以直接操作内部原生元素
VTU 对「自定义输入组件」也给了另一种官方写法——直接找到组件内部的原生 input：

```js
const wrapper = mount(CustomInput)
await wrapper.find('.text-input input').setValue('text')
```

**取舍**：这种写法更短，但它依赖内部 DOM 结构（`.text-input`），属于对实现细节的耦合；接 `onUpdate:modelValue` 的写法验的是对外契约，重构时更稳。公共组件优先后者。

### `defineModel` 组件怎么测
**官方文档没有写**——VTU 的 v-model 页通篇未出现 `defineModel` 或 `<script setup>`。

能依据的是 Vue 官方对宏展开的说明：*`defineModel` is a convenience macro. The compiler expands it to… a prop named `modelValue`… an event named `update:modelValue`…*

既然展开后就是 props + 事件那套，**上面的回写模式在原理上适用**。但这一步是**推断，不是官方原话**。

### 修饰符：官方未提及
`.lazy` / `.number` / `.trim` 在 VTU 的 v-model、forms、async、migration、faq 等页面里**没有任何测试写法**，本卡不给做法。

⚠️ 别混淆：VTU 里的 `submit.prevent`、`keydown.meta.c.exact.prevent` 是**事件修饰符**（字符串传给 `trigger`），与 v-model 修饰符无关。

## 我的理解 / 类比
测 v-model 像**验证一台对讲机的通话闭环**：按一下话筒（`setValue`）是「我说了」，看指示灯亮（`emitted`）只证明「信号发出去了」——但对方到底有没有收到并回话，得自己把接收端接上（`onUpdate:modelValue`）。官方那句 `expect(wrapper.props('modelValue')).toBe('test')` 验的正是「对方回话后，我这边的显示也变了」。

## 关联
- [[组件测试的输入与输出]] —— 上位卡：v-model 本质就是 props + emits 的组合，本卡是它在双向场景下的具体写法
- [[Vue 组件 v-model]] —— 被测概念本体：`defineModel` 的展开与用法
- [[组件挂载与 stub]] —— 前置：用 mount 还是 shallow 决定能操作到多深
