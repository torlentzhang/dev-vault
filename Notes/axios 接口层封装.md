---
type: concept
tags: [axios, HTTP, 前端工程化, 代码分层]
created: 2026-11-04
aliases: [接口层封装, API 层封装, service 层封装, axios 封装]
source: axios 官方文档 https://axios.rest/pages/advanced/create-an-instance · https://axios.rest/pages/advanced/interceptors · https://axios.rest/pages/advanced/type-script · https://axios.rest/pages/advanced/cancellation · https://axios.rest/pages/advanced/error-handling
---
# axios 接口层封装

## 一句话定义
接口层封装是把**一份预配置的 axios 实例**和**一组按业务归类的请求函数**收进模块（`userService.login()`），让业务代码不必在每处关心 baseURL、鉴权头、错误处理和响应解包。

## 用于解决什么问题
不封装时，每个页面自己 `axios.get`、自己往 header 里塞 token、自己判断业务 `code`、自己写 loading 和错误提示。改一次鉴权方式或后端域名，要全库搜一遍请求代码；同一个接口在两处调用，错误处理逻辑还对不上。

## 要点

### 它其实捆了三件独立的事
| 层 | 职责 | 谁提供这个能力 |
| --- | --- | --- |
| 实例配置 | baseURL、timeout、默认 header | `axios.create()` |
| 横切逻辑 | 统一鉴权、错误处理、token 刷新、loading | 拦截器 |
| 模块归类 | `userService.xxx()` 这种按业务聚拢的调用形态 | 模块系统本身，不是 class |

判断时先分清卡在哪一层：前两层是 axios 的真实能力，第三层只是命名空间。**把「实例」当资源（少而稳定），把「模块」当目录（随业务长）。**

### 官方推荐的骨架
大于单文件的应用，用 `axios.create()` 得到预配置实例，而不是直接调用全局 `axios`。实例自带自己的 defaults，请求级配置再覆盖它。挂在实例上的拦截器**只作用于该实例**，因此鉴权头和错误处理应当集中注册在实例上，而不是散落在业务调用处。

```ts
// utils/axios.ts —— 实例与拦截器只在这里出现一次
import axios from 'axios'

export const http = axios.create({ baseURL: '/api', timeout: 10000 })

http.interceptors.request.use((config) => {
  const token = localStorage.getItem('token')
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})

http.interceptors.response.use(
  (res) => res,
  (err) => Promise.reject(err)
)
```

TypeScript 下用泛型标注响应体，`res.data` 即被推断为目标类型；请求拦截器里的参数类型是 `InternalAxiosRequestConfig`，不是 `AxiosRequestConfig`。取消请求用 `AbortController` + `signal`（`CancelToken` 已废弃），用 `axios.isCancel()` 判别；错误用 `axios.isAxiosError<T>()` 收窄。

### 三处落地位置
| 位置 | 放什么 | 不该放什么 |
| --- | --- | --- |
| `utils/axios.ts` | 实例、拦截器、统一解包 | 任何业务方法 |
| `api/*.ts` | 按业务分组的请求函数 | `axios.create`、拦截器、登录态字段 |
| `stores/*.ts` | 登录态、缓存、业务状态 | 请求细节 |

service 层只 import 实例、不自己造实例，保证配置与拦截器只有一个来源，也不存在「谁的实例」这类歧义。

### 实例层：class 包装 vs 工厂函数

企业模板里常见的一种形态，是用类把实例包起来，再导出一个单例：

```ts
class AxiosService {
  private axiosInstance: AxiosInstance
  constructor(baseURL: string, timeout: number) {
    this.axiosInstance = axios.create({ baseURL, timeout })
  }
  public getInstance() { return this.axiosInstance }
}
const axiosService = new AxiosService('/api', 10_000)
export default axiosService

// 调用侧
axiosService.getInstance().get('/user')
```

它与工厂函数导出实例**功能等价**，两种写法都只有一个实例、配置同样集中：

```ts
export function createHttp(baseURL: string, timeout = 10_000) {
  const instance = axios.create({ baseURL, timeout })
  return instance
}
export default createHttp('/api')
```

