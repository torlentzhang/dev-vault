# 大前端知识库 · README

> 个人技术知识库，以「前端 → Node → 全栈」为主线，沉淀长期可理解、可检索、可复用的知识卡片。
> 覆盖范围：Web 平台 / JavaScript / TypeScript / 浏览器 / 前端框架 / Node 服务端 / 工程化 / DevOps / 版本协作。

## 一、知识体系

采用 **Zettelkasten（原子卡片）+ Obsidian 双链 + MOC（内容地图）** 的组合，并引入 AI Agent 负责检索、创建、重构与日常维护。

- **原子卡片**：以「一个可独立理解、复习、复用的知识主题」为基本单元，写在 `Notes/`。
- **双链**：卡片之间用 `[[...]]` 建立真实关系，形成可生长的知识网络，而非线性文档堆。
- **MOC**：用 `MOC-` 前缀命名的内容地图，写在 `Index/`，负责导航而非严格分类。
- **PARA**：作为组织工作流的长期方向（区分「流动内容」与「长期沉淀」），目前尚未落地为独立目录。

核心原则：

> **目录负责存放，卡片负责知识，MOC 负责导航，双链负责关系，Tags 负责属性。**

这五件事互不等价——不要用目录层级、MOC 或 Tags 互相代替。判断一张卡片该放哪里，先看它回答的是哪个「核心问题」。

## 二、目录结构

```text
dev-vault/
├── Index/          # MOC 与顶层 Index，全部平铺，不建嵌套文件夹
├── Notes/          # 原子知识卡，直接以概念命名
├── Templates/      # Obsidian 模板
├── .agents/skills/ # Agent Skills（新增 / 重构卡片的可执行流程）
├── AGENTS.md       # AI 协作与治理规则
└── README.md       # 本文件
```

| 目录 | 用途 | 当前内容 |
|------|------|----------|
| `Index/` | MOC 内容地图，用 `MOC-` 前缀分层，全部平铺 | `Index.md`（顶层大地图）+ 9 个 L1 领域 MOC + 3 个 L2（`MOC-Frontend-Vue`、`MOC-Web-CSS`、`MOC-Engineering-PackageManagement`）+ 1 个 L3（`MOC-Frontend-Vue-Router`），共 14 个文件 |
| `Notes/` | 原子卡片（一张卡一个主题） | 37 张 |
| `Templates/` | 笔记模板 | 4 个：`concept-card` / `literature-card` / `howto-card` / `moc` |
| `.agents/skills/` | Agent Skill 说明 | `vault-add-card`（新增卡片）、`vault-refactor-card`（改名 / 移动卡片） |

> 说明：`Index` / `Notes` 中的内容全部纳入 Git；`.obsidian` 的插件代码与运行时数据、`.workbuddy/` 工作区记忆由 `.gitignore` 排除。

## 三、MOC 层级

MOC 是**导航地图，不是严格分类树**，推荐最多三层，**禁止创建 L4**：

```text
L1 Domain    MOC-Frontend
└── L2 Subdomain    MOC-Frontend-Vue
    └── L3 Topic    MOC-Frontend-Vue-Router
```

- **L1（领域）**：稳定的大知识领域，每个用「核心问题」界定边界。
- **L2（子领域）**：领域内部相对稳定的知识系统。
- **L3（主题）**：仅当某主题本身已形成独立知识地图时才创建；**允许 L3 存在，不代表应该主动创建 L3**。

只有当具备「稳定的知识边界、独立的导航价值、长期增长可能、多个相关知识主题」时才新建 MOC；卡片数量只作辅助参考，不能机械触发。若 L3 仍然过大，优先在 MOC 内部用 `##` / `###` 重组或拆分知识卡，而不是继续加层级。

## 四、领域全景（9 个 L1 MOC）

| L1 MOC | 核心问题 | 当前状态 |
|--------|----------|----------|
| `MOC-Web` | Web 平台本身是什么？ | 已开 L2 `MOC-Web-CSS`（7 张：基础与基线 / 预编译语言 / 原子化 CSS），一级仅留 `Iconify为什么是图标基础设施`；HTML 与布局主题待沉淀 |
| `MOC-JavaScript` | JS 语言本身怎么工作？ | 空骨架，待填 |
| `MOC-TypeScript` | TS 类型系统与工程能力是什么？ | 空骨架，待填 |
| `MOC-Browser` | 浏览器如何运行 Web？ | 空骨架，待填 |
| `MOC-Frontend` | 如何构建现代前端应用？ | 已开 L2 `MOC-Frontend-Vue`，一级只链下一级 MOC |
| `MOC-Node` | 如何使用 JS 构建服务端？ | 空骨架，待填 |
| `MOC-Engineering` | 如何开发、构建、管理和发布代码？ | 已开 L2 `MOC-Engineering-PackageManagement`（10 张：pnpm / npm / corepack / npx），一级留 `nvm切换node导致pnpm不兼容`；本地规范 / 构建工具待填 |
| `MOC-DevOps` | 如何部署、运行和维护软件？ | 空骨架，待填 |
| `MOC-Development` | 如何进行版本控制和团队协作？ | 已链 1 张（`git fork 协作模式`），其余小节待填 |

