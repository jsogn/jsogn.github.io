---
layout: post
title: Linux文件的隐藏属性
tags:
  - linux
categories:
  - linux
abbrlink: 1747
---

### 前言

Linux系统中的文件除了具备一般权限和特殊权限之外，还有一种隐藏权限，即被隐藏起来的权限，默认情况下不能直接被用户发觉。明明权限充足但却无法删除某个文件的情况，或者仅能在日志文件中追加内容而不能修改或删除内容，这在一定程度上阻止了黑客篡改系统日志的图谋，因此这种“奇怪”的文件也保障了Linux系统的安全性。

<!--more-->

### chattr 命令

chattr命令用于设置文件的隐藏权限，格式为“chattr [参数] 文件”。如果想要把某个隐藏功能添加到文件上，则需要在命令后面追加“+参数”，如果想要把某个隐藏功能移出文件，则需要追加“-参数”

共有以下8种模式：

- a：让文件或目录仅供附加用途；
- b：不更新文件或目录的最后存取时间；
- c：将文件或目录压缩后存放；
- d：将文件或目录排除在倾倒操作之外；
- i：不得任意更动文件或目录；
- s：保密性删除文件或目录；
- S：即时更新文件或目录；
- u：预防意外删除。


选项：


- -R：递归处理，将指令目录下的所有文件及子目录一并处理；
- -v<版本编号>：设置文件或目录版本；
- -V：显示指令执行过程；
- +<属性>：开启文件或目录的该项属性；
- -<属性>：关闭文件或目录的该项属性；
- =<属性>：指定文件或目录的该项属性。


新建一个普通文件，并为其设置不允许删除与覆盖（+a参数）权限，然后再尝试将这个文件删除：

```
[root@linuxprobe ~]# echo "for Test" > linuxprobe
[root@linuxprobe ~]# chattr +a linuxprobe
[root@linuxprobe ~]# rm linuxprobe
rm: remove regular file ‘linuxprobe’? y
rm: cannot remove ‘linuxprobe’: Operation not permitted
```

### lsattr 命令

lsattr命令用于显示文件的隐藏权限，格式为“lsattr [参数] 文件”。

- -E：可显示设备属性的当前值，但这个当前值是从用户设备数据库中获得的，而不是从设备直接获得的。
- -D：显示属性的名称，属性的默认值，描述和用户是否可以修改属性值的标志。
- -R：递归的操作方式；
- -V：显示指令的版本信息；
- -a：列出目录中的所有文件，包括隐藏文件。


lsattr经常使用的几个选项-D，-E，-R这三个选项不可以一起使用，它们是互斥的，经常使用的还有-l,-H，使用lsattr时，必须指出具体的设备名，用-l选项指出要显示设备的逻辑名称，否则要用-c，-s，-t等选项唯一的确定某个已存在的设备。

在Linux系统中，文件的隐藏权限必须使用lsattr命令来查看，平时使用的ls之类的命令则看不出端倪

一旦使用lsattr命令后，文件上被赋予的隐藏权限马上就会原形毕露。此时可以按照显示的隐藏权限的类型（字母），使用chattr命令将其去掉：

```
[root@linuxprobe ~]# lsattr linuxprobe
-----a---------- linuxprobe
[root@linuxprobe ~]# chattr -a linuxprobe
[root@linuxprobe ~]# lsattr linuxprobe 
---------------- linuxprobe
[root@linuxprobe ~]# rm linuxprobe 
rm: remove regular file ‘linuxprobe’? y
```



