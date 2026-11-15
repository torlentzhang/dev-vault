# obsidian-skills 来源记录

本目录 `.agents/skills/` 下的 7 个技能来自官方仓库 **obsidianmd/obsidian-skills**，此处记录版本与更新方法。这不是技能文件——DSH 只扫描 `.agents/skills/` 的子目录，本文放在 `.agents/` 根下不会被当作技能解析。

## 来源

| 项 | 值 |
| --- | --- |
| 仓库 | https://github.com/obsidianmd/obsidian-skills |
| 许可证 | MIT，Copyright (c) 2026 Steph Ango (@kepano)，全文见同目录 `obsidian-skills-LICENSE.txt` |
| 锁定 commit | `398127736bd4ba42d767b1419fee1a927508370b` |
| commit 日期 | 2026-11-11 |
| 对应版本 | `plugin.json` 声明 1.2.0 |
| 安装日期 | 2026-11-15 |

## 为什么锁 commit 而不锁 tag

仓库仅有 `1.1.0` 一个 tag，但 `plugin.json` 已声明 `1.2.0` —— 版本号在 manifest 里递增，未补打 tag。因此锁 tag 会拿到旧内容，锁 commit SHA 才准确对应 1.2.0。

## 已安装的 7 个技能

源路径：`plugins/obsidian/skills/`，平铺拷贝到 `.agents/skills/`。

| 技能 | 大小 | 依赖 |
| --- | --- | --- |
| `obsidian-markdown` | 6.1 KB + references 3.5 KB | 无 |
| `obsidian-bases` | 17.1 KB + references 7.5 KB | 无 |
| `json-canvas` | 8.7 KB + references 6.5 KB | 无 |
| `obsidian-cli` | 6.4 KB | `obsidian` CLI，且 Obsidian 桌面端需运行 |
| `obsidian-headless` | 4.5 KB | `ob` CLI（`npm i -g obsidian-headless`）+ Sync/Publish 订阅 |
| `defuddle` | 0.9 KB | `defuddle` CLI（`npm i -g defuddle`） |
| `knap` | 4.1 KB | Knap CLI |

无依赖的三个（markdown / bases / canvas）装上即可用；其余四个在依赖装好前调用只会提示安装方式。

## 为什么是平铺而不是嵌套

上游仓库对 OpenCode 建议「clone 整个仓库、保留 `plugins/obsidian/skills/<name>/` 结构」，因为 OpenCode 递归搜索 `**/SKILL.md`。DeepSeek Harness 的本地技能 provider **不递归**，只扫描根目录的直接子项并识别 `<name>/SKILL.md`，所以必须把 7 个技能目录平铺到 `.agents/skills/` 下；保留 `obsidian-skills/` 这层嵌套会导致一个都发现不了。

同理，上游的 `.claude-plugin/`、`.cursor-plugin/`、`.agents/plugins/marketplace.json`、`plugin.json`、`assets/icon.png` 均未拷贝——它们服务于 Claude Code / Cursor / Codex 的插件市场，DSH 不读取。

## 更新方法

改动 SHA 后重跑：

```bash
cd /Users/torlent/Documents/dev-vault

SHA=<新的 commit SHA>
TMP=$(mktemp -d)
curl -sL --fail -o "$TMP/o.tar.gz" \
  "https://codeload.github.com/obsidianmd/obsidian-skills/tar.gz/$SHA"
tar -xzf "$TMP/o.tar.gz" -C "$TMP"
SRC="$TMP/obsidian-skills-$SHA/plugins/obsidian/skills"

rm -rf .agents/skills/{defuddle,json-canvas,knap,obsidian-bases,obsidian-cli,obsidian-headless,obsidian-markdown}
cp -R "$SRC"/{defuddle,json-canvas,knap,obsidian-bases,obsidian-cli,obsidian-headless,obsidian-markdown} .agents/skills/
cp "$TMP/obsidian-skills-$SHA/LICENSE" .agents/obsidian-skills-LICENSE.txt

rm -rf "$TMP"
```

更新后同步修改本文件里的「锁定 commit」「commit 日期」「对应版本」三行。

查最新 commit：

```bash
curl -s https://api.github.com/repos/obsidianmd/obsidian-skills/commits/main \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['sha'])"
```

## 注意

- 上游发布新版时可能改动 `SKILL.md` 的 `name` 字段。DSH 要求 name 匹配 `^[a-z0-9]+(?:-[a-z0-9]+)*$`，不合规的技能会被静默丢弃（仅一条 warn 日志），更新后建议核对技能目录条数。
- 7 条技能描述合计约 1772 字符，是每轮对话的固定上下文开销，与是否实际调用无关。
