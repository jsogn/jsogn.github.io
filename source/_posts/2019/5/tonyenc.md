---
layout: post
title: php代码加密-tonyenc
tags:
  - php
categories:
  - php
abbrlink: 50056
date: 2019-05-16 19:19:11
---

### 前言

最近公司出售了一个项目，商业源码自然要加密，网上的一些加密被破解的可能性较大，用起来也不方便，作为github的伟大搬运工，找到了一个简洁、高性能、跨平台的 PHP7 代码加密扩展-tonyenc

<!-- more -->

github项目地址：https://github.com/lihancong/tonyenc

### 安装

```shell
git clone https://github.com/lihancong/tonyenc

cd tonyenc

phpize

./configure --with-php-config = [自己php版本的php-config文件]

make && make install
```

> php版本要7.0以上

修改php.ini文件追加扩展

```shell
extension=tonyenc.so
```

> MAMP软件中修改php.ini模版文件，是无效的，要修改真实的php.ini文件

搞定后重启环境，测试是否成功

```shell
php -m |grep tonyenc
```

### 加密

#### 加密不可逆，切记要备份！！！！

加密文件，这个文件它提供了，名字叫tonyenc.php，复制保存到合适位置，然后执行

```shell
php tonyenc.php [需要加密的文件或路径]
```