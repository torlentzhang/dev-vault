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

## 额外区分：deprecated 与 unmet peer 是两码事
- **deprecated（废弃）**：作者主动在 registry 标记"这个包别用了"，是包本身的健康问题。`pnpm outdated` 看不出来（它只显示版本落后），要遍历 lockfile + 查 registry 的 `deprecation` 字段，或用 `pnpm audit`（注意淘宝镜像没实现 audit 接口会报错）。
- **unmet peer（同伴依赖不匹配）**：只是"插件要求的宿主包版本范围，和你实际装的对不上"，包本身好好的，只是潜在集成风险。两者性质不同，别混为一谈。

## 处理原则：能对齐就升，上游滞后就忽略
- **能对齐的（你控制的直接依赖）**：直接升级到互相匹配的版本即可。例：`eslint-plugin-oxlint` 与 `oxlint` 版本需对齐，双双升到 `1.86.0` 后 peer 警告消失。
- **上游滞后的（父包还没发兼容版）**：例 `@vue-macros/devtools` 最新版仍只认 vite 4~7，而项目已用 vite 8；`@vue-macros/volar` 要的 `vue-tsc` 还是精确 `3.0.8`，项目已用 `3.3.9`。这种只是 WARN、不阻塞 dev/build，**暂时忽略即可，千万别为了消警告把主依赖降级**（降级 vite / vue-tsc 反而丢新功能）。等上游发新版自然消。

## 关联
- [[pnpm为什么拦截自执行脚本]]
- [[Corepack]]
- [[pnpm add与pnpm install的区别]] —— 手动补装缺失的 peer 包：`pnpm add <pkg>`
