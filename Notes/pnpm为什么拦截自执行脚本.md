---
type: concept
tags: [pnpm, 包管理器, 依赖安全, 生命周期脚本]
source: https://pnpm.io/cli/approve-builds · https://pnpm.io/10.x/settings#onlybuiltdependencies
created: 2026-08-14
---
# pnpm 为什么拦截依赖的自执行脚本

## 一句话定义
pnpm 出于安全与确定性，默认**不运行**依赖包的生命周期脚本（如 postinstall / build / prepare），需要你显式 `pnpm approve-builds` 批准，否则带原生编译步骤的包会缺产物而报错。

## 要点
- npm / yarn 默认会执行依赖的 install 脚本，pnpm 默认拦截——这是 pnpm 更安全但更易踩坑的地方
- 受影响的是"安装期需要自己跑脚本"的包，典型如带原生二进制 / 需要编译的：cypress、esbuild、protobuf、better-sqlite3 等
- `pnpm approve-builds` 交互式勾选允许哪些包执行 build 脚本，结果写进 package.json 的 `pnpm.onlyBuiltDependencies`
- 也可直接在该字段手写允许的包，或用 `pnpm rebuild <pkg>` 单独补跑被拦的脚本

## 我的理解 / 类比
pnpm 把"依赖能偷偷跑脚本"这扇门默认关上了；npm 是门大开。最容易撞上的是 cypress——第一次用它就因 postinstall 被拦而缺二进制文件报错，批准一次即可。

## 关联
- [[vite+Vue3工程的初始化与启动]] —— 原理-实践：本卡讲拦截机制，该卡是 pnpm install 起一个 Vite 工程的完整落地流程
- [[Corepack]] —— 前置：pnpm 版本经 corepack 提供，拦截行为随版本变化，先确认版本
- [[pnpm处理peer依赖警告]] —— 对比：同属 pnpm 安装期提示，一个是脚本被拦需批准，一个只是 peer 版本不匹配的警告
- [[pnpm add与pnpm install的区别]] —— 装包时想直接放行脚本：`pnpm add --allow-build=<pkg>`
- [[npx]] —— 另一条执行外部包的路径，安全边界不同：npx 靠包名正确性防钓鱼，pnpm 靠脚本拦截
