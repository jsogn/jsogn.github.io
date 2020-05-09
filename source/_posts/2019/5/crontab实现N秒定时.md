---
layout: post
title: crontab实现N秒定时
tags:
  - linux
categories:
  - linux
abbrlink: 65133
date: 2019-05-27 10:50:57
---

### 前言

crontab 命令最小的执行时间是一分钟，有时需要按秒执行定时任务，有两种方法可以实现

<!--more-->

### 使用延时来实现每N秒执行

通过延时方法 sleep N 来实现每N秒执行，首先创建一个php脚本test.php,本例test.php放在home目录下，功能是把当前时间写入/log/run.log

```shell
crontab -e
```

输入以下语句保存退出：

```shell
* * * * * php /home/test.php
* * * * * sleep 10; php /home/test.php
* * * * * sleep 20; php /home/test.php
* * * * * sleep 30; php /home/test.php
* * * * * sleep 40; php /home/test.php
* * * * * sleep 50; php /home/test.php
```

使用 tail -f 查看执行情况

```shell
tail -f /log/run.log
```

60必须能整除间隔的秒数，如果间隔的秒数太少，例如2秒执行一次，这样就需要在crontab 加入60/2=30条语句

### 编写shell脚本实现

在脚本中使用for语句指定秒数执行，创建脚本文件crontab.sh,j本例放在home目录下：

```shell
#!/bin/bash

step=2 #间隔的秒数，不能大于60

for (( i = 0; i < 60; i=(i+step) )); do
    $(php '/home/test.php')
    sleep $step
done

exit 0
```

crontab -e 输入以下语句后保存退出：

```shell
* * * * * /home/crontab.sh
```

使用 tail -f 查看执行情况，可以发现run.log每2秒被写入一条记录。如果60不能整除间隔的秒数，则需要调整执行的时间。例如需要每7秒执行一次，就需要找到7与60的最小公倍数，7与60的最小公倍数是420（即7分钟）。

则 crontab.sh step的值为7，循环结束条件i<420， crontab="" -e可以输入以下语句来实现:`<="" p="">`

```shell
*/7 * * * * /home/crontab.sh
```