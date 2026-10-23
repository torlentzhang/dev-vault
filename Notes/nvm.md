---
type: concept
tags: [node, nvm, 版本管理, 工具]
created: 2026-10-19
aliases: [Node 版本管理器, nvm.sh]
source:
  - https://github.com/nvm-sh/nvm
---
# nvm

## 一句话定义
nvm 是一个用 shell 脚本实现的 node 版本管理器：它把目标版本的 `~/.nvm/versions/node/<版本>/bin` 插到 `PATH` 最前面，让同一台机器上的多个 node 版本随时切换。

## 用于解决什么问题
一台机器只装一个 node 时，切换项目就得卸载重装，某个项目指定的版本会把另一个项目直接弄到启动失败；想并存多个版本只能手动改 PATH 指向对应目录，忘了切回来时也看不出当前用的是哪一套。

## 要点
- **本质是 shell 函数，不是可执行文件**：nvm 靠 `source "$NVM_DIR/nvm.sh"` 加载进当前 shell，所以 `which nvm` 无效，验证要用 `command -v nvm`；也因此安装后必须**新开终端**才生效。默认安装目录 `$HOME/.nvm`（若设置了 `$XDG_CONFIG_HOME` 则为 `$XDG_CONFIG_HOME/nvm`）。
- **切换靠 PATH 插队**：`nvm use 18` 只影响当前 shell 的 PATH 顺序，进程退出即失效，不写进任何全局配置。
- **默认版本是另一件事**：`nvm alias default 18` 决定的是「以后新开 shell 从哪个版本起步」，而不是立即切换当前 shell（`nvm use` 才负责当前 shell）。第一个安装的版本会成为默认。
- **`.nvmrc` 做项目级声明**：项目根放一行版本（`22` / `lts/*` / `node`），`nvm use` 与 `nvm install` 不带参数时自动读取；查找会**从当前目录向上级回溯**，所以在任意子目录都命中；`nvm which` 同样读取，都没找到时以退出码 127 结束。（`nvm exec` / `nvm run` 也做同样查找，但未命中时回退到当前 node，README 称其为 undefined behavior。）
- **每个 node 版本有独立全局包目录**：`~/.nvm/versions/node/<版本>/lib/node_modules`，切版本后全局命令「不见了」是正常现象。
- **Windows 是另一个项目**：原始 nvm 只支持 Linux / macOS（Windows 需 WSL / Git Bash / Cygwin）；原生 Windows 用的 NVM for Windows 是独立项目，官方 README 明确说这类替代品 *"neither supported nor developed by us"*。
- **不要用 npm 安装**：官方只提供 install.sh（curl / wget）与 git clone；npm registry 上的同名 `nvm` 包是完全无关的废弃包，废弃说明原文为 *"This is NOT the correct nvm."*

常用命令：

| 命令 | 作用 |
| --- | --- |
| `nvm install 22` | 安装指定版本（`nvm install node` 装最新） |
| `nvm use 22` | 当前 shell 切到该版本 |
| `nvm ls` / `nvm ls-remote` | 已安装列表 / 可安装列表 |
| `nvm current` | 当前 shell 实际生效的版本（不受 `.nvmrc` 影响） |
| `nvm alias default 22` | 设定新 shell 的默认版本 |
| `nvm exec 22 node -v` | 用指定版本在子 shell 里跑命令 |
| `nvm which 22` | 打印该版本可执行文件路径 |

## 我的理解 / 类比
把 nvm 想成一套「node 试衣间」：机器里同时挂着好几件版本，`nvm use` 是当场换上哪一件（只对你这个 shell 有效），`nvm alias default` 是设定明早出门默认穿哪一件。

## 实战建议
- 项目里放 `.nvmrc` 并提交进仓库，团队就能用同一个 node 版本；但**声明了不等于自动切换**，默认仍要手动 `nvm use`（README 提供的自动 cd 钩子属社区方案，nvm 维护者不提供支持）。
- 想固定「这个项目就用这个版本」，可以用 `nvm alias <项目名> 22` 自定义别名。
- 切版本后要先在新版本下重装全局包，或直接用 `nvm install 22 --reinstall-packages-from=18` 迁移；该参数**不会**顺带升级 npm，需要时另加 `--latest-npm`。

## 关联
- [[npm全局安装路径]] —— 原理与实践：本卡讲「怎么切版本」，那张卡讲「切完之后全局包落到哪」，两者合起来才是 nvm 的完整机制
- [[nvm切换node导致pnpm不兼容]] —— 因果：切版本后全局 pnpm 罢工，根因就是本卡讲的版本隔离
- [[Corepack]] —— 互补：与其依赖全局安装的包管理器，不如用 corepack 按项目声明版本
