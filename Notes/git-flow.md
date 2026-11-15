---
type: howto
tags: [git, 分支, 协作, git-flow]
created: 2026-11-13
aliases: [Git Flow, 分支模型]
---
# git-flow

## 适用场景

两条长期主分支加三类临时支撑分支的分支模型，2010 年由 Vincent Driessen 提出。适合**明确版本化、需要同时维护多个线上版本**的项目。持续交付的 Web 应用通常用不上它的全部结构，用其精简子集即可。

## 分支角色

| 分支 | 从哪切出 | 合回哪里 | 生命周期 |
| --- | --- | --- | --- |
| `master` | — | — | 长期。HEAD 始终是生产就绪状态，每次合入即一次发布并打 tag |
| `develop` | — | — | 长期。集成分支，承接下一次发布的全部开发 |
| `feature-*` | `develop` | `develop` | 临时 |
| `release-*` | `develop` | `master` **和** `develop` | 临时 |
| `hotfix-*` | `master` | `master` **和** `develop` | 临时 |

## 步骤

### 1. 功能开发放在 feature 分支
```bash
git switch develop
git switch -c myfeature
```
分支名不得是 `master`、`develop`、`release-*`、`hotfix-*`。

### 2. 功能完成，用 --no-ff 合回 develop
```bash
git switch develop
git merge --no-ff myfeature
git branch -d myfeature
git push origin develop
```

### 3. 准备发布：从 develop 切 release 分支
```bash
git switch develop
git switch -c release-1.2
# 改版本号、修最后的 bug，不再加新功能
```
版本号在切 release 分支这一刻才确定，而不是更早。

### 4. 发布：合入 master 并打 tag，再合回 develop
```bash
git switch master
git merge --no-ff release-1.2
git tag -a 1.2

git switch develop
git merge --no-ff release-1.2   # 关键：不回灌，下个版本就丢了这些修复
git branch -d release-1.2
```

### 5. 线上紧急修复：hotfix 从 master 切出
```bash
git switch -c hotfix-1.2.1 master
# 修 bug
git switch master && git merge --no-ff hotfix-1.2.1 && git tag -a 1.2.1
git switch develop && git merge --no-ff hotfix-1.2.1
```
例外：若当前存在 `release-*` 分支，hotfix 应合回该 release 分支而不是 `develop`，修复会在 release 完成时一并进入 `develop`。

## 为什么合入一律用 --no-ff

`--no-ff` 强制生成合并提交，即使本来可以快进。代价是多一个空提交，换来两件事：

- 历史里留下「这个功能分支存在过」的痕迹，一组提交被清晰归组；
- **整组回滚**变得简单——revert 那条合并提交即可。快进合并后各提交平铺在主干上，只能逐个 revert。

## 精简用法：只留两条主分支

把 `feature-*` / `release-*` / `hotfix-*` 全部去掉，只保留 `master` + `dev`（即 `develop` 改名）两条长期分支，全部日常开发直接提交到 `dev`，再定期 PR 回 `master`。支撑分支带来的收益（并行开发、发布隔离）单人项目用不上，去掉后流程只剩一条不变式：

> **`master` 始终是 `dev` 的祖先。**

维持这条不变式靠两件事：

1. **PR 合并方式必须是 Create a merge commit**，不能 Squash，也不能 Rebase and merge；
2. 合并后立刻把 `master` 快进回 `dev`：

```bash
git fetch origin
git switch dev
git merge --ff-only origin/master
git push origin dev

git rev-parse dev origin/master   # 两个哈希应一致
```

`--ff-only` 是保险丝：快进不成立时直接报错退出，而不是默默造出一条多余合并提交。

## 注意事项 / 踩坑

- **Squash and merge 会彻底破坏回同步**。它把 `dev` 的多个提交压成 `master` 上一条全新提交，与 `dev` 原提交无祖先关系，`dev` 从此与 `master` 分叉，`--ff-only` 必然失败。GitHub 文档还指出：squash 后继续在同一 head 分支开发，之前已 squash 的提交会再次出现在新 PR 里，冲突也要反复解决。
- **Rebase and merge 同理**。它把 `dev` 的提交改写成副本放到 `master` 上，`dev` 的原提交仍在，同样分叉。
- **合并方式误选后的补救**：
  ```bash
  git rebase origin/master                  # 内容已在上游的提交会被自动丢弃
  git push --force-with-lease origin dev    # 必须带 --force-with-lease
  ```
  冲突就 `git rebase --abort`，改用 `git reset --hard origin/master` 重新对齐。
- **`--force-with-lease` 不要写成 `--force`**：它以远程跟踪引用为租约，远端若已被别人推过就拒绝覆盖，避免吞掉他人的提交。
- **`dev` 上有未推送提交时** `--ff-only` 会拒绝。此时改用 `git rebase origin/master`：重放的只是合并后新增的那几个提交，推送仍是快进，无需 force。
- **不要对已推送到 `dev` 的提交做 `amend` / `rebase -i`**：改写公共历史后回同步和后续 PR 都会变乱。
- **`--no-ff` 的反面代价**：合并提交多了以后 `git log` 噪音变大，`git bisect` 也更绕。这是 git-flow 被批评的地方，也是作者本人在 2020 年补充反思、建议持续交付项目改用更简单流程的原因。
- **CLI 工具现状**：原 `nvie/gitflow` 与社区版 `git-flow-avh` 两个仓库均已归档；在维护的是 `gittower/git-flow-next`。

## 关联
- [[git fork 协作模式]] —— 同样以 PR/MR 汇入主干，区别在于它解决「无推送权限的跨仓贡献」，本卡解决「仓库内的分支组织与版本发布」。
