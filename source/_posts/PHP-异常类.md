---
title: PHP-异常类
date: 2020-06-22 11:27:10
cover: /blog/img/php.jpg
tags:
  - PHP
  - 异常
categories:
  - PHP
---
PHP 异常与错误 —— Throwable
Throwable

官方文档地址： http://php.net/manual/en/class.throwable.php

PHP 支持版本： 7

> PHP7 异常与错误处理机制 Exception, Error, Throwable
Throwable 是 PHP 7 中可以用作任何对象抛出声明的基本接口，包括 Expection （异常）和 Error （错误）。

```
Throwable {
    
  /* 抽象方法 */
  abstract public string getMessage ( void ) // 获取抛出的消息内容

  abstract public int getCode ( void ) // 获取抛出的错误代码

  abstract public string getFile ( void ) // 获取产生异常的文件名

  abstract public int getLine ( void ) // 获取相关行号

  abstract public array getTrace ( void ) // 获取追踪信息，返回数组形式

  abstract public string getTraceAsString ( void ) // 获取追踪信息，返回字符串形式

  abstract public Throwable getPrevious ( void ) // 返回上一个 Throwable

  abstract public string __toString ( void ) // 抛出的对象以字符串形式返回，可以用 echo 打印相应结果

}
```
请一定要注意，没有特殊说明：本例 PHP Version < 7
　　说起 PHP 异常处理，大家首先会想到 try-catch，那好，我们先看一段程序吧：有一个 test.php 文件，有一段简单的 PHP 程序，内容如下，然后命令行执行：php test.php

```
1 <?php
2 $num = 0;
3     try {
4         echo 1/$num;
5
6 } catch (Exception $e){
7         echo $e->getMessage();
8 }
9 ?>
```
　　我的问题是：这段程序能正确的捕捉到除 0 的错误信息吗？
　　如果你回答能，那你就把这篇文章看完吧！应该能学点东西。

本文章分 5 个部分介绍我的异常处理的理解：

一、异常与错误的概述

二、ERROR 的级别

三、PHP 异常处理中的黑科技

四、巧妙的捕获错误和异常

五、自定义异常处理和异常嵌套

六、PHP7 中的异常处理

一、异常与错误的概述
　　 PHP 中什么是异常：
　　程序在运行中出现不符合预期的情况，允许发生（你也不想让他出现不正常的情况）但他是一种不正常的情况，按照我们的正常逻辑本不该出的错误，但仍然会出现的错误，属于逻辑和业务流程的错误，而不是编译或者语法上的错误。

PHP 中什么是错误：
　　属于 php 脚本自身的问题，大部分情况是由错误的语法，服务器环境导致，使得编译器无法通过检查，甚至无法运行的情况。warning、notice 都是错误，只是他们的级别不同而已，并且错误是不能被 try-catch 捕获的。

上面的说法是有前提条件的：
　　在 PHP 中，因为在其他语言中就不能这样下结论了，也就是说异常和错误的说法在不同的语言有不同的说法。在 PHP 中任何自身的错误或者是非正常的代码都会当做错误对待，并不会以异常的形式抛出，但是也有一些情况会当做异常和错误同时抛出(据说是，我没有找到合适的例子)。也就是说，你想在数据库连接失败的时候自动捕获异常是行不通的，因为这就不是异常，是错误。但是在 java 中就不一样了，他会把很多和预期不一致的行为当做异常来进行捕获。

PHP 异常处理很鸡肋？
　　在上面的分析中我们可以看出，PHP 并不能主动的抛出异常，但是你可以手动抛出异常，这就很无语了，如果你知道哪里会出问题，你添加 if else 解决不就行了吗，为啥还要手动抛出异常，既然能手动抛出就证明这个不是异常，而是意料之中。以我的理解，这就是 PHP 异常处理鸡肋的地方（不一定对啊）。所以 PHP 的异常机制不是那么的完美，但是使用过框架的同学都知道有这个情况：你在框架中直接写开头那段 php“自动”捕获异常的代码是可以的，这是为什么？看过源码的同学都知道框架中都会涉及三个函数：register_shutdown_function，set_error_handler，set_exception_handler 后面我会重点讲解着三个黑科技，通过这几个函数我们可以实现 PHP 假自动捕获异常和错误。

二、ERROR 的级别
　　只有熟悉错误级别才能对错误捕捉有更好的认识。 ERROR 有不同的错误级别，我之前的一篇文章中有写到：http://www.cnblogs.com/zyf-zhaoyafei/p/3649434.html
　　下面我再总结性的给出这几类错误级别：

