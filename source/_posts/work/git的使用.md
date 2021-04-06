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

+ .gitignore文件未生效。先把本地缓存删除（改变成未track状态），然后再提交
`git rm -r --cached .`
`git add .`
`git commit -m 'update .gitignore'`

## 删除分支

+ 远端分支
`git push :dev  或者 git push origin --delete 远程分支名称`

+ 本地分支

`git branch -d 分支名称`

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
+ git remote rm origin # 删除远程库

## git commit之后，撤销commmit

+ git reset --soft HEAD^ （仅仅是撤回commit操作，您写的代码仍然保留。）
+ git reset --hard 8dsadad2dsa 这个可以回退到具体的某个版本 
+ HEAD^的意思是上一个版本，也可以写成HEAD\~1。如果你进行了2次commit，想都撤回，可以使用HEAD~2
+ git push -u origin master -f  强制push到远程。origin：远程仓库名  master：分支名称  -f：force，意为强制、强行
+ git reset --hard af42d7ed09 回退到摸个版本

### 参数

+ --mixed（默认）：不删除工作空间改动代码，撤销commit，并且撤销git add .
+ --soft：不删除工作空间改动代码，撤销commit，不撤销git add .
+ --hard：删除工作空间改动代码，撤销commit，撤销git add . 。注意完成这个操作后，就恢复到了上一次的commit状态。

+ git commit --amend commit注释写错了，只是想改一下注释。

## 查看git配置

+ git config --list # 查看
+ git config --edit # 编辑

## 统计每个人的增删行数

+ git log --format='%aN' | sort -u | while read name; do echo -en "$name\t"; git log --author="$name" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -; done

## 取消某次合并

+ git merge --abort #如果Git版本 >= 1.7.4
+ git reset --merge #如果Git版本 >= 1.6.1

## 将本地所有分支与远程保持同步

+ git fetch --all

## 命名规范

git 分支分为集成分支、功能分支和修复分支，分别命名为 develop、feature 和 hotfix，均为单数。不可使用 features、future、hotfixes、hotfixs 等错误名称。

1. git主分支(master)。它是自动建立，用于发布重大版本更新
2. git开发主分支(develop)。日常开发在此分支上进行
3. git临时性分支：用于应对一些特定目的的版本开发(验证OK后，应该删除此分支)，主要有： 　

+ 功能（feature）分支：它是为了开发某种特定功能，从Develop分支上面分出来的。开发完成后，要再并入Develop。可以采用feature-的形式命名。
+ 预发布（release）分支：指发布正式版本之前（即合并到Master分支之前），我们可能需要有一个预发布的版本进行测试。预发布分支是从Develop分支上面分出来的，预发布结束以后，必须合并进Develop和Master分支。它的命名，可以采用release-的形式
+ 修补bug（hotfix）分支：软件正式发布以后，难免会出现bug。这时就需要创建一个分支，进行bug修补。修补bug分支是从Master分支上面分出来的。修补结束以后，再合并进Master和Develop分支。它的命名，可以采用hotfix-***的形式。

一个分支尽量开发一个功能模块，不要多个功能模块在一个分支上开发。 feature 分支在申请合并之前，最好是先 pull 一下 develop 主分支下来，看一下有没有冲突，如果有就先解决冲突后再申请合并。

## 关闭忽略大小写

`git config core.ignorecase false`
