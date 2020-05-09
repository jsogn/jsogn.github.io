---
layout: post
title: nmap基本使用
tags:
  - tool
categories:
  - hacker
abbrlink: 5820
---

### 前言

Nmap 是个端口扫描器，这意味着它可以向一些指定 IP 的 TCP 或 UDP 端口发送封包，并检查是否有响应。如果有的话，这意味着端口是打开的，因此，端口上运行着服务

Nmap代表Network Mapper，是一款用于网络探索和安全审计的开源工具，它与Kali Linux标准兼容，但也可用于Windows，OSX和许多其他UNIX平台。Nmap还有一个称为Zenmap的图形用户界面。

<!--more-->

#### 打开端口扫描和操作系统检测

使用以下命令确定活动主机：

```
nmap -sP 192.168.0.0-100
```

使用以下命令在其中一个实时主机上启动带有操作系统检测的SYN扫描

```
nmap -sS [IP地址] -O
```

使用以下命令启动一个开放端口扫描和版本检测

```
nmap -sV 192.168.0.1 -A
```

将-v添加到命令中时，可以增加冗长度

```
nmap -sV 192.168.0.13 -A -v
```



#### 服务器是否响应 ping，或者服务器是否打开

使用-sn参数，我们让 Nmap 只检查是否服务器响应 ICMP 请求（或 ping）

```
nmap -sn 192.168.56.102
```

#### 打开了哪些端口

调用 Nmap 的最简方式，它只指定目标 IP。所做的事情是先 ping 服务器，如果它响应了，Nmap 会向 1000 个 TCP 端口列表发送探针，来观察哪个端口响应，之后报告响应端口的结果

```
nmap 192.168.56.102
```


#### Nmap 向服务器询问正在运行的服务的版本，并且基于它猜测操作系统

- sV请求每个被发现的开放端口的标识（头部或者自我识别），这是它用作版本的东西。

- O告诉 Nmap，尝试猜测运行在目标上的操作系统。使用开放端口和版本收集的信息。


```
nmap -sV -O 192.168.56.10
```

####  更加清楚的查看这个端口，并且看看可以确认什么

使用此命令，让 nmap 在主机上的 FTP 端口（-p 21）上运行其默认脚本（-sC）

```
nmap -sC 192.168.56.102 -p 21
```

#### Nmap 包含了一些脚本，用于测试 WAF 的存在

```
map -p 80,443 --script=http-waf-detect 192.168.56.102
```

#### 另一个 Nmap 脚本，可以帮助我们识别所使用的设备，并更加精确

```
nmap -p 80,443 --script=http-waf-fingerprint www.example.com
```


#### 有一些其它的实用参数：

- sT：通常，在 root 用户下运行 Nmap 时，它使用 SYN 扫描类型。使用这个参数，我们就强制让扫描器执行完全连接的扫描。它更慢，并且会在服务器的日志中留下记录，但是它不太可能被入侵检测系统检测到。
- Pn：如果我们已经知道了主机是活动的或者不响应 ping，我们可以使用这个参数告诉 Nmap 跳过 ping 测试，并扫描所有指定目标，假设它们是开启的。
- v：这会开启详细模式。Nmap 会展示更多关于它所做事情和得到回复的信息。参数可以在相同命令中重复多次：次数越多，就越详细（也就是说，-vv或-v -v -v -v）。
- p N1,N2,Nn：如果我们打算测试特定端口或一些非标准端口，我们可能想这个参数。N1到Nn是打算让 Nmap 扫描的端口。例如，要扫描端口 21，80 到 90，和 137，参数应为：-p 21,80-90,137。

- --script=script_name：Nmap 包含很多实用的漏洞检测、扫描和识别、登录测试、命令执行、用户枚举以及其它脚本。使用这个参数来告诉 Nmap 在目标的开放端口上运行脚本。查看一些 Nmap 脚本，它们在：https://nmap.org/nsedoc/scripts/。

### Nmap选项摘要

**用法：nmap [扫描类型] [选项] {目标规格}**

#### 目标规格：

可以传递主机名，IP地址，网络等。
例如：scanme.nmap.org，microsoft.com/24,192.168.0.1; 10.0.0-255.1-254

- -iL <inputfilename>：从主机/网络列表中输入
- -iR <num hosts>：选择随机目标
- -exclude <host1 [，host2] [，host3]，…>：排除主机/网络
- -excludefile <exclude_file>：从文件中排除列表

#### 主机发现：

- -sL：列表扫描 – 仅列出要扫描的目标
- -sn：Ping扫描 – 禁用端口扫描
- -Pn：将所有主机视为联机 – 跳过主机发现
- -PS / PA / PU / PY [portlist]：TCP SYN / ACK / UDP / SCTP发现到指定端口
- -PE / PP / PM：ICMP回显，时间戳和网络掩码请求发现探测
- -PO [协议列表]：IP协议Ping
- -n / -R：从不执行DNS解析/总是解析[默认：有时]
- -dns-servers <serv1 [，serv2]，…>：指定自定义DNS服务器
- -system-dns：使用操作系统的DNS解析器
- -traceroute：每个主机的跟踪跳转路径

#### SCAN技术：