```
1 Fatal Error:致命错误（脚本终止运行）
2 E_ERROR // 致命的运行错误，错误无法恢复，暂停执行脚本
3 E_CORE_ERROR // PHP 启动时初始化过程中的致命错误
4 E_COMPILE_ERROR // 编译时致命性错，就像由 Zend 脚本引擎生成了一个 E_ERROR
5 E_USER_ERROR // 自定义错误消息。像用 PHP 函数 trigger_error（错误类型设置为：E_USER_ERROR）
6
7 Parse Error：编译时解析错误，语法错误（脚本终止运行）
8 E_PARSE //编译时的语法解析错误
9
10 Warning Error：警告错误（仅给出提示信息，脚本不终止运行）
11 E_WARNING // 运行时警告 (非致命错误)。
12 E_CORE_WARNING // PHP 初始化启动过程中发生的警告 (非致命错误) 。
13 E_COMPILE_WARNING // 编译警告
14 E_USER_WARNING // 用户产生的警告信息
15
16 Notice Error：通知错误（仅给出通知信息，脚本不终止运行）
17 E_NOTICE // 运行时通知。表示脚本遇到可能会表现为错误的情况.
18 E_USER_NOTICE // 用户产生的通知信息。
```
　　由此可知有 5 类是产生 ERROR 级别的错误，这种错误直接导致 PHP 程序退出。
　　可以定义成：

1 ERROR = E_ERROR | E_CORE_ERROR | E_COMPILE_ERROR | E_USER_ERROR | E_PARSE
三、PHP 异常处理中的黑科技
　　前面提到框架中是可以捕获所有的错误和异常的，之所以能实现应该是使用了黑科技，哈哈！其实也不是什么黑科技，主要是三个重要的函数：

1：set_error_handler()
　　看到这个名字估计就知道什么意思了，这个函数用于捕获错误，设置一个用户自定义的错误处理函数。

```
1 <?php
2 set_error_handler('zyferror');
3 function zyferror($type, $message, $file, $line)
4 {
5 　　 var_dump('<b>set_error_handler: ' . $type . ':' . $message . ' in ' . $file . ' on ' . $line . ' line .</b><br />');
6 }
7 ?>
```
　　当程序出现错误的时候自动调用此方法，不过需要注意一下两点：第一，如果存在该方法，相应的 error_reporting()就不能在使用了。所有的错误都会交给自定义的函数处理。第二，此方法不能处理以下级别的错误：E_ERROR、 E_PARSE、 E_CORE_ERROR、 E_CORE_WARNING、 E_COMPILE_ERROR、 E_COMPILE_WARNING，set_error_handler() 函数所在文件中产生的 E_STRICT，该函数只能捕获系统产生的一些 Warning、Notice 级别的错误。
　　并且他有多种调用的方法：

```
1 <?php
2 // 直接传函数名 NonClassFunction
3 set_error_handler('function_name');
4
5 // 传 class_name && function_name
6 set_error_handler(array('class_name', 'function_name'));
7 ?>
```
　　 2：register_shutdown_function()
　　捕获 PHP 的错误：Fatal Error、Parse Error 等，这个方法是 PHP 脚本执行结束前最后一个调用的函数，比如脚本错误、die()、exit、异常、正常结束都会调用，多么牛逼的一个函数啊！通过这个函数就可以在脚本结束前判断这次执行是否有错误产生，这时就要借助于一个函数：error_get_last()；这个函数可以拿到本次执行产生的所有错误。error_get_last();返回的信息：
　　[type] - 错误类型
　　[message] - 错误消息
　　[file] - 发生错误所在的文件
　　[line] - 发生错误所在的行

```
1 <?php
2 register_shutdown_function('zyfshutdownfunc');
3 function zyfshutdownfunc()
4 {
5 if ($error = error_get_last()) {
6             var_dump('<b>register_shutdown_function: Type:' . $error['type'] . ' Msg: ' . $error['message'] . ' in ' . $error['file'] . ' on line ' . \$error['line'] . '</b>');
7 }
8 }
9 ?>
```
　　通过这种方法就可以巧妙的打印出程序结束前所有的错误信息。但是我在测试的时候我发现并不是所有的错误终止后都会调用这个函数，可以看下面的一个测试文件，内容是：

