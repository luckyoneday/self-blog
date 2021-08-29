---
title: "Git 操作 tips"
date: 2020-08-30T20:49:58+08:00
lastmod: 2020-08-30T20:49:58+08:00
draft: false
toc: true
keywords: ["daily"]
description: "Git 操作的一些命令"
tags: ["git"]
author: "youting"
---

## 下载大项目时超时

具体报错：

```bash
remote: Counting objects: 66352, done.
remote: Compressing objects: 100% (10417/10417), done.
error: RPC failed; curl 18 transfer closed with outstanding read data remaining
fatal: The remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
```

解决方法：

```bash
$ git clone http://github.com/large-repository --depth 1
$ cd large-repository
$ git fetch --unshallow
```

[参考链接](https://stackoverflow.com/questions/38618885/error-rpc-failed-curl-transfer-closed-with-outstanding-read-data-remaining)

## 解决冲突无法 commit

具体报错：

```bash
fatal: cannot do a partial commit during a merge.
```

解决方案：

```bash
git commit -i xxx.js
```

## git rebase

### 合并所在分支的多个 commit

```bash
# 合并之前的三条提交
git rebase -i HEAD~3

# 或者
git rebase -i 三条提交的前一个commitId
```

执行完上述命令后会出现一个交互性的 vim 窗口，通过提示将多次的 commit 信息保留为只剩一个（进行 `squash`），操作完后按 esc 随后输入 `:wq` 即可保存并退出。然后运行：

```bash
git add .
git rebase --continue
```

如果想放弃这次 rebase 可以执行：`git rebase --abort`。

### 合并别的分支

```bash
git rebase 目标分支
```

## 参考资料

- [参考链接](https://stackoverflow.com/questions/5827944/git-error-on-commit-after-merge-fatal-cannot-do-a-partial-commit-during-a-mer)
