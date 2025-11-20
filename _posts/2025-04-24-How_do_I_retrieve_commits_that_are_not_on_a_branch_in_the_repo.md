---
title: 在repo非分支上的提交怎么找回？
date: 2025-04-11 00:56:51 +0800
categories: [Blogging]
tags: [git]
mermaid: true
---
repo sync后git是没有分支的，使用git br命令可以查看出来，这时git push提交了代码，repo upload时发现没分支提交不了。
这时，repo start创建了一个分支，原来非分支上的代码就找不到了，用git br是看不到的

## Git非分支上的提交，怎么找回？

**1. 查看提交历史**

首先，你可以使用 git reflog 命令查看所有的引用日志，包括那些在分离头指针状态下的提交：

```bash
git reflog
```

这将列出你最近的所有操作，包括提交、检出等。你会看到类似于以下的输出：

```bash
abc1234 HEAD@{0}: commit: Your commit message
def5678 HEAD@{1}: checkout: moving from master to 
...
```

**2. 找到你的提交**

在 git reflog 的输出中，找到你想要恢复的提交的哈希值（例如 abc1234）。

**3. 创建一个新分支**

一旦你找到了提交的哈希值，你可以创建一个新的分支来保存这个提交：

```bash
git checkout -b new-branch-name abc1234
```

这将创建一个名为 new-branch-name 的新分支，并将 HEAD 指向你想要恢复的提交。

**4. 确认恢复**

你可以使用 git log 命令确认你的提交已经被恢复到新分支上：

```bash
git log
```

**5. 继续工作**

现在你可以在新分支上继续工作，或者将这些更改合并到其他分支中。

## 要将 0201 分支的提交复制到 0202 分支上：

**1. 切换到 0202 分支**


首先，确保你在 0202 分支上：

```bash
git checkout 0202
```

**2. 确定要复制的提交**


根据你提供的 git reflog 输出，0201 分支的最新提交是 6344f1b7d。如果你想复制这个提交，可以使用它的哈希值。

**3. 使用 git cherry-pick 复制提交**


使用 git cherry-pick 命令将 6344f1b7d 提交复制到 0202 分支：

```bash
git cherry-pick 6344f1b7d
```

**4. 处理冲突（如果有）**


如果在复制过程中出现冲突，Git 会提示你解决冲突。你需要手动解决冲突，然后使用以下命令标记冲突已解决：

```bash
git add `<conflicted-files>`
git cherry-pick --continue
```

**5. 确认提交**


完成后，你可以使用 git log 命令确认提交已经成功复制到 0202 分支：

```bash
git log
```

**6. 继续工作**


现在你可以在 0202 分支上继续工作，或者进行其他操作。
