# 大前端 Obsidian Vault — AI 协作约定

## 仓库概览
个人知识库（前端→Node 全栈）；方法：PARA + Zettelkasten + Obsidian 双链/MOC

## 核心规则
- 知识点先校验再写卡（最高优先级）：生成卡片前先校验知识点正确性，以官方文档 / RFC / 权威源码为准，不确定就先 fetch 官方文档核实；若用户说法有误，先明确指出并给出依据，纠正后再继续，不顺着错误写卡
- 新增原子卡：套 05-Templates 模板、frontmatter 字段以所选模板为准、以概念命名
- 单一归属：卡片只链**唯一的最末级 MOC**（未开二级时即一级 MOC 本身）；上一级 MOC 只链下一级 MOC、不再直接链卡片，并在卡末 `## 关联` 双向回链
- 内容边界：通用 howto 留 02-Notes；项目偶发踩坑归 03-Projects
- tags 沿用既有卡片词汇（如 node、package-manager、moc、engineering），不新造近义词
- README 索引同步（强制）：02-Notes 增删、卡片改名或 MOC 链接变化后，必须同步 README 两处表格——「二、目录编排」02-Notes 行（总数 + 卡名清单）、文末 MOC 归属表（已链数 + 卡名清单），否则误导后续 agent。
  计数方法：总数 = `ls 02-Notes | wc -l`（如有占位文件须排除）；MOC 已链数 = `grep -c '^- \[\[' 00-Index/<MOC文件名>.md`

## MOC 分层约定（一级 / 二级）
- 层级封顶两级：`MOC-领域.md`（一级）→ `MOC-领域-子主题.md`（二级），全部平铺在 `00-Index`，不开三级
- 开二级的判据（满足任一）：一级 MOC 总链接数 ≥ 30；单个 `##` 小节链接数 ≥ 10；某子主题自身能分出 ≥ 6 个 `##` 小节
- 切口原则：切在「平行兄弟」层，不切在「子集」层——对：`MOC-框架-Vue` / `MOC-框架-React`（Vue、React 平行）；错：`MOC-框架-VueRouter`（路由是 Vue 的子集，开在这里会把日后的响应式 / 状态管理逼向三级）
- 二级 MOC 内小节膨胀的处置：优先在该 `##` 下加 `###` 分组；或在概念确实可再分时降低卡片粒度（判据仍是「能否独立讲清一个概念」，不得为排版硬拆）；一律不得再开三级 MOC
- 开二级时必须同步：README「一、」散文段 +「二、目录编排」表 +「四、领域全景」表 + `00-Index/Index.md` + `vault-add-card` / `vault-refactor-card` 两处 skill

## 安全分级
- L1 只读（读模板 / 搜索 / 读 MOC / fetch 查文档）：直接执行
- L2 增删改（新建 / 改名 / 移动卡片、改 MOC / README / 05-Templates 模板 / AGENTS.md / `.agents/skills` 下 SKILL.md）：先说明将动的文件路径与内容概要，获用户同意后执行
- L3 高危（装包、改远程、git push、删目录、写家目录）：必须显式确认，并给出可回滚方案

## 可用 Skills（按需加载 .agents/skills/ 下对应 SKILL.md）
- vault-add-card：新增原子卡全流程
- vault-refactor-card：改名/移动全库链同步
