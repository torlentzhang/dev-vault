# 大前端 Obsidian Vault — AI 协作约定

## 仓库概览
个人知识库（前端→Node 全栈）；方法：PARA + Zettelkasten + Obsidian 双链/MOC

## 核心规则
- 知识点先校验再写卡（最高优先级）：生成卡片前先校验知识点正确性，以官方文档 / RFC / 权威源码为准，不确定就先 fetch 官方文档核实；若用户说法有误，先明确指出并给出依据，纠正后再继续，不顺着错误写卡
- 新增原子卡：套 05-Templates 模板、frontmatter 字段以所选模板为准、以概念命名
- 单一归属：卡片只链唯一 MOC，并在卡末 `## 关联` 双向回链
- 内容边界：通用 howto 留 02-Notes；项目偶发踩坑归 03-Projects
- tags 沿用既有卡片词汇（如 node、package-manager、moc、engineering），不新造近义词
- README 索引同步（强制）：02-Notes 增删、卡片改名或 MOC 链接变化后，必须同步 README 两处表格——「二、目录编排」02-Notes 行（总数 + 卡名清单）、文末 MOC 归属表（已链数 + 卡名清单），否则误导后续 agent。
  计数方法：总数 = `ls 02-Notes | wc -l`（如有占位文件须排除）；MOC 已链数 = `grep -c '^- \[\[' 00-Index/<MOC文件名>.md`

## 安全分级
- L1 只读（读模板 / 搜索 / 读 MOC / fetch 查文档）：直接执行
- L2 增删改（新建 / 改名 / 移动卡片、改 MOC 与 README）：先说明将动的文件路径与内容概要，获用户同意后执行
- L3 高危（装包、改远程、git push、删目录、写家目录）：必须显式确认，并给出可回滚方案

## 可用 Skills（按需加载 .agents/skills/ 下对应 SKILL.md）
- vault-add-card：新增原子卡全流程
- vault-refactor-card：改名/移动全库链同步
