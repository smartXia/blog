---
title: 从正则表达式的iUs说说模式修正符
cover: img/php.jpeg
categories:
  - PHP
date: 2021-05-25 19:06:09
tags:
---

本想做个简单的采集程序，发现被抓页面代码的规律后发现抓下来的内容没有放到一个数组中，而是放在一个元素中，无奈找遍资料发现在正则表达式后加上”/iUs”后竟然可以了。
hexo
网上关于iUs的说明多数都是抄袭的，没有做过多的解释，对于一个小学毕业证是买来的人来说是在是不好理解。不过幸亏Google让我找到答案。

“iUs” 在这里叫“模式修正符”。模式修正符其实就是几个字母，可以一次使用一个也可以一次使用多个，每一个都具有一定的意义，模式修正符是对正则表达式的扩展；“/模式修正符”，其中正斜线“/”为边界符。下表列出来有那些模式修正符：


模式修正符	说明
i	表示在和模式进行匹配进不区分大小写
m	将模式视为多行，使用^和$表示任何一行都可以以正则表达式开始或结束
s	如果没有使用这个模式修正符号，元字符中的”.”默认不能表示换行符号,将字符串视为单行
x	表示模式中的空白忽略不计
e	正则表达式必须使用在preg_replace替换字符串的函数中时才可以使用(讲这个函数时再说)
A	以模式字符串开头，相当于元字符^
Z	以模式字符串结尾，相当于元字符$
U	正则表达式的特点：就是比较“贪婪”，使用该模式修正符可以取消贪婪模式
### 1，模式修正符m。

```
<?php
$pattern = ‘/^abc/m’;
$string = ‘bcd
abc
cba’;
if (preg_match($pattern, $string, $arr)) {
echo “正则表达式<b>{$pattern}</b>和字符串<b>{$string}</b>匹配成功<br>”;
print_r($arr);
} else {
echo “<font color=’red’>正则表达式{$pattern}和字符串{$string}匹配失败</font>”;
}
?>
```

匹配结果是成功的。注意：我们在使用模式修正符m的时候，将匹配字符串看成是多行而不是默认的单行，所以任何一行只要是以abc开头，就匹配成功。但是，如果能匹配的行前面有空格的话，就不能匹配了!除非修改正则表达式的匹配模式。

### 2，模式修正符s。

``` 
<?php
$pattern = ‘/a.*c/s’;
$string = ‘adsadsa
c’;
if (preg_match($pattern, $string, $arr)) {
echo “正则表达式<b>{$pattern}</b>和字符串<b>{$string}</b>匹配成功<br>”;
print_r($arr);
} else {
echo “<font color=’red’>正则表达式{$pattern}和字符串{$string}匹配失败</font>”;
}
?>
```

这次的匹配记过也是成功的。如果你将上例中的模式修正符s去掉的话，匹配就会失败。因为模式修正符s将匹配字符串看作是单行的，所以这个时候，元字符中的”.”就可以表示换行符号了。

### 3，模式修正符x。

```
<?php
$pattern = ‘/a c/x’;
$string = ‘a c’;
if (preg_match($pattern, $string, $arr)) {
echo “正则表达式<b>{$pattern}</b>和字符串<b>{$string}</b>匹配成功<br>”;
print_r($arr);
} else {
echo “<font color=’red’>正则表达式{$pattern}和字符串{$string}匹配失败</font>”;
}
?>
```

这次的匹配结果是失败的。因为我们使用模式修正符x取消了模式中的空格。注意：我们无法使用模式修正符取消\s表示的空白。

### 4，模式修正符A。

```
<?php
$pattern = ‘/ac/A’;
$string = ‘acahgyghvbm’;
if (preg_match($pattern, $string, $arr)) {
echo “正则表达式<b>{$pattern}</b>和字符串<b>{$string}</b>匹配成功<br>”;
print_r($arr);
} else {
echo “<font color=’red’>正则表达式{$pattern}和字符串{$string}匹配失败</font>”;
}
?>

```
正则表达式表示的含义是匹配以ac开头的字符串，结果成功。

模式修正符Z表示的是以字符串结尾的匹配，和A的用法是一样的，我们不再进行演示。

### 5，模式修正符U。

这个模式修正符是十分重要的!在正则表达式中，其本身是“贪婪”的。那什么是贪婪模式呢?贪婪模式的意思就是说，正则表达式默认会在查找到第一个匹配后，继续尝试后面的匹配，如果能找到匹配，则匹配最大的范围字符串。但有的时候这并不是我们想要的结果，所以我们需要取消贪婪模式。

我们还是先看一个贪婪模式的例子：

```
<?php
$pattern = ‘/<b>.*<\/b>/’;
$string = ‘<b>welcome</b> <b>to</b> <b>phpfuns</b>’;
if (preg_match($pattern, $string, $arr)) {
echo “正则表达式<b>{$pattern}</b>和字符串<b>{$string}</b>匹配成功<br>”;
print_r($arr);
} else {
echo “<font color=’red’>正则表达式{$pattern}和字符串{$string}匹配失败</font>”;
}
?>
```


这个实例的本意是匹配welcome，但是结果却匹配了welcome to phpfuns整个字符串(注意我们的字符串’welcome to phpfuns’，其开头和结尾正好构成了正则表达式的模式匹配，所以匹配成功)，这就是正则表达式的贪婪模式。当然，这不是我们要的结果。

取消贪婪模式
我们可以使用模式修正符U和元字符?两种方式取消正则表达式的贪婪模式。

模式修正符U取消贪婪模式

```
<?php
$pattern = ‘/<b>.*<\/b>/U’;
$string = ‘<b>welcome</b> <b>to</b> <b>phpfuns</b>’;
if (preg_match($pattern, $string, $arr)) {
echo “正则表达式<b>{$pattern}</b>和字符串<b>{$string}</b>匹配成功<br>”;
print_r($arr);
} else {
echo “<font color=’red’>正则表达式{$pattern}和字符串{$string}匹配失败</font>”;
}
?>
```

元字符?取消贪婪模式

```
<?php
$pattern = ‘/<b>.*?<\/b>/’;
$string = ‘<b>welcome</b> <b>to</b> <b>phpfuns</b>dsadsadas’;
if (preg_match($pattern, $string, $arr)) {
echo “正则表达式<b>{$pattern}</b>和字符串<b>{$string}</b>匹配成功<br>”;
print_r($arr);
} else {
echo “<font color=’red’>正则表达式{$pattern}和字符串{$string}匹配失败</font>”;
}
?>
```

注意元字符的位置，我们必须在“”之前结束贪婪模式，才能达到我们的目的

[转载自26点的博客]: http://www.iamlintao.com/2275.html

