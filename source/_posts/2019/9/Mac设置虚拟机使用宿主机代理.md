---
layout: post
title: Mac设置虚拟机使用宿主机代理
tags:
  - mac
categories:
  - 科学上网
abbrlink: 24292
---


### 前言

前提：mac 本身已经安装 ss，并且可以通过 ss 科学上网（或者 win 也一样）
想要实现：Parallels Desktop 里的虚拟机也想科学上网（或者 win 里安装 vmware 也一样，或者是另一台物理机也是一样的）

<!--more-->

### 方法一：虚拟机也安装一个 ss 客户端

对于有窗口的系统，比如你虚拟机里安装的是 win、或者 ubuntu 等，那么再安装一个客户端是很方便的，这就相当于在另一台电脑里使用 ss 客户端，既然 mac 上你会用了，那在其他电脑上也是一样的。

但是如果虚拟机里是最小化安装的纯命令行的 centos，那么使用客户端可能有一定的麻烦，yum 无法安装，pip 安装的感觉也是 ssserver，并没有客户端，所以客户端还得编译，编译还有很多依赖，编译好还得写对配置文件，很多人都不太清楚这个，所以还是挺麻烦的。

### 方法二：虚拟机设置代理到宿主机

即虚拟机里设置代理到 mac（这里 mac 就是虚拟机的宿主机），让虚拟机通过 mac 的 ss 科学上网，这里如果宿主机换成 win，虚拟机软件换成 vmware 或 virtualbox，它的原理也都是一样的。

**设置方法：**

1、首先把宿主机的 ss 设置里的 Local Socks5 Listen Address 由原来的`127.0.0.1`设置为`0.0.0.0`，如果需要通过 HTTP 代理，那么也要把 ss 里的 HTTP 选项打开，并把 HTTP proxy Listen Address 地址由原来的`127.0.0.1`设置为`0.0.0.0`，这样做表示代理所有 ip，而不只是本机的`127.0.0.1`。如果用的是其他科学上网工具，也有些写成 “share over LAN(通过局域网共享)”，如果有这个选项，选上了就表示监听`0.0.0.0`。

2、搞清楚虚拟机是通过什么方式联网的，虚拟机连网无非有两种方式：

*   桥接
*   NAT

如果是桥接连网，那么只要找出宿主机的联网 ip 即可（mac 的话，一般都是 wifi，或者通过转接头插网线的话，那就是转接头对应的 ip）

如果是 NAT 连网的，那么要找出宿主机中 NAT 网卡的 ip（在 mac 里使用 parallels desktop 虚拟机的话，NAT 网卡一般是 parallels Shared 开头的）

3、在虚拟机里的`~/.bashrc`或`~/.zshrc`里，添加以下两句的其中一句：

一般填入局域网IP即可

```
export ALL_PROXY=SOCKS5://IP:端口
export ALL_PROXY=HTTP://IP:端口
```

第一句表示使用 SOCKS5 代理，第二句表示使用 HTTP 代理，ip 就是第 2 步中找到的 ip，端口就是 ss 对应的端口，打开 ss 的设置里就有，一般 ss 有两个端口，一个 socks5 端口，一个 http 端口，找到对应端口填进去即可。

最后 source 一下配置文件：
```
source ~/.bashrc
```
或者用 zsh shell 的话就是：
```
source ~/.zshrc
```

测试 ip 是哪里的：

```
curl https://ip.cn
```

如果显示的是代理服务器所在地址 (比如美国) 和 ip，那说明代理设置成功。

然后试试能否访问 google：
```
curl https://www.google.com
```

如果是 Windows，可以直接在 cmd 里设用`set http_proxy=http://127.0.0.1:1087`，`set https_proxy=http://127.0.0.1:1087`，`set all_proxy=http://127.0.0.1:1087`

* * *

更好的写法

```
# 设置使用代理
alias setproxy="export https_proxy=http://127.0.0.1:1087; export http_proxy=http://127.0.0.1:1087; export all_proxy=socks5://127.0.0.1:1086; echo 'Set proxy successfully'"
# 设置取消使用代理
alias unsetproxy="unset http_proxy; unset https_proxy; unset all_proxy; echo 'Unset proxy successfully'"
```