| 差异点 | class 包装 | 工厂函数 |
| --- | --- | --- |
| 调用侧 | `service.getInstance().get()` | `http.get()` |
| 配置注入 | 构造函数参数 | 函数参数 |
| 多后端 | `new AxiosService(url2)` 或 `createHttp(url2)` | 同 |
| 实例类型 | 需要额外记一个 `AxiosService` 类型 | 直接用 `AxiosInstance` |
| 单例保证 | 导出的常量只有一个 `new` 时才成立 | 与写法无关 |

两种写法的取舍只在两处：一是 `.getInstance()` 使每个调用点多一次访问，且项目里同时存在包装类型与 `AxiosInstance` 两套名字；二是 class 有一个真实用途——**充当唯一入口点**，可把懒注册的扩展能力（日志、埋点、重试）收在 `getInstance()` 里，而不必改所有调用点。工厂函数同样能做到，只是入口在配置文件而非类方法。

判断某层包装是否有实质职责，用这一条：**能否删掉它、只改 import 就让业务代码照常运行**。能删就是纯壳；删了要改一堆调用点，说明它承担了入口职责。

### class 与对象字面量：调用形态相同
```ts
// class 形态
export class UserService {
  login(data: LoginDTO) { return http.post<LoginResult>('/user/login', data) }
  register(data: RegisterDTO) { return http.post<LoginResult>('/user/register', data) }
  exit() { return http.post('/user/exit') }
}
export const userService = new UserService()

// 对象字面量形态
export const userService = {
  login: (data: LoginDTO) => http.post<LoginResult>('/user/login', data),
  register: (data: RegisterDTO) => http.post<LoginResult>('/user/register', data),
  exit: () => http.post('/user/exit'),
}
```
调用方写法完全一致（`userService.login()`）。差别只在以下几件事上：

| 差异点 | class | 对象字面量 |
| --- | --- | --- |
| 需不需要 `new` | 需要 | 不需要 |
| 方法被解构后还能否调用 | 用 `this` 的写法会丢 `this` | 能 |
| 多份独立实例 | 天然支持 | 需手写工厂 |
| 硬私有字段 `#x` | 支持 | 不支持 |
| 导出的类型 | 类名可直接当类型 | 需 `typeof userService` |

上述情况都不涉及时的典型场景（单后端、单鉴权、无实例状态），class 只是多一层语法包装，不提供额外能力，本身也不是 bug 来源。

### 三个真实的坑
1. **`this` 被解构掉**：class 方法里写 `this.http.xxx()`，一旦 `const { login } = userService` 或把方法当回调传出去，`this` 就是 `undefined`。对象字面量用闭包访问实例，没有这个问题。
2. **往共享实例上重复注册拦截器**：拦截器叠加的条件是「注册目标长期存活 + 注册代码被重复执行」——即对同一个模块级实例或全局 `axios` 多次调用 `interceptors.use()`，热更新与单测中会累积，表现为请求头出现两个 `Authorization`、错误提示弹两次。相反，`axios.create()` 写在构造函数里时每次 `new` 得到全新实例，各带自己的拦截器，不会叠加。风险在于把 `interceptors.use()` 写在对同一个实例的其他位置，而不是构造函数本身。
3. **service 里存状态**：在 class 里加 `private token` 或 `private cachedUser`，登录态就有了第二份，与 Pinia 里那份不同步，最终表现为「明明登录着却 401」。service 应当无状态。

### class 什么时候是唯一解
需要多份独立实例且每份带可变状态、需要 `extends` 内置类（如 `class ApiError extends Error` 才能保住 `instanceof` 与 `stack`）、需要 `instanceof` 判定、需要 `#` 硬私有、或框架要求（装饰器、Web Components）。接口层这几种情况通常一个都不占。

## 我的理解 / 类比
实例像**一栋楼的总配电箱**：全楼只装一个，所有线路都从它取电，跳闸和维护只看这一处。service 像**楼里的功能房间**——「登录间」「注册间」，你进去按按钮就行，不用自己接线。而 class 还是对象字面量，只是房间的门牌是刻在铜牌上还是贴张纸：进门方式一样，屋里也一样的灯。

## 关联
- [[Mockjs]] —— 请求发出后由谁在客户端拦截，与本卡「请求怎么发出」是相对的另一侧
- [[vite-plugin-mock]] —— 开发期在 dev server 侧拦截同一批接口，不侵入这里的 service 写法
- [[Pinia 选项式与组合式]] —— service 不存状态时，登录态与缓存归它管
