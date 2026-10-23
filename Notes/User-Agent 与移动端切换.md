---
type: concept
tags: [web, http, user-agent, mobile, client-hints]
created: 2026-10-11
aliases: [User-Agent, UA, UA 检测, 移动端切换, UA-CH, User-Agent Client Hints]
source:
  - https://wicg.github.io/ua-client-hints/
  - https://www.chromium.org/updates/ua-reduction/
  - https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-CH-UA-Mobile
---

# User-Agent 与移动端切换

## 一句话定义
User-Agent（UA）是浏览器在每个 HTTP 请求里自带的身份声明字符串；「用 UA 做移动端切换」就是解析它判断当前是否为移动设备，从而跳转移动版路由或切换布局。

## 用于解决什么问题
服务端不知道请求来自手机还是桌面，只能统一下发最保守的页面，或者做个「切换到移动版」的链接让用户自己点。

## 要点
- **两种检测位置**：客户端 `navigator.userAgent`；服务端读请求的 `User-Agent` 头。
- **典型移动标记**：`Mobi`、`Android`、`iPhone`、`iPad`、`iPod`、`Windows Phone`，常用正则 `/Mobi|Android|iPhone/i`。
- **Vue 实践**：在全局前置守卫里按 UA 重定向（见 [[导航守卫]]）：
  ```ts
  router.beforeEach((to) => {
    const isMobile = /Mobi|Android|iPhone/i.test(navigator.userAgent)
    if (isMobile && !to.path.startsWith('/m/')) return '/m' + to.path
  })
  ```
- **陷阱 · iPadOS 13+ 伪装**：iPad 上 Safari 默认 UA 与桌面 macOS Safari 完全相同，纯 UA 检测会误判为桌面。补救核心是**触摸能力**：`/Macintosh/.test(ua) && navigator.maxTouchPoints > 0` 即 iPad；屏幕宽度只能当辅助（无法区分大屏手机与小平板）。Android 平板 UA 通常也无 `Mobile` 字样，同样靠触摸 + 尺寸综合判断。
- **陷阱 · UA 可伪造**：手机浏览器自带「请求桌面版」，DevTools / 扩展也能改；UA 只是「浏览器声称自己是谁」，永远不要拿它做安全或鉴权判断。
- **陷阱 · Chrome UA 缩减（2023 起）**：Chrome 110+ 移动 UA 的设备型号与 Android 版本被抹掉（统一成 `Android 10; K`），版本号冻结为 `主版本.0.0.0`；靠 UA 字符串精确识别设备已不可靠。
- **陷阱 · 缓存串版本**：服务端按 UA 分流时，CDN 可能把桌面版缓存给手机用户，需 `Vary: User-Agent`（按 Client Hints 分流则 `Vary: Sec-CH-UA-Mobile`）。
- **现代替代 · User-Agent Client Hints（UA-CH）**：
  - 请求头 `Sec-CH-UA-Mobile: ?1`（`?1` = 移动，`?0` = 非移动），低熵 hint，默认随请求发送；`Sec-CH-UA`、`Sec-CH-UA-Platform` 同理。
  - JS API：`navigator.userAgentData.mobile` / `.platform` / `.brands`（仅安全上下文，HTTPS）。
  - **仅 Chromium 系支持**（Chrome / Edge / Opera / Samsung），Safari、Firefox、整个 iOS 均不支持 → 必须保留 UA 正则兜底。
- **最佳实践**：布局适配优先响应式（CSS 媒体查询），不用 UA 切布局；UA 只用于必须知道浏览器身份的场景（兼容性降级、统计分析）；真要判断移动：`navigator.userAgentData?.mobile ?? /Mobi|Android|iPhone/i.test(ua)`，再辅以触摸 / 屏幕宽度。

## 我的理解 / 类比
UA 像快递单上的「寄件人自述」——浏览器自己填的，可以写错，也可以故意写假；Client Hints 则像按需出示的身份证——服务器明确要哪项才给哪项，够用又少泄露。

## 关联
- [[导航守卫]] —— 移动端跳转常在前置守卫里按 UA 重定向
