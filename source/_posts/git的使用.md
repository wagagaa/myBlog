---
title: git的使用
date: 2019-10-29 22:39:46
tags: git
---

## 分支切换代码

查看目录所在分支
`git branch -a`

创建新分支
`git branch  b1`

切换分支
`git checkout b1`

创建并切换到新分支(==上面的两步合集)
`git checkout -b b1`

## 比较文件(整体或者是单个文件)

`git diff <localbranch> <remote/branch>`

## 合并分支

把dev分支的工作成果合并到master分支上(当前在master)
`git merge dev`

## 恢复修改的文件

`git checkout -- aaa.txt # aaa.txt为文件名`

## 避免频繁设置目录

`git config --global credential.helper store`

## 删除远端分支

`git push :dev`