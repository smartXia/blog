---
title: git 使用 connot stat
date: 2021-05-17 14:12:35
tags:
cover:
categories:
  - git
---

1.记一次使用bug
![](http://s2-cdn.oneitfarm.com/c71c1444ef7b472fbad9c71ba803fa95.png)
git error: cannot stat 
原因是因为，在某个编辑器打开了master分支的一个文件，然后切换到feat分支，文件并不消失，拉取时候出现问题，

解决办法：关掉编辑器