---
layout: post
title: shell编程基础
tags:
  - shell
categories:
  - linux
abbrlink: 16363
---

### 前言

Shell的概念是源自Unix的命令解释器。Shell不仅可解释用户输入的命令，同时，可解释执行基于命令的脚本语言。使用shell脚本能提高用户操作和管理员进行系统管理的效率。shell脚本擅长处理纯文本类型的数据，而Linux中几乎所有的配置文件、日志都是纯文本类型。

<!--more-->

### 脚本书写规范

```
1.脚本统一存放目录
2.选择解释器, 开头要写#! XXX,内核根据#!后的解释器来确定用哪个解释器解释脚本内容
3.编辑脚本使用vim, 配置~/.vimrc方便个人书写习惯
4.文件名规范，结尾以sh结束
```

### shell的基本元素

```
1.#！/bin/bash 必须的，指出shell的类型
2.# 注释。在shell中，注释写在#之后，#之后的内容不会执行
3.变量
4.控制 循环分支
```

### shell中的特殊符号

```
1.#! 注明执行脚本采用的shell
2.$ 变量符。
与反斜杠转义符相反，使其后的普通字符作为变量名，如$a表示变量a的值。变量
字符长度超过1个时，用{}括起来
3.单引号。
被引起的字符全部做普通字符，即全部原样echo 'my $SHELL'
4.双引号
引号内的内容，除$、转义符\、倒引号`这三个保留特殊功能，其他字符均做普通字符。
5.倒引号(数字1键旁边的那个键）
引号内的字符串当做shell命令行解释执行（同样的功能也可以使用$()来使用），得到的结果
取代整个倒引号括起来的部分。
6.反斜线
反斜线是转义字符，它能把特殊字符变成普通字符。在某个字符前面利用反斜杠（\）能够阻止
shell把后面的字符解释为特殊字符。
7.*  代表0个或者多个特殊字符
例子 yum.* 代表的可以使yum.也可以是yum.a、yum.ab、yum.abc 当然小数点后面可以有多个字母
8.? 代表的是任意一个字符
例子 yum.? 可以是yum.a yum.b yum.c，但是要注意小数点后面必须有任意一个字符
9.[]
代表的是中括号中的任意一个
[abcdef] 可以是a b c d e f 中的任意一个字母当然也可以是数字
[-]代表的是一个范围
[a-z] 表示的是字母a到z之间的所有字母
[^]^是反向选择符号从字面意思可以知道也就是非的意思
[^abc]表示只要不a b c 这三个字符中的任意一个就选择
10.$( )
可以将命令替换输出赋值给变量
11.{}
通过括号扩展可以生成需要的字串，括号中可以包含连续的序列或使用逗号分隔的多个项目，连续的序列包括一个起点和一个终点
user@computer: ~$ echo {a,b,c}
 a b c
user@computer: ~$ echo user{1,5,8}
user1 user5 user8
user@computer: ~$ echo {0..10}
0 1 2 3 4 5 6 7 8 9 10
user@computer: ~$ mkdir {dir1,dir2,dir3}
user@computer: ~$ ls –ld dir{1,2,3}
```


### 变量

shell变量可以保存路径名、文件名或者一个数字等。分为三类：

```
本地变量: (局部变量)只在创建它们的Shell中使用，可以在shell程序内任意使用和修改它
们。
环境变量: 可以在创建它们的Shell及其派生出来的任意子程序中使用。有些变量是用户创建
的，其他的则是专用的（比如PATH、HOME)。是系统环境的一部分，不必去定义它们，可以在
shell程序中使用它们 。还能在shell中加以修改。
内部变量: 由系统提供的。与环境变量不同，用户不能修改它们。
```


#### 本地变量

```
本地变量 在用户现在的shell生命期的脚本中使用
 变量名=值
1.等号两边不可以有空格
2.取值包含空格，必须用双引号括起来
3.Shell变量可以用大小写字母，区分大小写

变量是弱类型的, 不用声明类型

# 变量声明及赋值格式
变量=值（等号两边不能有空格）

# 变量的引用
 $变量名
 ${变量名}
 变量名为1个字符时建议使用方式一，多余一个字符时建议使用方式二
 举例: $a ${abc}

