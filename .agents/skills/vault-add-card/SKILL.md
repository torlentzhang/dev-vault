---
name: vault-add-card
description: 在 Obsidian vault 中新增一张原子卡片（概念/文献/howto），并完成 MOC 挂链与双向回链。当用户要求"新建笔记/新增卡片/整理一个知识点到 vault"时使用。
---

# 新增原子卡片（vault-add-card）

适用：在 02-Notes 目录新增一张原子卡片。

## 前置
- 只读操作（读模板、读 MOC、搜已有卡片、fetch 查文档）属 L1，可直接做。
- 写盘属 L2：按 AGENTS.md「安全分级」，先向用户说明将创建的文件路径与内容概要，获同意后再写。

## 步骤
0. 校验知识点（先做，用户最在意）：写卡前先校验知识点正确性——以官方文档 / RFC / 权威源码为准，不确定就先 fetch 官方文档核实。若用户说法有误，先明确指出并给出依据，纠正后再继续；不顺着错误写卡。
1. 判断卡片类型，选 05-Templates 对应模板：
   概念卡 → concept-card.md｜文献卡 → literature-card.md｜操作/配方卡 → howto-card.md
2. 确定单一归属 MOC：先按内容定位一级领域，再挂到该领域的**最末级 MOC**（未开二级时即一级 MOC 本身）。
   （pnpm/corepack/nvm→MOC-工程化；CSS 预编译→MOC-前端基础；Vue 等框架→MOC-框架；该领域若已开二级，如 `MOC-框架-Vue`，则挂二级）。
   不要硬塞进多个 MOC；卡片只链最末级，上一级 MOC 不再直接链卡片（二级判据与切口原则见 AGENTS.md「MOC 分层约定」）。
   - 小节膨胀处置：挂进的 `##` 小节卡片过多时，在该小节下用 `###` 分组（如 `### 原理` / `### 配置实战` / `### 进阶`）；**不得再开三级 MOC**；若考虑降低卡片粒度，判据是「能否独立讲清一个概念」，不得为排版硬拆。
3. 命名：直接以概念命名（沿用 vault 风格，如 `corepack是什么`、`Sass与SCSS的语法区别`）。
4. 写 02-Notes/<卡片名>.md：frontmatter 字段以所选模板为准——
   concept / literature 补 type / tags / created 及 aliases / source；
   howto 模板只有 type / tags / created，不要自行添加 aliases / source。
   tags 沿用既有卡片词汇，不新造近义词。
5. 在归属 MOC 的对应小节末尾加一行 `- [[卡片名]]`；若该 `##` 下已用 `###` 分组，则加进对应分组。
6. 写双向关联：卡片末尾加 `## 关联` 区，沿用 vault 格式 `- [[卡片名]] —— 一句话说清关系`，
   只链真正相关的已有卡片（因果/互补/操作依赖）；并同步在被链卡片的 `## 关联` 补回链。不为凑数硬链。
   约束——**正文禁止内联链接**：卡片正文（`## 关联` 区除外）不得出现任何 `[[...]]`。
7. 自检关联闭环：逐个打开新卡 `## 关联` 中的链接，确认对方卡片的 `## 关联` 已有回链；缺则补上。
   同时 grep 新卡正文，确认 `## 关联` 区之外无任何 `[[...]]` 内联链接（含 MOC 列表与 `## 关联` 之外的区块均不得出现）。
8. 同步 README（位置与计数方法见 AGENTS.md「README 索引同步」）；
   若本次牵动 MOC 分层（如新建 / 拆分二级 MOC、卡片在一二级 MOC 间迁移），还需同步 README「一、」散文段与 `00-Index/Index.md`。

## 最小示例（概念卡）
任务：新增「npx和npm exec的区别」
1. 校验：fetch npm 官方文档（docs.npmjs.com/cli/commands/npm-exec）确认 npx 自 npm 7 起基于 npm exec 实现，无误后继续
2. 类型：概念卡 → concept-card.md；归属：MOC-工程化（主语是包管理）
3. 写 `02-Notes/npx和npm exec的区别.md`，frontmatter：
   type: concept / tags: [node, package-manager] / created: <当天> / aliases: [npm exec] / source: 官方文档链接
4. 在 `MOC-工程化`「包管理」小节末加 `- [[npx和npm exec的区别]]`
5. 卡末 `## 关联` 写 `- [[npx是什么]] —— npx 即 npm exec 的快捷入口`，并到 `npx是什么` 的 `## 关联` 补回链
6. 自检两卡互链可达；README 同步：02-Notes 总数 +1、两处卡名清单补入、MOC-工程化「已链 N 张」+1

## 注意事项
- 通用 howto/配方卡留在 02-Notes（05-Templates 有 howto-card，属合规用法）。
- 某次项目的偶发踩坑、具体项目真实配置，归 03-Projects 的具体项目笔记，不要堆进通用卡。
- 知识点校验是硬性前置：发现错误先指出并给依据，不顺着错误继续。
- 正文零内联链接（硬规则）：所有卡片间 `[[...]]` 必须集中在 `## 关联` 区（及 MOC 列表），正文任何位置禁用；
  图谱只认 `[[...]]` 不分区，正文内联会制造非预期的悬空边，与双向可达约定冲突。