- -sS / sT / sA / sW / sM：TCP SYN / Connect（）/ ACK / Window / Maimon扫描
- -sU：UDP 扫描-sN
- / sF / sX：TCP Null，FIN和Xmas扫描
- –scanflags < flags>：自定义TCP扫描标志
- -sI <zombie host [：probeport]>：空闲扫描
- -sY / sZ：SCTP INIT / COOKIE-ECHO扫描
- -sO：IP协议扫描
- -b <FTP中继主机>：FTP反弹扫描

#### 端口规格和扫描 顺序：-p

<端口范围>：仅扫描指定的端口

- 例如：-p22; -p1-65535; -p U：53,111,137，T：21-25,80,139,8080，S：9
- -exclude-ports <端口范围>：从扫描中排除指定端口
- -F：快速模式 – 扫描端口少于默认扫描
- -r：连续扫描端口 – 不随机化
- -top-ports <number>：扫描<number>最常用的端口
- -port-ratio <ratio>：扫描比<ratio>

#### 服务/版本检测：

- -sV：探测开放端口以确定服务/版本info
- -version-intensity <level>：设置从0（亮）到9（尝试所有探测器）
- -version-light：限制最可能的探测器2）
- -version-all：尝试每个探测器（强度9）
- -version-trace：显示详细版本的扫描活动（用于调试）

#### SCRIPT SCAN：

- -sC：相当于-script =默认
- -script = <Lua脚本>：<Lua scripts>是逗号分隔的
- 目录列表，脚本文件或脚本类别
- -script-args = <n1 = v1，[ n2 = v2，…]>：为脚本提供参数
- -script-args-file = filename：在文件中提供NSE脚本参数
- -script-trace：显示所有发送和接收的数据
- -script-updatedb：更新脚本数据库。
- -script-help = <Lua scripts>：显示有关脚本的帮助。
- <Lua scripts>是脚本文件或
- 脚本类别的逗号分隔列表。

#### 操作系统检测：

- -O：启用操作系统检测
- -osscan-limit：限制操作系统检测为有前途的目标
- -osscan-guess：猜测操作系统更积极

#### 时间和性能：

- 采用`<time>`的选项以秒为单位，或附加’ms’（毫秒），‘s’（秒），’m’（分钟）或’h’（小时） ）。
- -T <0-5>：设置定时模板（更高更快）
- -min-hostgroup / max-hostgroup <size>：并行主机扫描组大小
- -min-parallelism / max-parallelism <numprobes>：探针并行化
- -min- rtt-timeout / max-rtt-timeout / initial-rtt-timeout <time>：指定
- 探测往返时间。
- -max-retries <tries>：端口扫描探测器重新传输的大小数量。
- -host-timeout <time>：在这个long
- -scan-delay 之后放弃目标/ -max-scan-delay <time>：调整探针之间的延迟
- -min-rate <number>：
- -max-rate <number>：发送数据包不超过每秒<number>

#### 防火墙/ IDS消除和防盗：

- -f; -mtu <val>：片段数据包（可选w / given MTU）
- -D <decoy1，decoy2 [，ME]，…>：用诱饵隐藏扫描
- -S <IP_Address>：欺骗源地址
- -e <iface>：使用指定的接口
- -g / -source-port <
- portnum >：使用给定的端口号-proxies <url1，[url2]，…>：通过HTTP / SOCKS4代理服务器的中继连接
- -data <十六进制字符串>：将自定义有效内容附加到已发送数据包
- -data-string <string>：将自定义ASCII字符串附加到已发送数据包
- -data-length <num>：将随机数据附加到已发送数据包
- -ip-options <选项>：发送指定ip选项的数据包
- -ttl <val>：设置IP生存时间字段
- -spoof-mac <
- -badsum：发送虚假TCP / UDP / SCTP校验和的数据包

#### OUTPUT

- -oN / -oX / -oS / -oG <file>：分别以正常，XML，s | <rIpt kIddi3
- 和Grepable格式输出扫描到给定的文件名。
- -oA <basename>：一次输出三种主要格式
- -v：提高详细级别（使用-vv或更多以获得更大效果）
- -d：提高调试级别（使用-dd或更多以获得更大效果）
- -reason：Display端口处于特定状态的原因
- -open：只显示打开的（或可能打开的）端口
- -packet-trace：显示所有发送和接收的数据包
- -iflist：打印主机接口和路由（用于调试）
- -log-errors：Log错误/警告到正常格式的输出文件
- -append-output：附加到指定的输出文件而不是clobber
- -resume <filename>：恢复中止的扫描
- -stylesheet <path / URL>：XSL样式表将XML输出转换为HTML
- -webxml：Nmap.Org的参考样式表，用于更多可移植的XML
- -no-stylesheet：防止关联XSL样式表w / XML输出

#### MISC：

- -6：启用IPv6扫描
- -A：启用OS检测，版本检测，脚本扫描和traceroute
- -datadir <dirname>：指定自定义Nmap数据文件位置
- -send-eth / -send-ip：使用原始以太网帧进行发送或IP数据包
- -privileged：假设用户具有完全特权
- -unprivileged：假定用户缺少原始套接字权限
- -V：打印版本号
- -h：打印此帮助摘要页面


#### 另见

虽然它最为流行，但是 Nmap 不是唯一可用的端口扫描器，并且，取决于不同的喜好，可能也不是最好的。下面是 Kali 中包含的一些其它的替代品：

- unicornscan
- hping3
- masscan
- amap
- Metasploit scanning module