---
title: PHP-strlen与mb_strlen
date: 2020-06-22 19:11:39
cover: /blog/img/php.jpg
tags:
  - PHP
  - strlen
categories:
  - PHP
---
在PHP中，strlen与mb_strlen是求字符串长度的函数
PHP内置的字符串长度函数strlen无法正确处理中文字符串，它得到的只是字符串所占的字节数。对于GB2312的中文编码，strlen得到的值是汉字个数的2倍，而对于UTF-8编码的中文，就是3倍（在 UTF-8编码下，一个汉字占3个字节）。

采用mb_strlen函数可以较好地解决这个问题。mb_strlen的用法和strlen类似，只不过它有第二个可选参数用于指定字符编码。例如得到UTF-8的字符串str长度，可以用mbstrlen(str长度，可以用mbstrlen(str,‘UTF-8’)。如果省略第二个参数，则会使用PHP的内部编码。内部编码可以通过 mb_internal_encoding()函数得到。

需要注意的是，mb_strlen并不是PHP核心函数，使用前需要确保在php.ini中加载了php_mbstring.dll，即确保“extension=php_mbstring.dll”这一行存在并且没有被注释掉，否则会出现未定义函 数的问题。

-----------------------------------------------------------------------

在strlen计算中，对待一个UTF8的中文字符，处理为3个字节长度，所以为3+1+2+1+9=16个

当mb_strlen的内码选择为UTF-8的时候，则会将中文字符当成一个字符,所以为3+1+2+1+3=10;

当mb_strlen的内码选择为gbk的时候，一个中文字符当成1.5个字符来处理来处理,最后就是:3+1+2+1+4.5=11.5

函数：mb_internal_encoding()会得到当前PHP使用的内部编码

strlen,得到的是字符串所占的字节数，所以在查看一个字符串的长度的时候，strlen并不能得到我们需要的真实值

mb_strlen 函数可以很好的处理这一点

 

注意：mb_strlen函数并不是php的核心函数，只是PHP的一个扩展函数，使用之前要判断是否加在的mbstring扩展模块，在Php.ini文件中可以查看相关配置


> strlen结果为什么是4
strlen在遇到第一个\0时结束，后面的字符无视。