已展开的下级 MOC：

| 层级 | MOC | 当前状态 |
|------|-----|----------|
| L2 | `MOC-Frontend-Vue`（Vue 子领域） | 已链 1 张（`vite+Vue3工程的初始化与启动`）+ 指向 L3 路由地图 |
| L2 | `MOC-Engineering-PackageManagement`（依赖与包管理器子领域） | 已链 10 张，按「版本语义 / 安装、更新与执行 / 解析与安全 / 配置与环境」4 个小节组织 |
| L2 | `MOC-Web-CSS`（CSS 子领域） | 已链 7 张，按「基础与基线 / 预编译语言 / 原子化 CSS」3 个小节组织 |
| L3 | `MOC-Frontend-Vue-Router`（Vue Router 主题） | 已链 16 张，按「自动路由 / 原理 / 配置实战 / 进阶」4 个小节组织 |

> 入口：`Index/Index.md` 顶层大地图链接全部 9 个 L1；下级 MOC 由上级 MOC 进入。

## 五、使用规范要点

以下约定已固化，新增或修改内容时请遵循；完整的治理原则见根目录 `AGENTS.md`，具体操作流程见 `.agents/skills/`。

1. **目录职责**：`Index/` 只放 MOC 与 Index，`Notes/` 只放原子卡片，`Templates/` 只放模板，不混放。
2. **卡片命名**：优先「稳定、简洁、可检索的概念名称」，如 `pnpm`、`Vue Router`、`Hash 与 History`、`normalize 与 reset`；避免 `xxx是什么` / `xxx怎么理解` 这类问句式命名（历史卡片中仍存在少量此类名称，新增请遵循新规范）。只有问题本身具备长期知识价值时才用问句式。
3. **双链的书写位置**：卡片之间的 `[[...]]` 集中写在卡片末尾的 `## 关联` 区，**禁止在正文内联**（MOC 列表与 `## 关联` 区除外），以免制造非预期的悬空边。
4. **多归属**：一张卡片可以被多个 MOC 引用，但**禁止创建重复知识**（不要出现 `pnpm` / `node-pnpm` / `package-pnpm` 三份）。分类时判断的是「知识的本质是什么」，而不是「它在哪里被使用」。
5. **Tags 是横向属性**：用于表达技术领域、知识类型、学习状态、工具等，不承担目录或 MOC 的分类职责；新增 Tag 前先检查是否已有语义相近的 Tag。
6. **frontmatter**：统一包含 `type` / `tags` / `created`；其中 `concept` 卡额外含 `aliases` / `source`，`literature` 卡额外含 `source` / `author`。新建笔记请基于 `Templates/` 中的对应模板。
7. **新增卡片**：建双链 —— 在归属 MOC 的相应小节追加 `- [[卡片名]]`；在卡片 `## 关联` 区链到真正相关的卡片，并在被链卡片中补回链，保证双向可达（只链真正相关的几张，不凑数）。
8. **改名 / 移动卡片**：用正则 `\[\[旧名(\|[^\]]*)?\]\]` 全库搜索并替换（MOC 列表、其他卡片的 `## 关联`、正文内联引用三处都要查），改完复查须 0 命中；同时检查 `aliases` 是否失效。

## 六、AI 协作

`AGENTS.md` 是本仓库的治理文件，定义 Vault 定位、MOC 规则、命名与 Tags 约定、Agent 权限分级（R0 只读 / R1 工作区修改 / R2 高风险操作）以及大规模重构的流程要求。复杂操作优先交给 `.agents/skills/` 下的 Skill 完成，Skill 与 `AGENTS.md` 冲突时以 `AGENTS.md` 的高层治理原则为准。

> **索引同步提醒**：新增 / 删除卡片、卡片改名、新增或改名 MOC、MOC 结构变化后，需检查并更新 `README.md` 与 `Index/` 中受影响的计数和链接。
