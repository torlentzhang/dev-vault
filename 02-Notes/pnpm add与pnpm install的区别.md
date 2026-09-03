---
type: concept
tags: [pnpm, 包管理器, 命令行, 依赖安装]
created: 2026-09-03
---
# pnpm add 与 pnpm install 的区别

## 一句话定义
`pnpm install` 是**按 package.json 同步全部依赖**；`pnpm add <pkg>` 是**装新包并写进 package.json**。pnpm 刻意把 npm 里语义混乱的 `npm install` 拆成了两个各司其职的命令。

## 要点

### 混乱的根源：npm 的 install 有双重语义
```bash
npm install          # 装全部依赖
npm install lodash   # 装新包 —— 同一个命令，完全不同的行为
```
pnpm 把这个拆开了，语义清晰得多：
```bash
pnpm install         # 按 package.json + lockfile 装全部依赖，不改 package.json
pnpm add lodash      # 装新包，并写入 dependencies
```

### 命令对照表
| 目的 | pnpm | npm |
|---|---|---|
| 装全部依赖 | `pnpm install`（简写 `pnpm i`） | `npm install` |
| 装生产依赖 | `pnpm add <pkg>`（默认） | `npm i <pkg>` |
| 装开发依赖 | `pnpm add -D <pkg>` | `npm i -D <pkg>` |
| 装可选依赖 | `pnpm add -O <pkg>` | `npm i -O <pkg>` |
| 锁精确版本（不写 `^`） | `pnpm add -E <pkg>` | `npm i -E <pkg>` |
| 全局安装 | `pnpm add -g <pkg>` | `npm i -g <pkg>` |
| 删包 | `pnpm remove <pkg>` | `npm uninstall <pkg>` |

### `pnpm install` 高频选项
- **`--frozen-lockfile`** —— lockfile 与 package.json 不一致就报错。**CI 环境默认 `true`**，本地默认 `false`。这是 pnpm 比 npm 严谨的地方，能防住"我这儿能跑"问题
- `--prod` / `-P` —— 只装 dependencies，跳过 devDeps（部署用）。注意：无 lockfile 时仍会解析 devDeps
- `--offline` —— 只从本地 store 装；`--prefer-offline` —— 跳过新鲜度检查，缺的再去拉
- `--lockfile-only` —— 只更新 lockfile，不动 node_modules
- `--fix-lockfile` —— lockfile 条目损坏时自动修
- `--force` —— 强制重装，重建 lockfile 与 node_modules（排查灵异问题用）

### `pnpm add` 高频选项
- `-w` —— 在 monorepo 里装到根 package。**不加这个往根 package 装会直接报错**
- `--save-peer` —— 写进 peerDependencies，同时作为 devDep 安装（写库时用）
- `--save-catalog` / `--save-catalog-name <name>` —— 存到 pnpm catalogs（v10.12.1+）
- `--allow-build=<pkg>` —— 安装时直接放行该包的构建脚本（v10.4.0+）
- `--filter <selector>` —— 只给指定的 workspace 子包添加

## 我的理解 / 类比
把 `install` 想成"**照单抓药**"——拿着 package.json 这张药方，把该有的补齐；把 `add` 想成"**加一味药**"——抓新药的同时，顺手把药方改了。npm 用同一个动词干两件事，pnpm 拆成两个，代价是多记一个命令，收益是永远不会误改 package.json。

还有个容易忽略的设计：`--frozen-lockfile` 在 CI 下默认开启，意味着**lockfile 过期 = 构建失败**，而不是悄悄装个新版本上去。这是把"依赖不确定性"提前暴露在 CI 阶段，而不是留到线上炸。

## 关联
- [[pnpm为什么拦截自执行脚本]] —— **装完却跑不起来时先查这篇**，pnpm 10 默认不跑 postinstall
- [[pnpm处理peer依赖警告]]
- [[corepack是什么]]
- [[nvm切换node导致pnpm不兼容]]
