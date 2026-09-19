---
type: howto
tags: [node, pnpm, 依赖管理, 踩坑]
created: 2026-08-14
---
# pnpm 安装时 peer dependencies 警告的排查

## 适用场景
pnpm install 后出现 `Issues with peer dependencies found. Run "pnpm peers check" to list them.`，需要理解它、判断要不要处理、怎么处理。

## 步骤
1. 跑 `pnpm peers check` 列出具体哪些包的 peer 依赖不匹配
2. 看输出里每行的 `→ peer xxx@x.y but z.w is installed`，定位冲突的包
3. 按情况处理（见下方注意事项）

## 注意事项 / 踩坑
- **peer dependency 是什么**：插件包声明"我需要宿主（你的项目）来提供某个库"，自己不装。典型如 Vue 生态插件要求项目里有 vue。
- 这不是报错，是 pnpm 的提醒：pnpm 默认不自动安装 peer 依赖、也不会擅自改版本，只检查是否匹配；npm/yarn 默认会帮你装一个兼容版本，所以少报警告，但可能塞了你不想要的版本。
- pnpm v8+ 默认 `auto-install-peers=true`，会补装缺失的 peer 依赖；但**版本不匹配**仍只警告、不自动改。
- 大多数 peer 警告不影响使用（版本差一点、实际兼容）可忽略。
- 真要处理时：升级/降级那个冲突的插件包，或手动 `pnpm add <缺失的peer包>`，或在 `.npmrc` 配 `auto-install-peers=true` / `strict-peer-dependencies=false`。

## 关联
- [[pnpm为什么拦截自执行脚本]]
- [[Corepack]]
- [[pnpm add与pnpm install的区别]] —— 手动补装缺失的 peer 包：`pnpm add <pkg>`
