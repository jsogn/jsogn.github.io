---
layout: post
title: 平滑重启php-fpm
tags:
  - php
categories:
  - php
abbrlink: 52587
---

### 前言

php-fpm 会对下面几个信号作（自己的）处理

- SIGINT, SIGTERM: immediate termination
- SIGQUIT: graceful stop
- SIGUSR1: re-open log file
- SIGUSR2: graceful reload of all workers + reload of fpm conf/binary

<!--more-->

### 理解

master进程可以理解以下信号

- INT （2）, TERM（15） 立刻终止
- QUIT （3） 平滑终止
- USR1 重新打开日志文件
- USR2 平滑重载所有worker进程并重新载入配置和二进制模块

#### 查看进程数

```bash
ps aux | grep -c php-fpm
```

#### 查看master进程号

```bash
ps aux|grep 'php-fpm: master' | awk '{print $2}'
```

#### 平滑重启

```bash
kill -USR2 `cat /usr/local/php/var/run/php-fpm.pid`
```
OR
```bash
kill -USR2 [pid]
```

### 脚本实现

#### centos脚本实现

```bash
#!/bin/bash
echo "php-fpm is reloading...."
PID=`ps aux | grep php-fpm | grep "master" |awk '{print $2}'`

[ $PID ] && kill -USR2 $PID || echo "php-fpm is useing(pid=$PID)"
echo "reload done!"
echo "php-fpm is reload..."
echo "reload done!"
```
