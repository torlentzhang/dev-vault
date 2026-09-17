---
type: howto
tags: [git, 协作, fork, 远程]
created: 2026-09-14
---
# Git Fork 协作模式操作指南

## 适用场景
参与团队仓库（开源或内部）开发、但没有直接推送权限时采用的协作方式：先把团队仓库 fork 到个人账号，在个人仓开发，再通过合并请求（PR/MR）把改动合回团队仓库。适用于开源贡献、跨团队协同、需要代码审查的提交流程。

## 步骤

### 1. fork + clone 到自己本地
- 在平台把团队仓库 **fork** 到个人账号；
- clone **你自己账号下的那份**（不是团队原仓库）到本地：
  ```bash
  git clone <你个人仓地址>
  ```
> 坑：直接 clone 团队原仓库也能拉下来，但没有 push 权限，写完推不上去。

### 2. 配置 upstream 并同步上游
- 只需一次：把团队原仓库设为 `upstream`（只用来拉）：
  ```bash
  git remote add upstream <团队仓库地址>
  ```
- 拉取并同步上游更新（rebase 保持线性历史）：
  ```bash
  git pull --rebase upstream main
  ```
> `git pull` 默认只认 `origin`。fork 模式里拉团队更新**必须显式写 `upstream`**；`git pull --rebase upstream main` 已内含 fetch，不必先单独 fetch（拆成 `fetch` + `rebase` 只是更可控的备选）。

### 3. 只推 origin，再提合并请求
- 推送到**个人仓**：`git push origin <分支>`
- 再从个人仓向团队仓发起**合并请求（PR/MR）**。
> 铁律：**origin = 你的（可读可写）｜ upstream = 团队的（只读）**；拉可从 upstream 拉，推只能往 origin 推。绝不直接 `git push upstream`，那会绕过评审。

### 4. 推回个人仓：用 --force-with-lease 同步上游
```bash
git pull --rebase upstream main
git push --force-with-lease origin main
```
> 个人仓该分支是你**独享**时可放心 force；`--force-with-lease` 比裸 `-f` 多一层保险。若个人仓也邀请他人协作，则不能随便 force。

### 5. 评审意见的修改
- 只改最近一次：`git commit --amend`（可加 `--no-edit` 只并改动不改信息）
- 改多个/更早：`git rebase -i HEAD~3`，把要改的提交标 `edit`，逐个暂停改完 `amend` + `git rebase --continue`
> amend / rebase -i 都改写历史 → 推 origin 仍需 `--force-with-lease`。想让评审清楚看到改了什么，也可**追加新 commit** 回应（过程更透明）。

### 6. rebase 中途冲突处理
- 遇冲突暂停，文件出现 `<<<<<<<` / `=======` / `>>>>>>>` 标记；
- 手动去标记 → `git add <文件>` 标记已解决 → `git rebase --continue`（可反复）；
- 控制钮：`--continue` 继续 ｜ `--abort` 整体放弃回退 ｜ `--skip` 跳过当前提交（**丢改动，慎用**）
> 冲突里用 `git add` 标记解决，**不要 `git commit`**。解乱了直接 `--abort` 重来。推荐 rebase 前先 `git stash` 收起未提交改动。

### 7. 收尾：删分支 + 清理
- 切回主干：`git checkout main`
- 删本地分支：`git branch -d <分支>`（rebase 历史被判定未合并时用 `git branch -D`）
- 删个人远程分支：`git push origin --delete <分支>`（平台合并时可勾选「删除源分支」）
- 清理本地过期远程跟踪引用：`git fetch --prune`
> 删完远程分支后本地 `git branch -a` 仍可能显示 `remotes/origin/旧分支`，`fetch --prune` 才清掉。再把个人 fork 的 main 同步到上游最新。

### 8. 辅助：git stash 暂存
- `git stash push -u` 暂存（含未跟踪）
- `git stash push -m "标题"` 暂存并自定义标题（旧 `git stash save` 已弃用）
- `git stash list` 列出 ｜ `git stash pop` 恢复并删 ｜ `git stash apply` 恢复不删 ｜ `git stash drop` 删除

## 注意事项 / 踩坑
- `git pull` 默认 `origin`，fork 模式拉团队更新**必须显式写 `upstream`**；
- 个人仓推送用 `--force-with-lease` 而非裸 `-f`；
- `git commit --amend` 只能改**最近一次**，更早用 `git rebase -i`；
- rebase 冲突用 `git add` 标记解决，**别用 `git commit`**；解乱了 `--abort` 重来；
- `git fetch --prune` 清的是「远程已删、本地还留着的远程跟踪引用」，与「删本地/远程分支」是三件不同的事；
- 主干统一写 `main`（老仓库可能 `master`）；合并请求 GitHub 称 PR、GitLab 称 MR，含义相同。

## 关联
- [[MOC-Development]]
