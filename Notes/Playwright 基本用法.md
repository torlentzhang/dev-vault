---
type: howto
tags: [测试, e2e, Playwright]
created: 2026-11-09
aliases: [Playwright, Playwright 测试, Playwright 用法, e2e 用例写法]
source: https://playwright.dev/docs/intro · https://playwright.dev/docs/writing-tests · https://playwright.dev/docs/locators · https://playwright.dev/docs/test-assertions · https://playwright.dev/docs/test-webserver · https://playwright.dev/docs/mock · https://playwright.dev/docs/network · https://playwright.dev/docs/codegen · https://playwright.dev/docs/api/class-page
---
# Playwright 基本用法

## 适用场景
已有能跑起来的 Web 应用，想用真实浏览器把「打开页面 → 点 → 填 → 断言结果」这条路径自动跑一遍。

本卡只覆盖最常用的骨架：装、配、定位、动作、断言、拦接口、跑、录制。fixture 自定义、并行分片、trace 深度分析、视觉对比、组件测试都不在这里。

## 本质
`@playwright/test` 一个包给齐三件事：**测试运行器 + 浏览器驱动 + 断言库**。所以不需要再配 Jest/Vitest 当 runner。

写一条用例永远是同一套骨架：

```
定位元素 → 执行动作 → 断言状态
```

中间不需要手写等待。

## 步骤

### 1. 安装

新项目（会交互式问四件事）：

```bash
npm init playwright@latest
```

问的是：TypeScript 还是 JavaScript（默认 TS）、用例目录名（默认 `tests`，若 `tests` 已存在则用 `e2e`）、要不要生成 GitHub Actions workflow、装不装浏览器二进制。官方说明这条命令**可以重复跑，不会覆盖已有测试**。

产物：

```
playwright.config.ts
tests/example.spec.ts
package.json
```

已有项目只补依赖：

```bash
npm i -D @playwright/test
npx playwright install --with-deps
```

`--with-deps` 会一并装系统依赖（Linux CI 上必需）。

### 2. 配置

最小可用 `playwright.config.ts`：

```ts
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:5173',
    trace: 'on-first-retry',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
  ],
  webServer: {
    command: 'npm run dev -- --port 5173',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
  },
})
```

三个必知项：

| 配置 | 作用 |
| --- | --- |
| `baseURL` | 让用例里写 `page.goto('/')` 即可，不用每次写全地址 |
| `webServer` | 跑测试前自动拉起 dev server，跑完关掉 |
| `projects` | 一份 spec 跑多套环境（不同浏览器 / 不同设备） |

默认**三个引擎全跑**（Chromium / Firefox / WebKit），所以配置里写一个 `chromium` 项目就是把范围收窄了。

### 3. 写第一条用例

```ts
import { test, expect } from '@playwright/test'

test('首页能打开并渲染', async ({ page }) => {
  await page.goto('/')
  await expect(page).toHaveTitle(/Vite/)
  await expect(page.locator('#app')).toBeVisible()
})
```

`{ page }` 不用自己创建，是运行器注入的**内置 fixture**。

### 4. 定位元素

按官方推荐优先级选，**越靠前越稳**：

| 定位器 | 找什么 | 适合 |
| --- | --- | --- |
| `getByRole()` | ARIA 角色 + 可访问名 | 按钮 / 链接 / 标题 / 复选框，**首选** |
| `getByLabel()` | 表单控件的 label | 表单字段 |
| `getByPlaceholder()` | placeholder 文本 | 没有 label 的输入框 |
| `getByText()` | 文本内容 | `div` / `span` / `p` 这类非交互元素 |
| `getByAltText()` | 图片 alt | `img` / `area` |
| `getByTitle()` | title 属性 | — |
| `getByTestId()` | `data-testid` | 上面都不行时的兜底 |

```ts
await page.getByRole('button', { name: 'Sign in' }).click()
await page.getByLabel('Password').fill('secret')
await expect(page.getByText('Welcome, John')).toBeVisible()
```

两个容易忽略的点：

- **locator 是惰性的**，不是「抓到手的元素快照」。所以组件重渲染后不用重新取。
- **`getByText` 永远归一化空白**，即使 `exact: true`——多个空格会被合成一个、换行变成空格、首尾空白被丢掉。

### 5. 动作

常用九个：

```ts
locator.click()
locator.fill('文本')
locator.check() / locator.uncheck()
locator.hover()
locator.focus()
locator.press('Enter')
locator.selectOption('value')
locator.setInputFiles('path')
```

动作前会自动等元素**可交互**（可见、稳定、不被遮挡、可接收事件），等不到就超时报错，不会静默跳过。

### 6. 断言（最关键的一条规则）

有两类，**行为完全不同**：

| 类型 | 例子 | 自动重试 | 要不要 `await` |
| --- | --- | --- | --- |
| Web 断言 | `await expect(locator).toBeVisible()` | **是**，默认 5 秒 | **必须** |
| 通用断言 | `expect(value).toBe(1)` | 否，立即执行 | **不要** |


常用 Web 断言：

```ts
await expect(locator).toBeVisible()
await expect(locator).toContainText('abc')
await expect(locator).toHaveText('abc')
await expect(locator).toHaveCount(3)
await expect(locator).toHaveAttribute('href', '/x')
await expect(locator).toHaveValue('abc')
await expect(locator).toBeEnabled() / toBeChecked()
await expect(page).toHaveTitle(/Vite/)
await expect(page).toHaveURL('/login')
```

