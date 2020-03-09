---
title: git的使用
date: 2019-10-29 22:39:46
tags: git
---

git 的常见使用。分支切换、
<!-- more -->
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

+ .gitignore文件未生效（因为后添加的忽略文件需要更新缓存）
`git config --global credential.helper store`

## 删除远端分支

`git push :dev`

## 提交代码时，忽略某一个文件不提交

`git update-index --assume-unchanged src/main/resource/config.propertites`

## 储存

### 储存文件

储藏会处理工作目录的脏的状态——即跟踪文件的修改与暂存的改动——然后将未完成的修改保存到一个栈上，而你可以在任何时候重新应用这些改动。

`git stash`

### 储藏应用

储藏最新的应用
`git stash apply`

储藏应用指定
`git stash apply stash@{2}`

### 删除储藏

`git stash drop stash@{0}`

应用储藏然后立即从栈上扔掉它
`git stash pop`

## git push

推送：git push 远端服务器名 本地分支 远端分支(refs/for/->加上可避免上传到服务器的merge)
`git push origin master:refs/for/master`
远端、分支位移的话,上面可简化为
`git push`

删除远端分支master
`git push origin master`

## tag

### 查看

+ git tag  # 查看列表
+ git tag -l seachName  # 搜索查看列表
+ git show tagName  # 查看tag的详细信息

### 新建

+ git tag newTAg  # 新建tag
+ git tag -a tagName -m "my tag"  # 给tag加备注
+ git tag -a v1.2 9fceb02 -m "my tag"  # 给某个commmit加tag、备注
+ git push origin v1.0  # 推送到远端
+ git checkout v1.0  # 切换标签

### 删除

+ git tag -d v1.0  # 删除tag
+ git push origin :refs/tags/v0.1.2  # 删除远端分支

## 远程库切换

+ git remote add xxx url  # 添加xxx远程库
+ git remote set-url xxx url  # 修改远程库

## git commit之后，撤销commmit

+ git reset --soft HEAD^ （仅仅是撤回commit操作，您写的代码仍然保留。）
+ HEAD^的意思是上一个版本，也可以写成HEAD\~1。如果你进行了2次commit，想都撤回，可以使用HEAD~2

### 参数

+ --mixed（默认）：不删除工作空间改动代码，撤销commit，并且撤销git add .
+ --soft：不删除工作空间改动代码，撤销commit，不撤销git add . 
+ --hard：删除工作空间改动代码，撤销commit，撤销git add . 。注意完成这个操作后，就恢复到了上一次的commit状态。

+ git commit --amend commit注释写错了，只是想改一下注释。
