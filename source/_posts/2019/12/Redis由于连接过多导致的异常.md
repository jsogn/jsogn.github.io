---
layout: post
title: Redis由于连接过多导致的异常
tags:
  - redis
categories:
  - redis
abbrlink: 16422
---

### 前言

近期服务器在高峰的时候经常报错，日志记录为：

```log
Redis->connect('127.0.0.1', 6379)
#1 {main}
  thrown in /wwwroot/test.php on line 9
[13-Jun-2019 11:07:47 PRC] PHP Fatal error:  Uncaught RedisException: Cannot assign requested address in /wwwroot/test.php:9
```
<!--more-->

### 解决方法

#### 方法一

执行命令修改如下 2 个内核参数  

```sh
sysctl -w net.ipv4.tcp_timestamps=1 开启对于 TCP 时间戳的支持, 若该项设置为 0，则下面一项设置不起作用

sysctl -w net.ipv4.tcp_tw_recycle=1 表示开启 TCP 连接中 TIME-WAIT sockets 的快速回收

Redis 错误 ：Cannot assign request

Could not connect to Redis at 127.0.0.1:6379: connect: Cannot assign request
```

经查官方 Wiki 是系统网络配置问题已经解决：

```sh
echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse
```

以上需要 root 权限对网络进行配置。

#### 方法二

通过调整内核参数解决，`vim /etc/sysctl.conf`，加入

```conf
net.ipv4.tcp_syncookies = 1 #表示开启 SYN Cookies。当出现 SYN 等待队列溢出时，启用 cookies 来处理，可防范少量 SYN 攻击，默认为 0，表示关闭；

net.ipv4.tcp_tw_reuse = 1 #表示开启重用。允许将 TIME-WAIT sockets 重新用于新的 TCP 连接，默认为 0，表示关闭，释放 TIME_WAIT 端口给新连接使用；

net.ipv4.tcp_tw_recycle = 1 #表示开启 TCP 连接中 TIME-WAIT sockets 的快速回收资源，默认为 0，表示关闭。

net.ipv4.tcp_fin_timeout = 30 #修改系統默认的 TIMEOUT 时间，调低端口释放后的等待时间，默认为 60s，修改为 15~30s

net.ipv4.tcp_max_tw_buckets = 10000# 通过设置它，系统会将多余的 TIME_WAIT 删除掉，此时系统日志里可能会显示：『TCP: time wait bucket table overflow』，多数情况下不用在意这些信息。
```

然后执行 `/sbin/sysctl -p` 让参数生效。

以上都可以通过命令（sysctl -w）方式操作，如：`sysctl -w net.ipv4.tcp_fin_timeout=30` ，只适合临时修改参数。

### TCP 网络参数优化

```sh
echo "1024 65535" > /proc/sys/net/ipv4/ip_local_port_range 设置向外连接可用端口范围 表示可以使用的端口为 65535-1024 个（0~1024 为受保护的)

echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse 设置 time_wait 连接重用 默认 0

echo 1 > /proc/sys/net/ipv4/tcp_tw_recycle 设置快速回收 time_wait 连接 默认 0

echo 180000 > /proc/sys/net/ipv4/tcp_max_tw_buckets 设置最大 time_wait 连接长度 默认 262144

echo 1 > /proc/sys/net/ipv4/tcp_timestamps  设置是否启用比超时重发更精确的方法来启用对 RTT 的计算 默认 0

echo 1 > /proc/sys/net/ipv4/tcp_window_scaling 设置 TCP/IP 会话的滑动窗口大小是否可变 默认 1

echo 20000 > /proc/sys/net/ipv4/tcp_max_syn_backlog 设置最大处于等待客户端没有应答的连接数 默认 2048

echo 15 > /proc/sys/net/ipv4/tcp_fin_timeout  设置 FIN-WAIT 状态等待回收时间 默认 60

echo "4096 87380 16777216" > /proc/sys/net/ipv4/tcp_rmem  设置最大 TCP 数据发送缓冲大小，分别为最小、默认和最大值  默认 4096    87380   4194304

echo "4096 65536 16777216" > /proc/sys/net/ipv4/tcp_wmem 设置最大 TCP 数据 接受缓冲大小，分别为最小、默认和最大值 　默认 4096    87380   4194304

echo 10000 > /proc/sys/net/core/somaxconn  设置每一个处于监听状态的端口的监听队列的长度 默认 128

echo 10000 > /proc/sys/net/core/netdev_max_backlog 设置最大等待 cpu 处理的包的数目 默认 1000

echo 16777216 > /proc/sys/net/core/rmem_max 设置最大的系统套接字数据接受缓冲大小 默认 124928

echo 262144 > /proc/sys/net/core/rmem_default  设置默认的系统套接字数据接受缓冲大小 默认 124928

echo 16777216 > /proc/sys/net/core/wmem_max  设置最大的系统套接字数据发送缓冲大小 默认 124928

echo 262144 > /proc/sys/net/core/wmem_default  设置默认的系统套接字数据发送缓冲大小 默认 124928

echo 2000000 > /proc/sys/fs/file-max 设置最大打开文件数 默认 385583
```

结合 ab 命令来压测机器优化网络，设置完记得保存

### 优化 Redis 命令

设置内存分配方式：

```
echo 1 > /proc/sys/vm/overcommit_memory
```

0 表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程。

1 表示内核允许分配所有的物理内存，而不管当前的内存状态如何。

2 表示内核允许分配超过所有物理内存和交换空间总和的内存

关闭 THP：

```
cho never > /sys/kernel/mm/transparent_hugepage/enabled
```

尽管 THP 的本意是为提升性能，但某些数据库厂商还是建议直接关闭 THP(比如说 Oracle、MongoDB 等)，否则可能导致性能下降，内存锁，甚至系统重启等问题。

```
echo 1024 >/proc/sys/net/core/somaxconn
```

限制了接收新 TCP 连接侦听队列的大小。对于一个经常处理新连接的高负载 web 服务环境来说，默认的 128 太小了。大多数环境这个值建议增加到 1024 或者更多。 服务进程会自己限制侦听队列的大小 (例如 sendmail(8) 或者 Apache)，常常在它们的配置文件中有设置队列大小的选项。大的侦听队列对防止拒绝服务 DoS 攻击也会有所帮助。