「等某元素出现」的正确写法就是断言本身，**不是** `waitForTimeout`：

```ts
// 对：断言的语义就是「最终会」
await expect(page.getByText('加载完成')).toBeVisible()

// 错：又慢又飘
await page.waitForTimeout(3000)
```

### 7. 组织：隔离与钩子

**每个 test 拿到全新的 BrowserContext**，相当于一个全新浏览器配置文件。 所以 cookie / localStorage 不会跨用例串。

```ts
test.describe('首页', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/')
  })

  test('标题正确', async ({ page }) => {
    await expect(page).toHaveTitle(/Vite/)
  })
})
```

| 钩子 | 跑几次 |
| --- | --- |
| `test.beforeEach` / `afterEach` | 每条用例一次 |
| `test.beforeAll` / `afterAll` | 每个 worker 一次 |

### 8. 拦截接口

四个动作，覆盖绝大多数需求：

```ts
// ① 直接返回假数据，请求不发出去
await page.route('**/api/v1/home', route =>
  route.fulfill({ json: { code: 200, data: {} } })
)

// ② 模拟接口挂掉
await page.route('**/api/v1/home', route => route.abort())

// ③ 真发请求，但改返回值（官方叫 modify API responses）
await page.route('**/api/v1/home', async route => {
  const response = await route.fetch()
  const json = await response.json()
  json.data.projects.push({ id: '99', title: '新增的' })
  await route.fulfill({ response, json })
})

// ④ 只改请求，不动响应（比如删掉某个 header）
await page.route('**/*', route => {
  const headers = route.request().headers()
  delete headers['x-secret']
  route.continue({ headers })
})
```

两条规则要记住：

- **后注册的覆盖先注册的**：*If a request matches multiple registered routes, the most recently registered route takes precedence.*
- **`page.route` 优先于 `context.route`**：两个都命中时用 page 上那个。

还有个不用写代码的选项：HAR 回放。

```ts
await page.routeFromHAR('./hars/home.har', { update: false })
```

### 9. 跑起来

```bash
npx playwright test                    # 全跑，默认无头
npx playwright test --headed           # 想看窗口
npx playwright test --ui               # UI 模式：watch + 时间旅行调试
npx playwright test e2e/home.spec.ts   # 只跑一个文件
npx playwright test --project=chromium # 只跑一个 project
npx playwright show-report             # 打开 HTML 报告
```

HTML 报告**只在有失败时自动弹**，全绿时要手动 `show-report`。

### 10. 不会写就先录

```bash
npx playwright codegen https://example.com
```

会开两个窗口：浏览器 + Inspector。你在浏览器里点，它在 Inspector 里生成代码，还能点图标生成断言（可见 / 文本 / 值）。生成的 locator 按 role → text → testid 排序，多元素命中时它会自动收紧。

带登录态录制：

```bash
npx playwright codegen --save-storage=auth.json https://example.com   # 录完存状态
npx playwright codegen --load-storage=auth.json https://example.com   # 下次复用
```

> `auth.json` 里有真实 cookie，必须进 `.gitignore`。

## 注意事项 / 踩坑

- **忘写 `await` 是最难查的错**。Playwright 的 API 全是 Promise，漏了不报错、不生效，用例「通过」了但其实什么都没做。
- **通用断言不要 `await`**。`expect(value).toBe()` 是同步的，多写 `await` 虽然不炸，但会误导阅读者以为它会重试。
- **不要把「等」写成 `waitForTimeout`**。等待已经内建，硬等只会让测试又慢又飘。
- **`getByText` 归一化空白**，所以别指望它校验空格和换行。
- **`webServer.url` 的判定很宽松**：官方写明返回 *2xx, 3xx, 400, 401, 402, or 403* 都算「服务器起来了」。所以接口全 500 也可能被判为已启动。
- **`reuseExistingServer` 不配会冲突**：本地已经有 dev server 在跑时，不设 `!process.env.CI` 会直接抛错。反过来 CI 上必须设为 `false`，否则可能复用到脏服务。
- **版本要一起升**：升级 `@playwright/test` 后要跟着 `npx playwright install`，否则二进制和库版本对不上。检查版本用 `npx playwright --version`。
- **默认跑三个引擎**：不写 `projects` 就是 Chromium + Firefox + WebKit 全跑，本地慢三倍；只想跑 Chrome 要显式收窄。

## 我的理解 / 类比
Playwright 像**一个极有耐心的助手**：你只描述「点这个按钮」「这行字应该出现」，它自己反复试，直到成功或超时才来告诉你失败。你不需要盯着秒表喊「再等两秒」。

而 `await` 就是**你下达指令时必须说完的那句话**——话说一半（漏 await），助手只听见你张了张嘴，什么也没干，然后安静地站着，用例还显示通过。

## 关联
- [[端到端测试工具]] —— 上游选型：先决定用 Playwright 还是 Cypress，本卡是选定之后的写法
- [[前端测试分类]] —— 本卡属于分层里最粗那一层（e2e）的落地
- [[组件测试的输入与输出]] —— 同一条原则：只断言用户可观察的东西，所以 locator 优先按 role / text 而不是内部类名
- [[vite-plugin-mock]] —— 都是「造接口数据」，但机制不同：mock 插件在 dev server 侧拦，`page.route` 在浏览器侧拦，e2e 里后者更可控
