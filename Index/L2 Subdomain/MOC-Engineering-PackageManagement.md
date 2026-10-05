---
type: moc
tags: [moc, engineering, package-manager]
created: 2026-09-17
---
# Package Management · MOC

> 子领域地图：依赖与包管理器的入口（版本语义 / 安装与更新 / 解析与安全 / 配置与环境）。

## 版本语义
（SemVer、依赖范围符号、升级规则 …）
- [[npm语义化版本号]]

## 安装、更新与执行
（add / install 语义、升级与检查、一次性执行 …）
- [[pnpm add与pnpm install的区别]]
- [[npm依赖更新方法]]
- [[依赖升级评估]]
- [[npm-check]]
- [[npx]]

## 解析与安全
（peer 依赖、生命周期脚本、依赖安全 …）
- [[pnpm处理peer依赖警告]]
- [[pnpm为什么拦截自执行脚本]]
- [[pnpm依赖废弃排查]]

## 配置与环境
（.npmrc / pnpm-workspace.yaml、corepack、全局安装落点 …）
- [[pnpm配置该写在哪]]
- [[Corepack]]
- [[npm全局安装路径]]
