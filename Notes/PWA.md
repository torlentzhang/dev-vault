---
type: concept
tags: [web, pwa]
created: 2026-10-02
aliases: [渐进式 Web 应用, Progressive Web App]
source: https://developer.mozilla.org/zh-CN/docs/Web/Progressive_web_apps
---
# PWA

## 一句话定义
PWA（Progressive Web App，渐进式 Web 应用）不是单一技术，而是用 Web 技术开发、通过渐进增强让网页获得接近原生 App 体验的一组能力组合：可安装、可离线、可推送。

## 用于解决什么问题
普通网页断网只剩浏览器的错误页，用户桌面没有入口、也收不到推送；想要这些体验，只能再维护一套原生 App。

## 要点
- 本质是「渐进增强」：能力逐层叠加，老浏览器不支持时退化为普通网页，依然可用
- 三块核心拼图：
  - **HTTPS** —— 必要前提：Service Worker 权限较高，浏览器只信任安全上下文
  - **Web App Manifest**（JSON）—— 应用的「身份证」：`name`、`icons`（192px / 512px）、`start_url`、`display: standalone` 等，让浏览器知道如何安装与全屏呈现
  - **Service Worker**（后台脚本）—— 拦截 `fetch` 请求，在「走网络 / 走缓存」间决策，实现离线与秒开；配合 Push API 实现推送通知
- 三个核心能力（经典框架）：**Reliable**（离线可靠）、**Fast**（缓存秒开）、**Engaging**（可安装、全屏、推送）
- 优势：一套代码全平台、免应用商店审查与等待更新、URL 即达、体积比原生 App 小
- 代价：浏览器支持度有差异（iOS Safari 最保守），后台能力与系统 API 仍不如原生 App，安装触发条件随浏览器版本演变
- 术语由 Alex Russell 与 Frances Berriman 于 2015 年提出，核心是「把 Web 的优势与原生 App 的体验结合」

## 我的理解 / 类比
普通网页是浏览器里的一个链接；PWA 是用户在手机上「添加到主屏幕」后，桌面图标一点就开、断网也能用的那个应用。HTTPS 是资格认证，Manifest 是应用的身份证，Service Worker 是守在浏览器和网络之间的快递员——它决定每个请求是走网络还是走缓存。

## 关联
[[vite-plugin-pwa]]