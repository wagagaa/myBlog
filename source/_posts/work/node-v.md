---
title: node版本管理的正确打开方式(mac)
date: 2021-03-08 10:10:08
---

mac切换node版本的步骤。切换后只限当前使用，重启还原为最后一次安装的node版本。

本文主要是针对安装了node的用户如何对node进行升级或者安装指定版本；
没有安装node的可以前往官网安装 。

<!-- more -->

## 安装方法

1.查看node版本，没安装的请先安装；

`$ node -v`

2.清除node缓存；

`$ sudo npm cache clean -f`

3.安装node版本管理工具'n';

`$ sudo npm install n -g`

4.使用版本管理工具安装指定node或者升级到最新node版本；

`$ sudo n stable （安装node最新版本）`

`$ sudo n 8.9.4 （安装node指定版本8.9.4）`

5.使用node -v查看node版本，如果版本号改变为你想要的则升级成功。

## 若版本号未改变则还需配置node环境变量

1.查看通过n安装的node的位置；

`$ which node (如：/usr/local/n/versions/node/6.12.3）`

2.cd进入/usr/local/n/versions/node/ 你应该能看到你刚通过n安装的node版本这里如：8.9.4；编辑/etc/profile;

`$ vim /etc/profile`

3.将node安装的路径（这里为：/usr/local/n/versions/node/8.9.4）添加到文件末尾；

```*
#set node path

export NODE_HOME=/usr/local/n/versions/node/8.9.4

export PATH=$NODE_HOME/bin:$PATH
```

4.wq退出保存文件，编译/etc/profile;

$ `source /etc/profile`

5.再次使用node -v查看node版本，不出意外版本号应该变为你想要的。
