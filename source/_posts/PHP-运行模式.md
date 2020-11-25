---
title: PHP-运行模式cli fastcgi
date: 2020-08-28 11:31:08
tags:
- cgi 
- fastcgi 
- cli
categories:
  - PHP
cover: /blog/img/php.jpg
---

　　**1.cgi全称“通用网关接口”(Common Gateway Interface)， 它可以让一个客户端，从浏览器向Web服务器上的程序请求数据，是客户端和程序之间传输数据的一种标准，另外CGI独立于任何语言，所以可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量。如php,perl,tcl等。CGI针对每个用户请求都要开单独的子进程去维护，执行结束处理掉这个进程。典型的fork-and-execute方式**

　　**2.fastcgi，根据1中cgi的特性，可以知道消耗很大，如果很多用户请求，则会申请很多个子进程。。这时候出现了FastCGI。FastCGI 像是一个常驻 (long-live) 型的 CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去 fork 一次 (这是 CGI 最为人诟病的 fork-and-execute 模式)。这个是当下用的最多的了。。linux+nginx+php+mysql**

　FastCGI的工作原理是：

*(1)、Web Server启动时载入FastCGI进程管理器【PHP的FastCGI进程管理器是PHP-FPM(php-FastCGI Process Manager)】（nginx);*
*(2)、FastCGI进程管理器自身初始化，启动多个CGI解释器进程 (在任务管理器中可见多个php-cgi.exe)并等待来自WebServer的连接。*
*(3)、当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。*
*(4)、FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器（运行在 WebServer中）的下一个连接。在正常的CGI模式中，php-cgi或 .exe在此便退出了。*
*在CGI模式中，你可以想象 CGI通常有多慢。每一个Web请求PHP都必须重新解析php.ini、重新载入全部dll扩展并重初始化全部数据结构。使用FastCGI，所有这些都只在进程启动时发生一次。一个额外的好处是，持续数据库连接(Persistent database connection)可以工作。*
　　*3.module形式一般用于apache，模块模式是以mod_php5模块的形式集成，此时mod_php5模块的作用是接收Apache传递过来的PHP文件请求，并处理这些请求，然后将处理后的结果返回给Apache。*
　　*4.cli模式。命令行执行php，一般不用。我们在linux下经常使用 "php -m"查找PHP安装了那些扩展就是PHP命令行运行模式；也可以直接命令行执行php xxx.php*



#### 1.php一共分为五大运行模式：包括ducgi 、fast-cgi、cli、isapi、apache 模块的 DLLCGI

  关于PHP目前比较常见的五大运行模式：

1）CGI（通用网关接口/ Common Gateway Interface）
2）FastCGI（常驻型CGI / Long-Live CGI）
3）CLI（命令行运行 / Command Line Interface）
4）LoadModule（Apache独有）：
在Apache配置文件httpd.conf里，通常加的LoadModule php7_module “D:/…/php71/php7apache2_4.dll”起到的作用就是这个
5）ISAPI（Internet Server Application Program Interface）
IIS独有：
备注：在PHP5.3以后，PHP不再有ISAPI模式，安装后也不再有php5isapi.dll这个文件。要在IIS6上使用高版本PHP，必须安装FastCGI 扩展，然后使IIS6支持FastCGI。  

### 2、php-cli 与php-fpm（fastcgi process manager）
![[**CGI、FastCGI和PHP-FPM关系图解**](https://www.awaimai.com/371.html)](https://img-blog.csdnimg.cn/20190420173441325.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpbnFpYW45OTk=,size_16,color_FFFFFF,t_70)

cli 模式就是常见的命令使用的php命令，其实他也可以提供http请求服务，内置了http服务器
fpm 是一个多进程架构的FastCgi 服务，内置PHP解释器进程常驻后台，自带进程管理支持进程池配置和配置Nginx使用

cli 和fpm 是两个运行方式 