---
date: '2026-03-06T10:19:31+08:00'
lastmod: '2026-03-06T10:19:31+08:00'
draft: false
title: Git Worktree：多分支并行开发的利器
slug: git-worktree-parallel-dev
description: Git 的分支功能很强大，但实际开发中经常遇到一个问题：需要在同一时间处理多个任务，比如主分支有紧急 Bug 要修，同时又想在一个新分支上做重构。这时传统的
  `git checkout` 会让你陷入 stash、切换、再 pop 的循环，环境重建也很麻烦。
tags:
- 通用
- git
- 掘金
- 公众号
categories:
- 工具
comment: false
toc: false
---

> Git Worktree：多分支并行开发的利器



Git 的分支功能很强大，但实际开发中经常遇到一个问题：需要在同一时间处理多个任务，比如主分支有紧急 Bug 要修，同时又想在一个新分支上做重构。这时传统的 `git checkout` 会让你陷入 stash、切换、再 pop 的循环，环境重建也很麻烦。 [

Git 2.6 引入了 `git worktree` 命令，它允许一个 Git 仓库对应多个工作目录，每个目录可以检出不同的分支，实现真正的并行开发。 

## 什么是 Worktree？

![illustration_1](illustration_1.png)

简单说，Worktree 就是一个仓库的多个「分店」。它们共享同一个 `.git` 目录（包含所有分支历史、提交记录），但拥有独立的文件系统和工作环境。

```
仓库.git（共享）
├── 主工作树（main 分支）
├── ../feature/（feature 分支）
└── ../bugfix/（bugfix 分支）
```

这样，你可以在不同目录同时编辑不同分支，互不干扰。

## 核心区别

| 特性 | 普通分支 | Worktree |
|------|----------|----------|
| 工作目录 | 单一目录，切换分支需 checkout | 多个独立目录同时存在 |
| 环境隔离 | 切换分支需重建 node_modules 等 | 各树独立环境 |
| 并行开发 | 需要 stash 保存变更 | 无需 stash，直接并行 |
| 存储开销 | 只占指针空间 | 共享 .git，只复制工作文件 |

这些差异让 worktree 特别适合一台机器上并行开发多个功能。 

## 基本使用

### 1. 创建 Worktree

在主仓库执行：

```bash
# 基于现有分支创建
git worktree add ../feature-branch feature-branch

# 创建新分支并检出
git worktree add ../new-feature -b new-feature
```

创建后，你得到一个新目录 `../feature-branch/`，里面是 `feature-branch` 分支的完整工作树，主目录保持不变。

### 2. 查看和管理

```bash
# 列出所有 worktree
git worktree list

# 删除 worktree（需先 commit 或 stash）
git worktree remove ../feature-branch

# 清理无效记录
git worktree prune
```

这些命令基本覆盖了 worktree 的日常管理需求。

### 3. 日常操作

所有 worktree 共享仓库状态：
- 在任意树 `git commit`、`git push`，其他树立即看到更新。
- `git pull`、`git fetch` 在任一树执行，全局生效。
- 分支删除 `git branch -d`，所有树同步反映。

你可以把它理解为：一个 .git，挂了多个「工作副本」。

## 典型场景：重构 + 紧急修复

![illustration_2](illustration_2.png)

这是 worktree 最实用的场景。

1. 主树在 master，创建重构树：
   ```bash
   git worktree add ../refactor refactor-branch
   ```

2. 重构树内开发、测试，拥有独立的 `node_modules`、构建目录等环境。

3. 主树遇紧急 Bug 时，直接在 master 修复、commit，无需打断重构，更不需要 stash / pop。

4. 重构完成后合并回主分支：
   ```bash
   # 重构树内
   git push origin refactor-branch
   
   # 主树执行合并
   git checkout master
   git merge refactor-branch
   git push origin master
   ```

5. 重构分支合并完毕后，可以删除对应 worktree：
   ```bash
   git worktree remove ../refactor
   ```

### 未提交改动也完全没问题

一个常见疑问是：如果我在 worktree 1 有未提交改动，还能切到 worktree 2 吗？

答案是：可以，随便切。

- Worktree 是多个独立工作目录，每个目录的未提交改动只存在于该目录下。
- 你可以在 worktree 1 改到一半不提交，直接 `cd` 到 worktree 2 继续干别的事。
- 不需要 stash，不需要 commit，Git 也不会阻止你进入其他 worktree。

示例：

```bash
# worktree 1
cd ./main-worktree
echo "temp" >> README.md
git status  # 有未提交改动

# 直接切到 worktree 2
cd ../feature-worktree
git status  # 这里是另一套状态，互不干扰
```

对比传统方式，你就能感受到差异：

```
传统：改到一半 → stash → checkout 其他分支 → 修完 → 再 pop（可能冲突）
Worktree：每个目录一套改动，来回切换只需要 cd
```

这就是 worktree 真正的多任务能力：**未提交状态天然隔离，不用通过 stash 做「上下文切换」。**

## 多用户协作完全兼容

有些人会担心：我在本地用 worktree，会不会影响别人切换到同一个分支？

答案是：不会。Worktree 是纯本地特性，对远程仓库和其他开发者完全透明。

- 用户1 在自己机器上：
  ```bash
  git worktree add ../task-001 task-001
  ```
  这只会让「用户1这台机器上的这个仓库」认为 `task-001` 已在某个 worktree 检出。

- 用户2 在另一台机器上，可以照常：
  ```bash
  git checkout task-001
  ```

或再建自己的 worktree，完全不受影响。

可以总结为：

| 用户场景 | 操作 | 结果 |
|----------|------|------|
| 用户1（用 worktree） | `git worktree add ../task-001 task-001` | 只锁定本地这个仓库的该分支 |
| 用户2（普通 clone） | `git checkout task-001` | 正常切换，无任何冲突 |
| 用户2（也用 worktree） | `git worktree add ../task-001 task-001` | 正常创建自己的 worktree |

协作流程不变：
1. 用户1 在自己的 worktree 中开发 `task-001`，`git commit && git push`。
2. 用户2 `git fetch && git checkout task-001 && git pull`，继续开发。
3. 所有人对 `task-001` 的 commit 最终都通过远程仓库合并，仍然走普通的 Git 流程（PR/MR、review 等）。

换句话说：**worktree 只提升个人效率，不改变团队协作模型。**

## 为什么推荐？

- 节省时间：无需频繁 stash / checkout / pop，也不用重建环境。
- 降低风险：每个工作树是独立沙盒，破坏了也只影响本目录。
- 并行开发：一台电脑上自然地「多窗口开发」，重构、修 Bug、写新需求互不打扰。 
- 节省空间：共享 `.git`，只复制实际工作文件。
- 团队友好：对远程和他人透明，不改协作流程。 

对比传统方式：

```
传统：stash → checkout master → 修 bug → checkout feature → pop → 解决可能的冲突 → 继续重构
Worktree：主树修 bug，重构树继续重构，最后 merge 一下
```

## 注意事项

- 同一个分支不能在同一仓库的多个 worktree 中同时检出（这是 Git 的保护机制）。
- 删除 worktree 前，记得 commit 或 stash，否则该目录的未提交改动会丢失。
- bare 仓库不支持 worktree（没有工作目录）。
- 结合 VS Code / JetBrains 多窗口使用体验极佳，可以一个窗口对应一个 worktree。

## 实用 Git Alias 配置

如果你已经习惯了像 `co = checkout` 这样的简写，可以顺手为 worktree 配一套 alias，进一步提高效率，以下是我的 alias 配置，可以参考一下。

在 `~/.gitconfig` 的 `[alias]` 部分添加：

```ini
[alias]
  co = checkout
  ci = commit
  st = status
  br = branch
  ss = stash
  sp = stash pop
  # Worktree 相关
  wta = worktree add
  wtl = worktree list
  wtr = worktree remove
  wtp = worktree prune
```

这样，你的日常命令会变得非常简洁：

```bash
git wta ../refactor refactor-branch    # 创建重构 worktree
git wtl                                # 查看所有 worktree
git wtr ../refactor                    # 删除重构 worktree
git wtp                                # 清理无效记录
```

结合前面的多场景使用，这几个 alias 足够覆盖大部分日常需求。

## 总结

Worktree 是 Git 隐藏的杀手锏，特别适合单机多任务开发。它让「一台电脑同时开发多个功能」从梦想变成现实，极大提升了开发体验和节奏感。

下次遇到「主分支有急活，但手头还有重构没完成」的场景，不妨试试：

```bash
git wta ../urgent-fix urgent-branch
```

你会发现，开发原来可以这么优雅。
