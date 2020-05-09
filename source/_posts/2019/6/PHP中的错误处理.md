---
layout: post
title: PHP中的错误处理
tags:
  - php
categories:
  - php
abbrlink: 30879
---

### 前言

PHP里有一套错误处理机制，可以使用set_error_handler接管PHP错误处理，也可以使用trigger_error函数主动抛出一个错误。

<!--more-->

### set_error_handler

set_error_handler()函数设置用户自定义的错误处理函数。函数用于创建运行期间的用户自己的错误处理方法。它需要先创建一个错误处理函数，然后设置错误级别。语法如下：

```php
set_error_handler(error_function, error_type)
```

- error_function :发生错误时运行的函数
- error_type : 错误级别，默认为E_ALL

如果使用该函数，会完全绕过PHP错误处理函数，如果有必要，自定义的错误处理程序必须终止脚本

如果在脚本执行前发生错误，那时自定义程序还没有注册，就不会用到自定义错误处理程序

### 自定义异常处理

```php

<?php

function customError($errno, $errstr, $errfile, $errline)
{
    echo '错误代码:'$errno.$errstr;
    echo '错误所在:'$errfile.$errline;
}

set_error_handler(customError, E_ALL|E_STRICT);

$a = ['o' => 2,4,5];

echo $a['o'];

```

自定义的错误处理函数一定要有这四个输入变量＄errno、＄errstr、＄errfile、＄errline

errno是一组常量，代表错误的等级，同时也有一组整数和其对应，但一般使用其字符串值表示，这样语义更好一点。比如E_WARNING，其二进制掩码为4.，表示警告信息

接下来，就是将这个函数作为回调参数传递给set_error_handler。这样就能接管PHP原生的错误处理函数了。要注意的是，这种托管方式并不能托管所有种类的错误，如E_ERROR、E_PARSE、E_CORE_ERROR、E_CORE_WARNING、E_COMPILE_ERROR、E_COMPILE_WARNING，以及E_STRICT中的部分。这些错误会以最原始的方式显示，或者不显示

### restore_error_handler

set_error_handler函数会接管PHP内置的错误处理，可以在同一个页面使用restore_error_handler()；取消接管

如果使用自定义的set_error_handler接管PHP的错误处理，代码里的错误抑制@将失效，这种错误也会被显示

在PHP异常中，异常处理机制是有限的，无法自动抛出异常，必须手动进行，并且内置异常有限。PHP把许多异常看作错误，这样就可以把这些“异常”像错误一样用set_error_handler接管，进而主动抛出异常

```php

function customError($errno, $errstr, $errfile, $errline)
{
    throw new Exception($errno.'|'.$errstr);
}

set_error_handler('customError', E_ALL|E_STRICT);

try
{
    $a = 5/0;
}
catch(Exception $e)
{
    echo $e->getMessage();
}

```

这样就能捕获到异常和非致命的错误，可以弥补PHP异常处理机制的部分不足


### register_shutdown_function

fetal error这样的错误捕获不到，也无法在发生此错误后恢复流程处理，但是还是可以使用一些特殊方法对这种错误进行处理。这需要用到`register_shutdown_function()`

此函数会在PHP程序终止或者die时触发一个函数

```php

<?php

class Shutdown
{
    public function stop()
    {
        if (error_get_last()) {
            print_r(error_get_last);
        }

        die('Stop');
    }

    register_shutdown_function([new Shutdown(), 'stop']);

    $a = new a();

    echo 'end';
}

```

对于fetal error还能做点收尾工作，但是PHP流程的终止是必然的。对于Parse error级别的错误，除了可以修改配置文件php.ini，什么都做不了

```
log_errors = On
error_log = usr/log/php.log
```

这样一旦PHP发生了错误，就会被记入log文件，方便以后查询

### trigger_error

和exception类似，错误处理也有对应抛出错误的函数，那就是trigger_error函数

```php

<?php

$d = 0;

if ($d == 0) {
    trigger_error('cannot d by zero', E_USER_ERROR);
}

echo 'break';
```

### 结语

在PHP中，错误和异常是两个不同的概念，这种设计从根本上导致了PHP的异常和其他语言相异。以Java为例，Java中，异常是错误唯一的报告方式。说到底，两者的区别就是对异常和错误的认识不同而产生的。PHP的异常绝大部分必须通过某种办法手动抛出，才能被捕获到，是一种半自动化的异常处理机制。 无论是错误还是异常，都可以使用handler接管系统已有的处理机制