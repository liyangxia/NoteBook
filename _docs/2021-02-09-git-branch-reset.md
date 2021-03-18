---
layout: post
title: Git修改本地和远程默认分支
---

今天在push代码的时候，习惯性的敲了master分支，但是报错：

```bach
error: src refspec master does not match any.
error: failed to push some refs to ‘***’
```

在没了解到”Black Lives Matter”运动的前提下，这个报错让我有点费解，后查阅后发现是改了默认的分支，从master到main，所以push的时候才会提示这个错误

查看当前的分支：

```bash
git branch -a
```

查看当前的分支引用

```bash
git show-ref
```

通过这两个命令已经确定当前新建的仓库确实是main分支，而不再是以前的master分支，因为是第一次提交，更换到main分支提交就行

```bash
git push -u origin main
```

对于之前已经提交过，而且是master的分支，可以进行修改

**Setup1**: 修改默认master分支的名字为main

```bash
git branch -m master main
```

**Setup2**: 修改分支引用HEAD到main

```bash
git symbolic-ref refs/remotes/origin/HEAD refs/remotes/origin/main
```

**Setup3**: push新的main分支

```bash
git push origin main
```

**Setup4**: Github repository setting 页面修改默认分支到main

**Setup5**: 删除远端的master分支

```bash
git push —delete origin master
```

至此分支的修改就完成了，详细的情况Github写了一个文档进行说明：https://github.com/github/renaming