```
1 <?php
2 register_shutdown_function('zyfshutdownfunc');
3 function zyfshutdownfunc()
4 {
5 if ($error = error_get_last()) {
 6             var_dump('<b>register_shutdown_function: Type:' . $error['type'] . ' Msg: ' . $error['message'] . ' in ' . $error['file'] . ' on line ' . \$error['line'] . '</b>');
7 }
8 }
9 var_dump(23+-+); //此处语法错误
10 ?>

　　自己可以试一下，你可以看到根本就不会触发 zyfshutdownfunc()函数，其实这是一个语法错误，直接报了一个：

1 <?php
2 Parse error: syntax error, unexpected ')' in /www/mytest/exception/try-catch.php on line 71
3 ?>
　　由此引出一个奇葩的问题：问什么不能触发，为什么框架中是可以的？其实原因很简单，只在 parse-time 出错时是不会调用本函数的。只有在 run-time 出错的时候，才会调用本函数，我的理解是语法检查器前没有执行 register_shutdown_function()去把需要注册的函数放到调用的堆栈中，所以就根本不会运行。那框架中为什么任何错误都能进入到 register_shutdown_function()中呢，其实在框架中一般会有统一的入口 index.php，然后每个类库文件都会通过 include \*\* 的方式加载到 index.php 中，相当与所有的程序都会在 index.php 中聚集，同样，你写的具有语法错误的文件也会被引入到入口文件中，这样的话，调用框架，执行 index.php，index.php 本身并没有语法错误，也就不会产生 parse-time 错误，而是 include 文件出错了，是 run-time 的时候出错了，所以框架执行完之后就会触发 register_shutdown_function();
　　所以现在可是试一下这个写法，这样就会触发 zyfshutdownfunc()回调了：

```
1 a.php 文件
2 <?php
3 　　// 模拟语法错误
4 　　 var_dump(23+-+);
5 ?>
6
7 b.php 文件
8 <?php
9 register_shutdown_function('zyfshutdownfunc');
10 function zyfshutdownfunc()
11 {
12 if ($error = error_get_last()) {
13             var_dump('<b>register_shutdown_function: Type:' . $error['type'] . ' Msg: ' . $error['message'] . ' in ' . $error['file'] . ' on line ' . \$error['line'] . '</b>');
14 }
15 }
16 require 'a.php';
17 ?>
```
　　 3：set_exception_handler()
　　设置默认的异常处理程序，用在没有用 try/catch 块来捕获的异常，也就是说不管你抛出的异常有没有人捕获，如果没有人捕获就会进入到该方法中，并且在回调函数调用后异常会中止。看一下用法：

```
1 <?php
2 set_exception_handler('zyfexception');
3 function zyfexception($exception)
4     {
5         var_dump("<b>set_exception_handler: Exception: " . $exception->getMessage() . '</b>');
6 }
7 throw new Exception("zyf exception");
8 ?>
```
四、巧妙的捕获错误和异常
　　 1：把错误以异常的形式抛出(不能完全抛出)
　　　　由上面的讲解我们知道，php 中的错误是不能以异常的像是捕获的，但是我们需要让他们抛出，已达到扩展 try-catch 的影响范围，我们前面讲到过 set_error_handler() 方法，他是干嘛用的，他是捕获错误的，所以我们就可以借助他来吧错误捕获，然后再以异常的形式抛出，ok，试试下面的写法：

```
1 <?php
2 set_error_handler('zyferror');
3 function zyferror($type, $message, $file, $line)
4 {
5 throw new \Exception($message . 'zyf错误当做异常');
 6     }
 7 
 8     $num = 0;
9 try {
10 echo 1/$num;
11 
12     } catch (Exception $e){
13 echo \$e->getMessage();
14 }
15 ?>
```
　　好了，试一下，会打印出：

1 Division by zero zyf123
　　流程：本来是除 0 错误，然后触发 set_error_handler()，在 set_error_handler()中相当与杀了个回马枪，再把错误信息以异常的形式抛出来，这样就可以实现错误以异常的形式抛出。大家要注意：这样做是有缺点的，会受到 set_error_handler()函数捕获级别的限制。

     2：捕获所有的错误
        由set_error_handler()可知，他能够捕获一部分错误，不能捕获系统级E_ERROR、E_PARSE等错误，但是这部分可以由register_shutdown_function()捕获。所以两者结合能出现很好的功能。
        看下面的程序：

```
1 a.php 内容：
2 <?
3 // 模拟 Fatal error 错误
4 //test();
5
6 // 模拟用户产生 ERROR 错误
7 //trigger_error('zyf-error', E_USER_ERROR);
8
9 // 模拟语法错误
10 var_dump(23+-+);
11
12 // 模拟 Notice 错误
13 //echo \$f;
14
15 // 模拟 Warning 错误
16 //echo '123';
17 //ob_flush();
18 //flush();
19 //header("Content-type:text/html;charset=gb2312");
20 ?>

