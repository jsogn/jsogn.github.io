---
layout: post
title: 设置CentOS服务器IP地址
tags:
  - linux
categories:
  - linux
abbrlink: 33748
---

### 前言

如何在CentOS服务器中配置网络IP地址

<!--more-->

### 设置IP地址

```shell
cd /etc/sysconfig/network-scripts/
```

查看配置信息

```shell
ifconfig
```

编辑网卡配置

```shell
vim ifcfg-eno1
```

编辑信息，建议 `ONBOOT=yes`，以后开机就会自动联网：

```ini
TYPE="Ethernet"
BOOTPROTO="none"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
NAME="eno1"
UUID="c63850e5-4c25-46c7-a030-574fcf824ad5"
DEVICE="eno1" #设备别名
ONBOOT="yes"
IPADDR1=202.95.22.222 #从IP地址1
PREFIX1="25"
IPADDR2=202.95.22.233 #从IP地址2
PREFIX2="25"
DNS1="8.8.8.8" #DNS服务器
IPADDR=202.95.22.212  #设置主IP地址
PREFIX=25
GATEWAY=202.95.8.129 #网关
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_PRIVACY=no
```

### 重启网络配置

```shell
systemctl restart network
 ```
