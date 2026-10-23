---
type: concept
tags: [node, npm, 包管理器, 依赖更新, 依赖管理]
created: 2026-09-15
aliases: [依赖更新, 依赖升级, npm update, ncu, dependabot, renovate]
source:
  - https://docs.npmjs.com/cli/commands/npm-update
  - https://docs.npmjs.com/cli/commands/npm-outdated
  - https://github.com/raineorshine/npm-check-updates
---
# npm 依赖更新方法

## 一句话定义
更新项目 npm 依赖有三条路径——手动自定义、命令行工具、平台 bot——自动化程度递增，决策权从你逐步转移到机器人。

## 用于解决什么问题
升级依赖时不按破坏性风险分档，就只能全手动或全自动：每个包都人肉评估 changelog 太累，放任 bot 直接改依赖树又可能悄悄跨主版本合并，两类做法都会在某个批次上出事。

## 要点
- **路径一 · 手动自定义（你决策）**
  - 直接编辑 `package.json` 里依赖的范围，再 `npm install` 让 lock 跟上；或 `npm install <pkg>@<ver>`、`npm install <pkg>@latest`
  - 适合大版本升级、需要看 changelog 评估破坏性变更时
  - ⚠️ 改完**必须**跑 `npm install` 更新 `package-lock.json`，否则别人装的还是旧依赖树（lock 优先于范围）
- **路径二 · 借助 npm 工具（半自动）**
  - `npm outdated`：只查过期，列出当前/想要/最新版本，不动手
  - `npm update`：**只升到 package.json 范围内允许的最新版**，受 `^` `~` 限制，**不跨主版本、不改范围符号**
  - `npm-check-updates`（简称 `ncu`）：**重写 package.json 范围到当前最新版，可跨主版本**（如 `^1` → `^2`），跑完需 `npm install` 应用
  - `npm-check`（交互式）：列出过期包可勾选更新
  - 临时跑这些工具用 `npx ncu -u` 即可，不占全局（执行手段见 [[npx]]，非本主题必要组成）
- **路径三 · 平台 bot（全自动）**
  - **Dependabot**：GitHub 原生，配 `.github/dependabot.yml`，可针对 CVE 自动提安全 PR
  - **Renovate**：GitLab / GitHub 都能用，更强——支持分组(grouping)、定时(schedule)、自动合并(automerge)、monorepo 感知
  - 企业内部也可能有自研或 Snyk 类 bot
  - 独门价值：自动提 PR + **实时 CVE 安全响应** + 可设合并节奏

## 三类对比

| 维度 | 手动 | npm 工具(update/ncu) | 平台 bot |
|---|---|---|---|
| 决策权 | 你 | 工具（按范围 / 按最新） | bot 自动 |
| 自动化 | 无 | 半自动 | 全自动 |
| 跨主版本 | 你决定 | ncu 可 / update 否 | 可（默认谨慎） |
| 安全 CVE 响应 | 靠你发现 | 靠你跑 | 实时自动 PR |
| 适合 | 大版本谨慎升级 | 本地快速刷新 | 长期维护、安全合规 |

## 常见误区
- **别把 `npm update` 当"升到最新"**：它受 `^` `~` 范围限制，主版本跳不上去；想升主版本用 `ncu`
- **bot 不是"更高级的 npm update"**：它的独门价值是安全响应与持续维护，团队项目 / CVE 高频场景里手工和本地工具都补不了这个实时性

## 我的理解 / 类比
三条路像"升级依赖的三档服务"：手动是 DIY 自己拧螺丝；npm 工具是电动螺丝刀（快，但只能按你画的范围线走）；平台 bot 是外包维修队（自动上门，还帮你盯着安全召回通知）。

## 关联
- [[npm语义化版本号]] —— 升级依赖本质就是改范围符号；`^` 不跨主版本直接决定 `npm update` 与 `ncu` 的行为差异，不懂 SemVer 就懂不了"为什么 update 升不上去"
- [[npm-check]] —— 路径二里"交互式挑升"的工具本体；本文只收定位与对比，具体 flags 见该卡