# 清除变量
unset 变量名
user@computer: ~$ name=Jack
user@computer: ~$ echo ${name}
user@computer: ~$ unset name  # 注意,name前没有$

# 设置只读变量
设置变量时，不想再改变其值，可以将之设为只读变量
 变量名=值
 readonly  变量名
```

#### 环境变量

Bash预设了很多环境变量，实际使用中，可以直接调用这些变量。环境变量可以用于所有子程序，着包括编辑器、脚本和应用

内置环境变量

```
HOME: 代表使用者的家目录。cd ~ 去到使用者的家目录 或者利用 cd 就可以直接回到使用者
家目录了。
SHELL: 目前这个环境使用的 SHELL 是哪个程序？ 如果是 bash 的话，预设是 /bin/bash
PWD：用户当前工作目录的路径。它指出用户目前在Linux文件系统中处在什么位置。它是由
Linux自动设置的
HISTSIZE: 这个与“历史命令”有关，曾经下达过的指令可以被系统记录下来，而记录的“数目”
则是由这个值来设定的。
PATH: 就是执行文件搜寻的路径，目录与目录中间以冒号(:)分隔， 由于文件的搜寻是依序由
PATH的变量内的目录来查询，所以，目录的顺序也很重要。
```


```
环境变量可以在命令行中设置，但用户注销时这些值将丢失
   环境变量均为大写
   必须用export命令导出

# 设置环境变量
variable-name=value
export variable-name(环境变量名大写)

# 显示环境变量
env 可以看到所有的环境变量
echo $环境变量名 （显示一个变量）

# 清除环境变量
unset 环境变量名
```


修改path环境变量

```
修改PATH环境变量，使脚本不用加路径，直接输入文件名字即可执行。
# 命令行修改环境变量
以下在用户user主目录下操作：
mkdir shdir && cd shdir
vi hello
chmod 755 hello
cd ～
export PATH=$PATH:$HOME/shdir
在任何目录下，输入hello即可执行该文件。
本方式下环境变量如果修改错了，exit退出后重新登陆即可恢复系统默认的值。
```

配置文件中修改环境变量

```
注意，修改环境变量前最好先备份一下旧的：
export tem=$PATH
echo $tem >>pathbake

需要知道环境变量与哪些配置文件有关：不同发行版会有不同，但命名还是有通性的：
find / -name “*profile”
find / -name “*bashrc”
全局配置文件/etc/profile
本地配置文件~/.bashrc
```


#### 内部变量

内部变量是Linux所提供的一种特殊类型的变量，这类变量在程序中用来作出判断。在shell程序内这类变量的值是不能修改的。

```
部分内部变量是：
$# 传送给shell程序的位置参数的数量
$? 最后命令的完成码或者在shell程序内部执行的shell程序（返回值）
$0 shell程序的名称
$* 调用shell程序时所传送的全部参数的单字符串，"参数1", "参数2"…形式保存的参数
$@ "参数1", "参数2"…形式保存的参数
$n 第n个参数
$$ 本程序的PID
$! 上一个命令的PID
```


### 输入与输出

```
# read 从键盘上读取变量的值
read  [选项]  变量名列表
    常用选项
    -a ANAME   将输入读入ANAME的数组
    -n NCHARS  读入N个字符
    -p PROMPT  显示一个提示
    -r         取消转移
    -s         安静模式，输入的字符将不会提示
    -t TIMEOUT 超过指定时间，read自动停止

# echo 显示字符串或变量的值
echo  [选项]  字符串
    常用选项
    -n  不在最后自动换行
    -e  启用反斜线控制字符的转换
    -E  不处理转义字符。此为缺省选项；
```


```sh
#! /bin/bash
# 输入一句话，打印输入的话
read -p 'please type some words, I will print them: ' words
echo $words
```

### 条件测试

```
# test
test 条件表达式
如果测试条件为真，test命令会返回0，否则返回一个非0的数值
test 语句与if/then和case语句一起，构成shell编程的控制转移结构

# []
[ 条件表达式 ]
方括号的内侧两边各需一个空格

条件表达式的值为真返回零，为假时返回非零值

