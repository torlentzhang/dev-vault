---
type: concept
tags: [pnpm, 包管理器, 依赖安全]
created: 2026-08-14
---
# pnpm 为什么拦截 cypress 脚本

## 一句话定义
pnpm 出于安全与确定性，默认**不运行**依赖包的生命周期脚本（如 postinstall / build），需要你显式 `pnpm approve-builds` 批准，否则带原生二进制的包会缺文件而报错。

## 要点
- npm / yarn 默认会执行依赖的 postinstall，pnpm 默认拦截——这是 pnpm 更安全但更易踩坑的地方
- 受影响的多是带原生模块的包：cypress、esbuild、protobuf、better-sqlite3 等
- `pnpm approve-builds` 会交互式让你勾选允许哪些包执行 build 脚本，结果写进 package.json 的 `pnpm.onlyBuiltDependencies`

## 我的理解 / 类比
pnpm 把"依赖能偷偷跑脚本"这扇门默认关上了；npm 是门大开。第一次用带原生二进制的包（尤其 cypress）就会撞上这扇关着的门。

## 关联
- [[vite+Vue3工程的初始化与启动]]
- [[corepack是什么]]
