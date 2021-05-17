---
title: Git-代码提交规范
date: 2020-09-02 14:13:27
tags:
cover: img/git.jpeg
categories:
  - git
---
Git commit message 是Git提交的必要信息，message的信息完整度也反映了工程师对于代码提交的重视程度，不清晰的git message信息甚至会让工程师完全回忆不起自己当初做了什么调整，导致后续代码维护成本特别大。因此为了提高线上代码库的管理程度，特此制定GIT commit message规范。

一、commit message格式
1、Type(必须)
用于说明 git commit 的类别，只允许使用下面的标识。
feat：新功能（feature）。
fix/to：修复bug，可以是QA发现的BUG，也可以是研发自己发现的BUG。
fix：产⽣diff并自动修复此问题。适合于一次提交直接修复问题
to：只产⽣diff不自动修复此问题。适合于多次提交。最终修复问题提交时使用fix
docs：文档（documentation）。
style：格式（不影响代码运行的变动）。
refactor：重构（即不是新增功能，也不是修改bug的代码变动）。
perf：优化相关，比如提升性能、体验。
test：增加测试。
chore：构建过程或辅助工具的变动。
revert：回滚到上一个版本。
merge：代码合并。
sync：同步主线或分⽀的Bug

2、scope(可选)
scope用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。
例如在Angular，可以是location，browser，compile，compile，rootScope， ngHref，ngClick，ngView等。
如果你的修改影响了不止一个scope，你可以使用*代替。

3、subject(必须)
subject是commit目的的简短描述，不超过50个字符。
1.建议使用中文。
2.结尾不加句号或其他标点符号。
根据以上规范 git commit message 将是如下的格式：
fix(DAO): 用户查询缺少username属性
feat(Controller): 用户查询接口开发

二、规范的好处
我们这样规范git commit到底有哪些好处呢？
1.便于程序员对提交历史进行追溯，了解发⽣了什么情况。
2.一旦约束了commit message，意味着我们将慎重的进行每一次提交，不能再一股脑的把各种各样的改动都放在一个git commit里面，这样一来整个代码改动的历史也将更加清晰。
3.格式化的commit message才可以用于自动化输出Change log。

三、标准执行监管
为了更好的执行标准，公司针对git提交会进行相关监管功能的研发，当工程师提了不合规的commit，会收到相关的邮件警告。