21 b.php 内容：
22 <?
23 error_reporting(0);
24 register_shutdown_function('zyfshutdownfunc');
25 function zyfshutdownfunc()
26 {
27 if ($error = error_get_last()) {
28             var_dump('<b>register_shutdown_function: Type:' . $error['type'] . ' Msg: ' . $error['message'] . ' in ' . $error['file'] . ' on line ' . $error['line'] . '</b>');
29         }
30     }
31 
32     set_error_handler('zyferror');
33     function zyferror($type, $message, $file, $line)
34     {
35         var_dump('<b>set_error_handler: ' . $type . ':' . $message . ' in ' . $file . ' on ' . \$line . ' line .</b><br />');
36 }
37
38 require 'a.php';
39 ?>
```
　　到此就可以解释开头的那个程序了吧，test.php 如果是单文件执行是不能捕获到错误的，如果你在框架中执行就是可以的，当然你按照我上面介绍的来扩展也是可以的。

五、自定义异常处理和异常嵌套

1：自定义异常处理

在复杂的系统中，我们往往需要自己捕获我们需要特殊处理的异常，这些异常可能是特殊情况下抛出的。所以我们就自己定义一个异常捕获类，该类必须是 exception 类的一个扩展，该类继承了 PHP 的 exception 类的所有属性，并且我们可以添加自定义的函数，使用的时候其实和之前的一样，大致写法如下：

```
1 <?php
2 class zyfException extends Exception
3 {
4 public function errorzyfMessage()
5 {
6 return 'Error line ' . $this->getLine().' in ' . $this->getFile()
7 .': <b>' . $this->getMessage() . '</b> Must in (0 - 60)';
 8         }
 9     }
10 
11     $age = 10;
12 try {
13 $age = intval($age);
14 if($age > 60) {
15             throw new zyfException($age);
16 }
17
18 } catch (zyfException $e) {
19         echo $e->errorzyfMessage();
20
21 }
22 ?>
```
　　 2：异常嵌套

异常嵌套是比较常见的写法，在自定义的异常处理中，try 块中可以定义多个异常捕获，然后分层传递异常，理解和冒泡差不多，看下面的实现：

```
1 <?php
2 $age = 10;
 3     try {
 4         $age = intval($age);
 5         if($age > 60) {
6 throw new zyfException($age);
 7         }
 8 
 9         if ($age <= 0) {
10 throw new Exception($age . ' must > 0');
11         }
12 
13     } catch (zyfException $e) {
14 echo $e->errorzyfMessage();
15 
16     } catch(Exception $e) {
17 echo \$e->getMessage();
18 }
19 ?>
```
　　当然也可以在 catch 中再抛出异常给上层：

```
1 <?php
2 $age = 100;
 3     try {
 4         try {
 5             $age = intval($age);
 6             if($age > 60) {
7 throw new Exception($age);
 8             }
 9 
10         } catch (Exception $e) {
11 throw new zyfException($age);
12 
13         }
14 
15     } catch (zyfException $e) {
16 echo \$e->errorzyfMessage();
17 }
18 ?>
```
六、PHP7 中的异常处理
　　现在写 PHP 必须考虑版本情况，上面的写法在 PHP7 中大部分都能实现，但是也会有不同点，在 PHP7 更新中有一条：更多的 Error 变为可捕获的 Exception，现在的 PHP7 实现了一个全局的 throwable 接口，原来老的 Exception 和其中一部分 Error 实现了这个接口(interface)，PHP7 中更多的 Error 变为可捕获的 Exception 返回给捕捉器，这样其实和前面提到的扩展 try-catch 影响范围一样，但是如果不捕获则还是按照 Error 对待，看下面两个：

```
1 <?php
2 try {
3 test();
4
5 } catch(Throwable $e) {
 6         echo $e->getMessage() . ' zyf';
7 }
8
9 try {
10 test();
11
12 } catch(Error $e) {
13         echo $e->getMessage() . ' zyf';
14 }
15 ?>
```
因为 PHP7 实现了 throwable 接口，那么就可以使用第一个这种方式来捕获异常。又因为部分 Error 实现了接口，并且更多的 Error 变为可捕获的 Exception，那么就可以使用第二种方式来捕获异常。下面是在网上找的 PHP7 的异常层次树：
Throwable
　　 Exception 异常
　　　　...
　　 Error 错误
　　　　 ArithmeticError 算数错误
　　　　　　 DivisionByZeroError 除数为 0 的错误
　　　　 AssertionError 声明错误
　　　　 ParseError 解析错误
　　　　 TypeError 类型错误
转载：
http://www.cnblogs.com/zyf-zhaoyafei/p/6928149.html 