---
layout: post
title: GoogleHacking
tags:
  - google
categories:
  - hacker
abbrlink: 47235
---

### 前言

Google hacker (Google黑客)是利用GOOGLE提供的搜索功能查找黑客们想找到的信息，一般是查找网站后台，网管的个人信息，也可以用来查找某人在网络上的活动

Google hacker 一般是做为黑客在入侵时的一个手段，在入侵过程中有时需要查找后台的登陆口就需要用到GOOGLE HACKER，有时猜解密码的时候google也是提供查找管理员资料的有效平台

<!--more-->

### 常用语法

#### intext

这个就是把网页中的正文内容中的某个字符做为搜索条件，例如在google里输入`intext:动漫`，将返回所有在网页正文部分包含”动漫”的网页，`allintext:`使用方法和`intext`类似

#### intitle

和intext差不多，搜索网页标题中是否有所要找的字符，例如搜索:`intitle:安全天使`，将返回所有网页标题中包含”安全天使”的网页，同理`allintitle:`也同`intitle`类似

#### cache

搜索google里关于某些内容的缓存，也许能找到一些好东西

#### define

搜索某个词语的定义，搜索:`define:hacker`，将返回关于hacker的定义

#### filetype

搜索指定类型的文件，撒网式攻击还是对特定目标进行信息收集都需要用到这个，例如输入:`filetype:doc`，将返回所有以doc结尾的文件URL，如果找.bak、.mdb或.inc也是可以的，获得的信息也许会更丰富


#### info

查找指定站点的一些基本信息

#### inurl

搜索:`inurl:www.123.net`可以返回所有和`www.123.net`做了链接的URL

#### site

搜索:`site:www.123.net`，将返回所有和123.net这个站有关的URL

还有一些操作符也是很有用的:

```
– 把某个字忽略

~ 同意词

. 单一的通配符

“” 精确查询
```

### 典型用法

- 找管理后台地址

```
site:xxx.com intext：管理后台登陆用户名密码系統账号

site:xxx.com inurl: login/admin/manage/manager/adminlogin/system 

site:xxx.com intitle 管理后台登陆
```

- 找上传类漏洞地址：

```
site:xxx.com inurl:file 

site:xxx.com inurl:upload 
```

- 找注入页面

```
site:xxx.com inurl:php?id=
```

- 找编辑器页面

```
site:xxx.com inurl:ewebeditor
```

### 实际应用

对于攻击者来说，可能最感兴趣的就是密码文件了，而google正因为其强大的搜索能力往往会把一些敏感信息透露出来，用google搜索以下内容

```
intitle:”index of” etc

intitle:”Index of” .sh_history

intitle:”Index of” .bash_history

intitle:”index of” passwd

intitle:”index of” people.lst

intitle:”index of” pwd.db

intitle:”index of” etc/shadow

intitle:”index of” spwd

intitle:”index of” master.passwd

intitle:”index of” htpasswd

“# -FrontPage-” inurl:service.pwd
```


同样可以用google来搜索一些具有漏洞的程序，例如ZeroBoard前段时间发现个文件代码泄露漏洞，我们可以用google来找网上使用这套程序的站点:

```
intext:ZeroBoard filetype:php
```

或者使用:

```
inurl:　outlogin.php?_zb_path= site:.jp
```


phpmyadmin是一套功能强大的数据库操作软件，一些站点由于配置失误，导致可以不使用密码直接对phpmyadmin进行操作，可以用google搜索存在这样漏洞的程序

```
intitle:phpmyadmin intext:Create new database
```

可以用google来搜索数据库文件，用上一些语法来精确查找能够获得更多东西(access的数据库,mssql、mysql的连接文件等等)

```
allinurl:bbs data

filetype:mdb inurl:database

filetype:inc conn

inurl:data filetype:mdb

intitle:”index of” data //在一些配置不正确的apache+win32的服务器上经常出现这种情况
```


### 实战演示

利用google完全是可以对一个站点进行信息收集和渗透的，下面用google对特定站点进行一次测试

首先用google先看这个站点的一些基本情况

```
site:xxxx.com
```

从返回的信息中，找到几个该校的几个系院的域名：

```
http://a1.xxxx.com

http://a2.xxxx.com

http://a3.xxxx.com

http://a4.xxxx.com
```

顺便ping了一下，应该是在不同的服务器，学校一般都会有不少好的资料

```
site:xxxx.com filetype:doc
```


得到N个不错的doc。先找找网站的管理后台地址：

```
site:xxxx.com intext:管理

site:xxxx.com inurl:login

site:xxxx.com intitle:管理
```

超过获得2个管理后台地址：

```
http://a2.xxxx.com/sys/admin_login.asp

http://a3.xxxx.com:88/_admin/login_in.asp
```


看看服务器上跑的是什么程序：

```
site:a2.xxxx.com filetype:asp

site:a2.xxxx.com filetype:php

site:a2.xxxx.com filetype:aspx

site:a3.xxxx.com filetype:asp
```


a2服务器用的应该是IIS，上面用的是asp的整站程序，还有一个php的论坛

