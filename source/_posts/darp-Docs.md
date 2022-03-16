---
title: darp Docs
categories:
  - dapr
date: 2022-03-16 14:59:18
tags:
cover: 
---
dapr 文档贡献
dapr 文档贡献规则：https://docs.dapr.io/zh-hans/contributing/contributing-docs/

dapr 文档网站使用hugo 开发工具：

Windows：安装流程
安装scoope
powerSheel ：执行命令：
```
set-executionpolicy remotesigned -scope currentuser

iex (new-object net.webclient).downloadstring('https://get.scoop.sh')

执行scoope help 查看是否安装正常

执行   scoop install hugo
      scoop install hugo-extended
这两部即可 完成对其安装

文档：https://gohugo.io/getting-started/installing/

```

### 1.执行doc仓库下载和安装依赖

仓库：
```
https://github.com.cnpmjs.org/dapr/docs.git

执行：
git submodule update --init --recursive

这步可能遇到下载不下来情况，可以按照此步骤进行操作

1.执行 git submodule update --init
2.去.gitmodules文件 进行编辑将所有的https://github.com  后缀加上 cnpmjs.org
(这个原理可以参考:谷歌插件---GitHub加速1.3.5)
3.然后利用git submodule sync更新子项目对应的url
4.git submodule update --init --recursive，最后执行

//s2-cdn.oneitfarm.com/6d3518411d074f9eae604f77da39da83.png
```

![](//s2-cdn.oneitfarm.com/a23f2220daff4375ba97522c7edc552c.png)

错误1.

![image-20220316151455745](//s2-cdn.oneitfarm.com/c036d24abc364f00be0870faec874818.png)

解决方案：

```
使用git代理：
git config *--global https.proxy*
执行命令后取消代理
git config --global --unset https.proxy
```



错误2：

![//s2-cdn.oneitfarm.com/b933985e39524c7ea6763610f552ecd7.png](//s2-cdn.oneitfarm.com/b933985e39524c7ea6763610f552ecd7.png)

解决方案：

```
这个是因为没有执行git module的 下载
```



### 2.安装依赖

此项目使用的还是npm 

````
npm install （很慢） 
````



