---
title: HEXO快捷方式
date: 2020-06-09 14:23:15
tags:
cover: https://d33wubrfki0l68.cloudfront.net/5997a40576f3beca7bbbd86fe79a795e9d520d8e/87f88/themes/screenshots/landscape.png
categories:
  - 建站
---



https://www.jianshu.com/p/1c888a6b8297?utm_source=oschina-app

### 新编写BLOG
hexo new [layout] <title>
### push到github：
hexo deploy


hexo clean

hexo deploy

### 每次都要执行 hexo clean 和 hexo deploy，不如写个新的脚本
```
// package.json
"dev": "hexo s",
"build": "hexo clean & hexo deploy"
```
npm run build