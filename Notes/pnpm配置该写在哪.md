---
type: concept
tags: [pnpm, npmrc, 配置, 包管理器]
created: 2026-09-14
aliases: [pnpm配置位置, pnpm npmrc]
source: https://pnpm.io/11.x/migration
---

# pnpm 配置该写在哪

## 一句话定义
pnpm v11 起 `.npmrc` 只认认证 / registry 类设置；`save-exact`、`node-linker` 等行为类配置必须写在 `pnpm-workspace.yaml`（项目级）或全局 `config.yaml`，写在 `.npmrc` 里等于没写。

## 要点
- **变更版本**：pnpm v11 正式切割（v10 周期已收紧）。`.npmrc` 从此只读取 auth / registry 设置（如 `_authToken`、`registry`、`@scope:registry`）。
- **写在 `.npmrc` 已无效的行为类设置**：`save-exact`、`save-prefix`、`node-linker`、`hoist-pattern`、`shamefully-hoist` 等所有非认证项。
- **正确位置 ① 全局**：macOS 下在 `~/Library/Preferences/pnpm/config.yaml` 加 `saveExact: true`，所有项目统一生效。
- **正确位置 ② 项目级**：项目根 `pnpm-workspace.yaml` 加 `saveExact: true`（即便单包项目也能有此文件，不写 `packages` 字段就只含 root package）。
- **正确位置 ③ 命令行单次**：`pnpm add pkg --save-exact`。
- **正确位置 ④ 环境变量**：`pnpm_config_save_exact=true pnpm add pkg`，或写进 shell profile 长期生效。
- **同批相关坑**：pnpm 不再读 `npm_config_*` 环境变量，要改名 `pnpm_config_*`（如 `npm_config_registry` → `pnpm_config_registry`）；也不再读 `package.json#pnpm` 字段。

## 我的理解 / 类比
把 `.npmrc` 想成 pnpm 的"钥匙串"——从 v11 起它只管钥匙（认证 / registry），其他所有"房间布局"（行为配置）都搬进了 YAML 文件（`pnpm-workspace.yaml` / `config.yaml`）。你往钥匙串里塞布局图，它压根不看，所以 `.npmrc` 里的 `save-exact=true` 对 pnpm 来说是隐形配置。

## 关联
- [[Corepack]] —— corepack 决定用 pnpm@10 还是 @11，版本直接决定 `.npmrc` 是否还读 `save-exact`
- [[nvm切换node导致pnpm不兼容]] —— 同系列 pnpm 版本 / 配置踩坑；那张卡里也提到 `corepack use pnpm@11`
- [[pnpm add与pnpm install的区别]] —— 这次 `save-exact` 失效正是出在 `pnpm add` 场景
