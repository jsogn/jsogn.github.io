---
layout: post
title: Linux文件访问控制列表
tags:
  - linux
categories:
  - linux
abbrlink: 36849
---

### 前言

如果希望对某个指定的用户进行单独的权限控制，就需要用到文件的访问控制列表（ACL）了。通俗来讲，基于普通文件或目录设置ACL其实就是针对指定的用户或用户组设置文件或目录的操作权限。另外，如果针对某个目录设置了ACL，则目录中的文件会继承其ACL；若针对文件设置了ACL，则文件不再继承其所在目录的ACL。

<!--more-->

### setfacl 命令

setfacl命令用于管理文件的ACL规则，格式为“setfacl [参数] 文件名称”。

文件的ACL提供的是在所有者、所属组、其他人的读/写/执行权限之外的特殊权限控制，使用setfacl命令可以针对单一用户或用户组、单一文件或目录来进行读/写/执行权限的控制。

其中，针对目录文件需要使用-R递归参数；针对普通文件则使用-m参数；如果想要删除某个文件的ACL，则可以使用-b参数。

设置用户在/root目录上的权限：

```
[root@linuxprobe ~]# setfacl -Rm u:linuxprobe:rwx /root
[root@linuxprobe ~]# su - linuxprobe
Last login: Sat Mar 21 15:45:03 CST 2017 on pts/1
[linuxprobe@linuxprobe ~]$ cd /root
[linuxprobe@linuxprobe root]$ ls
anaconda-ks.cfg Downloads Pictures Public
[linuxprobe@linuxprobe root]$ cat anaconda-ks.cfg
[linuxprobe@linuxprobe root]$ exit
```

常用的ls命令是看不到ACL表信息的，但是却可以看到文件的权限最后一个点（.）变成了加号（+）,这就意味着该文件已经设置了ACL了。


```
[root@linuxprobe ~]# ls -ld /root
dr-xrwx---+ 14 root root 4096 May 4 2017 /root
```


### getfacl 命令

getfacl命令用于显示文件上设置的ACL信息，格式为“getfacl 文件名称”。

想要设置ACL，用的是setfacl命令；要想查看ACL，则用的是getfacl命令。

使用getfacl命令显示在root管理员家目录上设置的所有ACL信息。

```
[root@linuxprobe ~]# getfacl /root
getfacl: Removing leading '/' from absolute path names
# file: root
# owner: root
# group: root
user::r-x
user:linuxprobe:rwx
group::r-x
mask::rwx
other::---
```

