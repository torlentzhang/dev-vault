# dev-vault Agent Wiki · README

> 以「前端 → Node → 全栈」为主线的 Agent Wiki，主要供 Agent 按需检索和复用，写入由用户显式触发，同时保留 Obsidian 人工阅读与维护能力。

## 项目目的

采用 **Zettelkasten（原子卡片）+ Obsidian 双链 + MOC（内容地图）** 的组合。Agent 按需检索已有知识，用户显式触发创建、整理与维护，目标是构建一张长期可理解、可检索、可扩展的知识网络，而非线性文档堆。

- **原子卡片**：以「一个可独立理解、复习、复用的知识主题」为基本单元，写在 `Notes/`。
- **双链**：卡片之间用 `[[...]]` 建立真实关系，形成可生长的知识网。
- **MOC**：用 `MOC-` 前缀命名的内容地图，写在 `Index/`，负责导航而非严格分类。
- **PARA**：作为组织工作流的长期方向（区分「流动内容」与「长期沉淀」），目前尚未落地为独立目录。

核心原则：

> **目录负责存放，卡片负责知识，MOC 负责导航，双链负责关系，Tags 负责属性。**

这五件事互不等价——不要用目录层级、MOC 或 Tags 互相代替。

## 仓库结构

```text
dev-vault/
├── Index/          # 导航产物：顶层 Index 与分层 MOC
├── Notes/          # 原子知识卡，以概念命名，平铺存放
├── Templates/      # Obsidian 模板（concept / howto / literature / moc / daily）
├── Daily/          # 日记，按年分目录
├── .agents/skills/ # Agent Skills（新增卡片、主题学习等可执行流程）
├── AGENTS.md       # AI 协作与治理规则
└── README.md       # 本文件
```

| 目录 | 用途 |
| --- | --- |
| `Index/` | MOC 内容地图，按 `L1 Domain` / `L2 Subdomain` / `L3 Topic` 分层存放；`Index.md` 为顶层入口 |
| `Notes/` | 原子卡片，一张卡一个主题，平铺存放，不按 MOC 建子目录 |
| `Templates/` | 卡片与 MOC 模板 |
| `Daily/` | 日记，按年分目录 |
| `.agents/skills/` | Agent Skill 说明 |

## 延伸入口

- 治理规则与 Agent 权限：`AGENTS.md`
- 内容导航入口：`Index/Index.md`
- 自动化操作流程：`.agents/skills/`
