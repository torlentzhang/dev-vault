---
type: moc
tags: [moc, engineering]
created: 2026-08-23
---
# Engineering · MOC

> 领域地图：如何开发、构建、管理和发布代码（工具链与工程规范）的入口。

## 包管理（pnpm / npm / corepack）
- [[MOC-Engineering-PackageManagement]]
（版本语义 / 安装与更新 / 解析与安全 / 配置与环境 …）

## 版本管理（nvm / node）
（nvm 切换、node 版本管理 …）
- [[nvm]]
- [[nvm切换node导致pnpm不兼容]]

## 本地规范
（ESLint / Prettier / Husky …）

## 测试
- [[MOC-Engineering-Testing]]
（测试分类与维度 / 单元测试 / 端到端测试 …）

## 构建工具（Vite / Webpack）
- [[MOC-Engineering-BuildPlugins]]
- [[Vite 环境变量]] `.env` 加载规则、mode 与 NODE_ENV、构建期静态替换
（dev server、打包优化、HMR、构建插件 …）

## 数据模拟（Mock）
- [[Mockjs]]
（前端联调假数据、XHR 拦截、接口模拟 …）
