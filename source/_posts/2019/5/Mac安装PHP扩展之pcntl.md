---
title: Mac 安装 PHP 扩展 pcntl
tags:
  - mac
  - php
categories:
  - php
abbrlink: 33840
---

在 Mac 下做 PHP 开发用的是 MAMP 集成开发环境，出现 PHP 不支持 pcntl 扩展，查下谷歌发现 MAMP 的集成环境是没有这个扩展包的，需要手动编译安装这个包。


### 具体步骤

```shell

# 下载源码包
wget http://cn.php.net/distributions/php-7.2.1.tar.gz

# 解压
tar zxvf php-7.2.1.tar.gz

# 进入文件执行编译
cd php-7.2.1/ext/pcntl
phpize
./configure
make

# 拷贝编译.so文件到MAMP extensions目录(具体的文件夹看自己的目录哦)
cp modules/pcntl.so /Applications/MAMP/bin/php/php7.2.1/lib/php/extensions/no-debug-non-zts-20170718

# 编辑php.ini引入扩展
echo "extension=pcntl.so" >> /Applications/MAMP/bin/php/php7.2.1/conf/php.ini
```

### 检查是否安装成功

```shell

php --ri pcntl
```
