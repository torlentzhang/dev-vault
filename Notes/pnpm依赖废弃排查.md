---
type: howto
tags: [pnpm, 依赖管理, 踩坑]
created: 2026-09-30
---
# pnpm 排查与清理废弃（deprecated）依赖

## 适用场景
项目里某些依赖"废弃了"，但不确定是真的被作者弃用（deprecated）还是只是版本过时（outdated）；
或想清理传递依赖里被标记弃用的包。

## 步骤
1. 先分清两种"旧"：`pnpm outdated` 看的是**版本落后**（outdated），不包含作者弃用；
   `deprecated` 是作者在 registry 主动标记的"别用了"。
2. 抓真正的 deprecated 包：
   - 跑 `pnpm install`，留意日志里的 `deprecated` / `WARN`（传递依赖的废弃可能不打出，需完整 install 而非 `--frozen-lockfile`）。
   - 或遍历 `pnpm-lock.yaml` 整棵依赖树，逐个查 registry 的 `deprecation` 字段（脚本化最稳，能揪出所有被标记的子依赖）。
3. 追溯来源：`pnpm why <pkg>` 看是谁拖进来的（通常是某条传递链）。
4. 处理：
   - **优先升级直接依赖**：新版父包常已换掉老的子依赖链，废弃包会"搭便车"消失（如升级 `eslint` / `@vue/test-utils`）。
   - **子依赖卡在无人维护的中间层**：用 `pnpm.overrides` 从顶层强制替换（见 [[pnpm配置该写在哪]]）。
     全局写法 `"ini": "^1.3.8"` 对整个依赖树生效；精准写法 `"@vue/test-utils>js-beautify>config-chain>ini": "^1.3.8"` 只覆盖某条链。

## 注意事项 / 踩坑
- `pnpm audit` 在国内淘宝镜像（`npmmirror.com`）会报错——镜像没实现 audit 接口，不是项目问题；换查 lockfile + registry 字段。
- **overrides 不能大版本乱跨**：子依赖可能是 CJS（`require`），而新版是 ESM-only（如 `ini` 5/7），硬升会 `ERR_REQUIRE_ESM` 直接崩；要选 CJS 安全修复版（如 `ini@^1.3.8`）。
- 真正的 deprecated 一定是 registry 标记，别把 outdated（仅是新版）当废弃去处理。

## 关联
- [[pnpm配置该写在哪]] —— overrides 的正确位置（pnpm-workspace.yaml）
- [[pnpm处理peer依赖警告]] —— deprecated 与 peer unmet 是两码事
- [[npm依赖更新方法]] / [[npm-check]] —— npm 侧的同类操作