a3服务器也是IIS，aspx+asp。web程序都应该是自己开发的。有论坛那就看看能不能遇见什么公共的FTP帐号什么的：

```
site:a2.xxxx.com intext:ftp://:
```

再看看有没有上传一类的漏洞：

```
site:a2.xxxx.com inurl:file

site:a3.xxxx.com inurl:load
```

在a2上发现一个上传文件的页面：

```
http://a2.xxxx.com/sys/uploadfile.asp
```

用IE看了一下，没权限访问。试试注射

```
site:a2.xxxx.com filetype:asp
```

一般学校的站点的密码都比较有规律，通常都是域名+电话一类的变形

```
site:xxxx.com //得到N个二级域名

site:xxxx.com intext:@xxxx.com //得到N个邮件地址，还有邮箱的主人的名字什么的

site:xxxx.com intext:电话 //N个电话 
```

把这些信息做个字典，挂上慢慢跑。过了一段时间就跑出4个帐号，2个是学生会的，1个管理员，还有一个可能是老师的帐号



google hack其实也都差不多是一些基本语法的灵活运用，或者配合某个脚本漏洞，主要还是靠个人的灵活思维

对于一些在win上跑 apache的应该多注意一下这方面，一个`intitle:index of`就差不多都出来了 `echo “召唤” > index.jsp` 现在看看首页，已经被改成: “召唤” 了。

也可以用WGET上传一个文件上去，然后`execute Command输入 cat file > index.html or echo “”> file echo “test” >> file` 这样一条条打出来，站点首页就成功被替换了

同样的也可以 `uname -a;cat /etc/passwd` 不过有点要注意，有些WEBSHELL程序有问题，执行不了的，

#### 搜索INC敏感信息

在google的搜索框中填入: `Code: .org filetype:inc` 

例：常用的google关键字： foo1 foo2 (也就是关联，比如搜索xx公司 xx美女) 

`operator:foo filetype:123` 类型 

`site:foo.com` 相对直接看网站更有意思，可以得到许多意外的信息 

`intext:foo intitle: fooltitle` 标题 `allinurl:foo` 搜索xx网站的所有相关连接。（踩点必备） 

`links:foo `不要说就知道是它的相关链接 

`allintilte:foo.com` 可以辅助`”-” “+”`来调整搜索的精确程度 直接搜索密码：(引号表示为精确搜索)

可以再延伸到上面的结果里进行二次搜索

`“index of” htpasswd / passwd filetype:xls username password email “ws_ftp.log” “config.php” allinurl:admin mdb service filetype:pwd`

或者某个比如pcanywhere的密码后缀cif等

再来点更敏感信息 `“robots.txt” “Disallow:” filetype:txt inurl:_vti_cnf` (FrontPage的关键索引啦，扫描器的CGI库一般都有) 

`allinurl: /msadc/Samples/selector/showcode.asp /../../../passwd /examples/jsp/snp/snoop.jsp phpsysinfo intitle:index of /admin intitle:”documetation” inurl: 5800(vnc的端口)`

或者desktop port等多个关键字检索 `webmin port 10000 inurl:/admin/login.asp intext:　Powered by GBook365 intitle:”php shell” “Enable stderr” filetype:php` 直接搜索到phpwebshell

```
foo.org filetype:inc

ipsec filetype:conf

intilte:”error occurred” ODBC request where (select|insert) 说白了就是说，可以直接试着查查数据库检索，针对目前流行的sql注射

“Dumping data for table” username password

intitle:”Error using Hypernews”

“Server Software”

intitle:”HTTP_USER_AGENT=Googlebot”

“HTTP_USER_ANGET=Googlebot” THS ADMIN

filetype:.doc site:.mil classified 直接搜索军方相关word
```
检查多个关键字：
```
intitle:config confixx login password

“mydomain.com” nessus report

“report generated by”

“ipconfig”

“winipconfig”
```

google缓存利用，搜索时候多”选搜索所有网站”

特别推荐：administrator users 等相关的东西，比如名字，生日等……最惨也可以拿来做字典


一些技巧集合：

1) index.of.password

1) filetype:blt “buddylist”

2) “access denied for user” “using password”

2) intitle:”index of” inurl:ftp (pub | incoming)

3) “http://:@www” domainname

3) filetype:cnf inurl:_vti_pvt access.cnf

4) auth_user_file.txt

4) allinurl:”//_vti_pvt/” | allinurl:”//_vti_cnf/”

5) The Master List

5) inurl:”install/install.php”

6) allinurl: admin mdb

6) intitle:”welcome.to.squeezebox”

7) passlist.txt (a better way)

7) intext:””BiTBOARD v2.0″ BiTSHiFTERS Bulletin Board”

8) “A syntax error has occurred” filetype:ihtml

8) intitle:　Login intext:”RT is ? Copyright”

9) “# -FrontPage-” inurl:service.pwd

9) ext:php program_listing intitle:MythWeb.Program.Listing

10) ORA-00921: unexpected end of SQL command

10) intitle:index.of abyss.conf