```


#### 文件状态判断

```
-d filename	若文件filename为目录文件，则返回真

-f filename	文件是否存在且为普通文件，则返回真

-r filename	若文件filename可读，则返回真

-s filename	若文件filename的长度大于0，则返回真

-w filename	若文件filename可写，则返回真

-x filename	若文件filename可执行，则返回真

-e filename	文件是否存在

```


```
#! /bin/bash
# 输入文件的绝对路径，判断文件是否存在
read  -p 'input file path: ' file
if [ -e $file ]
    then
    echo '文件存在'
else
    echo '文件不存在'
fi
```


#### 条件语句

```
if  [ 条件表达式 ]
    then
    命令序列1
else
    命令序列2
fi
当"条件表达式"的测试值为真时，执行"命令序列1"，否则，执行"命令序列2"。命令序列中的命令
可以是一个或者多个。


if [ 条件表达式 ]; then
    命令序列
fi
当"条件表达式"的测试值为真时，执行"命令序列",否则，执行条件语句后面的命令。条件表达
式与then之间的分号";"起命令分隔符的作用。

语法形式三

if test 条件表达式1
    then
    命令序列1
elif [ 条件表达式2 ]
    then
    命令序列2
else
    命令序列3
fi
这是包含二层嵌套的条件语句，当"条件表达式1"为真时，执行"命令序列1",否则，在"条件表
达式2"为真的情况下，执行"命令序列2"，否则，执行"命令序列3","命令序列3"属于第2个条
件语句的一部分。
```

```sh
#! /bin/bash
# 判断输入的路径是文件还是目录
read -p 'please input the file path: ' file
if [ -d $file ]
    then
    echo 'this is a directory'
elif [ -f $file ]
    then
    echo 'this is a file'
else
    echo 'wrong file type, or the file do not exist'
fi
```

#### 数字操作符

```
n1 –eq n2判断数字n1与n2是否相等，若相等，返回0，否则，返回1

n1 –ne n2判断数字n1与n2是否不等，若不等，返回0，否则，返回1

n1 –lt n2判断数字n1是否小于n2，若是，返回0，否则，返回1

n1 –gt n2判断数字n1是否大于n2，若是，返回0，否则，返回1

n1 –le n2判断数字n1是否小于或等于n2，若是，返回0，否则，返回1

n1 –ge n2 判断数字n1是否大于或等于n2，若是，返回0，否则，返回1

```

#### 字符串操作符

```
string 若字符串string非空，则返回真


-n string 若字符串string长度大于0，则返回真


-z string 若字符串string长度为0，则为返回真


string1 = string2 若字符串string1和string2相等，则返回真


string1 != string2 若字符串string1和 string2不等，则返回真

```

#### 逻辑操作符

```
e1 –a e2	逻辑表达式e1和e2同时为真时，返回0，否则，返回1
e1 –o e2	逻辑表达式e1和e2有一个为真时，返回0，否则，返回1
! e1	    若逻辑表达式e1不为真时，返回0，,否则，返回1
```

### 循环语句

```
for 变量名 in 参数列表
    do
    命令列表
done
将"参数列表"中的元素依次赋给"变量名"，在每次赋值后执行"命令列表"，"参数列表"表示"变
量名"的取值范围

for ((初始化变量值；结束循环条件；运算))
    do
    命令序列
done

while [ 条件表达式 ]
    do
    命令列表
done
循环执行"命令列表"中的命令，直至"条件表达式"的值为假。

Until [ 条件 ]
    do
    命令序列
Done
直到条件满足时循环结束

```


```sh
#! /bin/bash
# 将指定目录下(参数传递$1)的所有以.txt为后缀的文件更名为*.doc
directory=$1
if [ ! $directory ]
  then
  echo "please input the argument directory"
  exit
fi
files=`ls ${directory}`
for file in $files;
  do
  if [ -f ${file} ]
    echo $file
    then
    suffix=${file#*\.}
    echo $suffix
    if [[ $suffix == "txt" ]]
      then
      prefix=${file%\.*}
      mv $directory/$file $directory/$prefix.doc
    fi
  fi
done
```

### 函数

```bash
functionname() {
    命令列表
    return
}
函数的调用方式为：
functionname arguments
```


