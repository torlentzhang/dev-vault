# 项目长期记忆：大前端知识库

## 笔记体系约定（卡片盒 Zettelkasten）
- 采用卡片笔记法：原子化（一卡一知识点）、用自己的话、双链 `[[连接]]` 织网
- 模板位置：`05-Templates/`，四种类型由 `type:` 字段区分：
  - `concept-card.md` 概念卡（type: concept）
  - `literature-card.md` 文献卡（type: literature）
  - `howto-card.md` 方法/踩坑卡（type: howto）
  - `moc.md` 索引卡 / Map of Content（type: moc）
- 通用模板 `note-template.md`、`daily-template.md` 保留不动
- 约定：纯 Markdown + YAML frontmatter，均带 `created:` 和 `tags:`，靠双链而非文件夹分类
- 已有索引入口：`00-Index/`（含 nvm+pnpm 笔记）；日记在 `01-Daily/`

## 阿博纠正过的工作纪律（务必遵守，防止再犯）
- **改文件前先出方案待确认**：涉及笔记 / 模板等内容的新增或修改，必须先给出设计思路与内容草案，待阿博明确同意后再动笔，不要直接写文件。（这也是全局文件操作确认规则的要求）
- **卡片写可沉淀的通用知识**：知识要抽象成放之四海皆准的要点，不记录机器特定的临时状态（如"本机 pnpm 由 corepack 提供"）；机器 / 环境细节应抽象为通用建议，或连回对应卡（如 corepack、nvm 踩坑卡），而非写当前机器现状。
