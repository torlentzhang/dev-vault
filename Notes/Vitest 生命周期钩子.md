---
type: concept
tags: [测试, 单元测试, Vitest, 钩子]
created: 2026-11-09
aliases: [Vitest 生命周期钩子, beforeEach, afterEach, beforeAll, afterAll, aroundEach, aroundAll, onTestFinished, onTestFailed, 清理函数]
source: https://vitest.dev/api/hooks · https://vitest.dev/guide/lifecycle · https://vitest.dev/guide/learn/setup-teardown · https://vitest.dev/config/sequence · https://vitest.dev/config/hooktimeout · https://vitest.dev/config/globalsetup · https://vitest.dev/guide/migration/jest
---
# Vitest 生命周期钩子

## 一句话定义
钩子是在测试之外**自动执行准备与清理**的函数，共 8 个分三类：普通生命周期（`beforeAll` / `afterAll` / `beforeEach` / `afterEach`）、包裹型（`aroundEach` / `aroundAll`）、Test Hooks（`onTestFinished` / `onTestFailed`）。

> 没有 `setUp` / `tearDown` 测试钩子。`setup` / `teardown` 只出现在 `globalSetup`（全局设置文件），不是测试钩子。

## 用于解决什么问题
每个测试都要连数据库、建 mock、清缓存——不抽出来就得在几十个 `test` 里重复；抽出来之后，这些准备的**收尾**又容易漏。钩子同时解决「重复」与「漏收尾」两件事。

## 要点

### 三类共 8 个
| 类别 | 钩子 | 粒度 |
| --- | --- | --- |
| 普通生命周期 | `beforeAll` / `afterAll` | 整个 suite 一次 |
| | `beforeEach` / `afterEach` | 每个测试一次 |
| 包裹型 | `aroundEach` / `aroundAll` | 把测试 / suite 包在某个上下文中执行 |
| Test Hooks | `onTestFinished` / `onTestFailed` | 测试体内注册，结束后触发 |

### 执行顺序
官方原文：*Test hooks are called in a stack order ("after" hooks are reversed) by default*

```
aroundAll → beforeAll
  └─ aroundEach → beforeEach → 测试 → afterEach
              → beforeEach 返回的清理函数
              → onTestFinished（永远反序）
              → 失败时 onTestFailed
afterAll → beforeAll 返回的清理函数
```

嵌套 `describe`：**外层 before 先跑，after 反序**（内层先收尾，外层后收尾）。

顺序可配置：`sequence.hooks` 默认 `'stack'`，可选 `'list'`（对齐 Jest）、`'parallel'`。**该配置不影响 `onTestFinished`，它永远反序。**

### 作用域
官方原文：*They apply to the current context: the file if they are used at the top-level or the current suite if they are inside a `describe` block.*

- `describe` 内 → 只作用于该块内的测试
- 文件顶层 → 作用于文件内所有测试（顶层测试被收进一个隐式 suite）

钩子超时默认 10 秒（浏览器模式 30 秒），由 `hookTimeout` 配置。

### ⭐ 钩子的返回值 = 清理函数
`beforeEach` / `beforeAll` 可以**返回一个函数**，Vitest 会把它收集起来，在测试（或 suite）结束后自动调用。官方原文：*It's similar to `afterEach`. The only difference is that it's executed after all other `afterEach` hooks.*

```js
beforeEach(async () => {
  const server = await startServer({ port: 3000 })
  const client = await connect(server)

  return async () => {
    await client.close()   // ← 闭包里直接拿到局部变量
    await server.close()
  }
})
```

**主要价值是闭包**：准备与清理写在同一处，共享同一批局部变量，不必把它们提升成外层变量，也不必在 `afterEach` 里重新查找。

`beforeAll` 同理，其清理函数在所有 `afterAll` 之后执行。

### `onTestFinished` 解决的是「断言失败就漏收尾」
写在测试体内的清理会在最后自然执行，但**断言一挂，测试末尾的代码就执行不到了**。官方原话：*if you try to restore the mock at the end of the test, it won't be restored if one of the assertions fails - using `onTestFinished` ensures the code always runs*

```js
test('...', () => {
  const spy = vi.spyOn(api, 'fetch')
  onTestFinished(() => spy.mockRestore())   // 断言失败也会执行
})
```

| | `afterEach` | `onTestFinished` / `onTestFailed` |
| --- | --- | --- |
| 注册位置 | 顶层或 `describe` 内 | **测试体内**，在体外调用会抛错 |
| 时机 | 测试结束后 | 明确排在 `afterEach` **之后** |
| 顺序 | 受 `sequence.hooks` 影响 | `onTestFinished` 永远反序，不受该配置影响 |
| 并发测试 | 全局钩子难以归属 | 并发时必须用测试上下文里的版本 |
| 失败场景 | 始终执行 | `onTestFailed` 仅在测试失败后触发（调试用） |

### `aroundEach` / `aroundAll`：把测试包进上下文
回调里拿到 `runTest` / `runSuite`，**必须调用**，测试才会真正执行：

```js
aroundEach(async (runTest) => {
  await db.transaction(runTest)   // 事务包裹整个测试，结束自动回滚
})
```

适用场景（官方列举）：AsyncLocalStorage 上下文、tracing span、数据库事务。
官方明确：**如果只是要「测试前后各跑一次」，用 `beforeEach` + 返回清理函数即可，不必上 `aroundEach`。**

### 五个坑
1. **隐式返回会被当成清理函数**
   Jest 没有这个约定，从 Jest 迁过来常踩：
   ```js
   // ❌ 箭头函数隐式返回，返回值被当作 teardown
   beforeEach(() => setActivePinia(createPinia()))
   // ✅
   beforeEach(() => { setActivePinia(createPinia()) })
   ```
   官方迁移页原话：*if they return something other than `undefined` or `null`*，需改写。

2. **`aroundEach` 不调 `runTest()` → 测试直接失败**；`aroundAll` 不调 `runSuite()` → 钩子报错且整个 suite 的测试被跳过（官方唯一明写「钩子失败后果」之处）。

3. **并发测试下不要用全局 `onTestFinished` / `onTestFailed`**：*Vitest doesn't track concurrent tests in global hooks*，要用测试上下文里的版本。

4. **类型检查模式下钩子不执行**（`vitest --typecheck` 时）。

5. **v5 默认 `clearMocks: true`**：在 `beforeAll` 或 setup 文件里制造的 mock 调用历史，会在断言它的测试开始前被清空——这类「在测试体外记录调用」的测试受影响最大。

## 我的理解 / 类比
`beforeAll` / `afterAll` 是**开馆前布置、闭馆后清场**：一天各一次，最省事，但客人（测试）之间会看到彼此留下的痕迹。
`beforeEach` / `afterEach` 是**每位客人的独立包厢**：进场前重置，离场后打扫，互不干扰，代价是每次都得来一遍。
返回的清理函数像**交房时一起领到的一把钥匙**——把它和「布置」写在一起，收尾时自然知道该关谁的灯。
`aroundEach` 则不同，它**把整个测试搬进另一个容器里运行**（如数据库事务），测试本身在里面跑，容器负责善后。

## 关联
- [[单元测试框架]] —— 上游：Vitest 作为框架的整体定位
- [[组件挂载与 stub]] —— 组件测试里的清理（如 `wrapper.unmount()`）常放在这些钩子里
