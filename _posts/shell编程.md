---
title: shell编程
date: 2023-02-03
categories:
- 服务器
tags: 
- shell
---





> 熟练掌握shell的语法, 写出优雅的脚本



## 一 : 多命令执行符

| 多命令执行符 | 格式           | 作用                                                         |
| ------------ | -------------- | ------------------------------------------------------------ |
| `;`          | 命令1`;`命令2  | 多个命令顺序执行, 命令之间没有任何逻辑联系                   |
| `&&`         | 命令1`&&`命令2 | 当命令1正确执行($?=0), 则命令2才会执行; 当命令1执行不正确($? ≠ 0), 则命令2不会执行 |
| `||`         | 命令1`||`命令2 | 当命令1执行不正确($?≠0), 则命令2才会执行; 当命令1正确执行($?=0), 则命令2不会执行 |

示例

```shell
# 判断命令是否正确执行
[root@localhost ~]# ls && echo yes || echo no
```



## 二 : 数值运算的方法

在 linux 中, 所有变量的默认类型是字符串型, 如果我需要进行数值运算, 可以采用的方式有三种

**推荐使用 `$((运算式))` 的方式**

### (一) `$((运算式))`或`$[运算式]`

代码示例

```shell
# 变量 ff 的值是 aa 和 bb 的和
[root@localhost ~]# aa=11
[root@localhost ~]# bb=22
[root@localhost ~]# ff=$(( $aa+$bb ))
[root@localhost ~]# echo $ff
33

# 变量 gg 的值是 aa 和 bb 的和
[root@localhost ~]# gg=$[ $aa+$bb ]
[root@localhost ~]# echo $gg
33
```

### (二) 数值运算工具 expr 或 let

**expr** 命令要求**运算符左右两侧必须有空格**, 否则运算不执行

```shell
# 给变量aa和变量bb赋值
[root@localhost ~]# aa=11
[root@localhost ~]# bb=22

# dd的值是aa和bb的和。注意"+"号左右两侧必须有空格
[root@localhost ~]# dd=$(expr $aa + $bb)

[root@localhost ~]# echo $dd 
33
```

**let** 命令对格式要求比较宽松, 所以**推荐使用 let 命令进行数值运算**

```shell
#给变量 aa 和变量 bb 赋值
[root@localhost ~]# aa=11
[root@localhost ~]# bb=22

#变量 ee 的值是 aa 和 bb 的和
[root@localhost ~]# let ee=$aa+$bb
[root@localhost ~]# echo $ee
33

#定义变量 n
[root@localhost ~]# n=20

#变量 n 的值等于变量本身再加 1
[root@localhost ~]# let n+=1
[root@localhost ~]# echo $n
21
```

### (三) 声明变量类型 `declare`

既然所有变量的默认类型是字符串型, 那么只要把变量声明为整数型不就可以了吗? 使用 `declare` 命令就可以实现声明变量的类型

```shell
[root@localhost ~]# declare [+/-][选项] 变量名
选项:
    -	: 给变量设定类型属性
    +	: 取消变量的类型属性
    
    -a	: 将变量声明为数组型(array)
    -i	: 将变量声明为整数型(integer) 
    -r	: 讲变量声明为只读变量(readonly)。
        注意, 一旦设置为只读变量, 既不能修改变量的值, 也不能删除变量, 甚至不能通过+r 取消只读属性
    -x	: 将变量声明为环境变量
    -p	: 显示指定变量的被声明的类型
```

代码示例, 了解即可

```shell
# 给变量 aa 和 bb 赋值
[root@localhost ~]# aa=11
[root@localhost ~]# bb=22
# 声明变量 cc 的类型是整数型, 它的值是 aa 和 bb 的和
[root@localhost ~]# declare -i cc=$aa+$bb
# 这下终于可以相加了
[root@localhost ~]# echo $cc
33
```



## 二 : 条件判断 test

### (一) 文件类型

相关参数 

| 选项          | 英文      | 作用                                                         |
| ------------- | --------- | ------------------------------------------------------------ |
| `-b` 文件     | block     | 判断该文件是否存在, 并且是否为块设备文件(是块设备文件为真)   |
| `-c` 文件     | character | 判断该文件是否存在, 并且是否为字符设备文件(是字符设备文件为真) |
| **`-d` 文件** | directory | 判断该文件是否存在, 并且是否为目录文件(**是目录为真**)       |
| **`-e` 文件** | exists    | 判断该文件是否存在(**存在为真**)                             |
| **`-f` 文件** | file      | 判断该文件是否存在, 并且是否为普通文件(**是普通文件为真**)   |
| **`-L` 文件** | link      | 判断该文件是否存在, 并且是否为符号链接文件(**是符号链接文件为真**) |
| `-p` 文件     | pipe      | 判断该文件是否存在, 并且是否为管道文件(是管道文件为真)       |
| **`-s` 文件** | size > 0  | 判断该文件是否存在, 并且是否为非空(**非空为真**)             |
| `-S` 文件     | socket    | 判断该文件是否存在, 并且是否为套接字文件(是套接字文件为真)   |

代码示例

```shell
# 命令格式
[root@localhost ~]# test -e /root/sh/
# 通常我们喜欢使用另一种格式,而且更通用,如下
[root@localhost ~]# [ -e /root/sh/ ]
[root@localhost ~]# echo $?
0 #判断结果为 0,/root/sh/目录是存在的

[root@localhost ~]# [ -e /root/test ]
[root@localhost ~]# echo $? 
1 #在/root/下并没有test文件或目录,所以"$?"的返回值为非零

# 也可以用&&和||
[root@localhost ~]# [ -d /root/sh ] && echo "yes" || echo "no" 
```



### (二) 文件权限

相关参数 

| 选项          | 作 用                                                        |
| ------------- | ------------------------------------------------------------ |
| **`-r` 文件** | 判断该文件是否存在, 并且是否该文件拥有读权限(有读权限为真)   |
| **`-w` 文件** | 判断该文件是否存在, 并且是否该文件拥有写权限(有写权限为真)   |
| **`-x` 文件** | 判断该文件是否存在, 并且是否该文件拥有执行权限(有执行权限为真) |
| `-u` 文件     | 判断该文件是否存在, 并且是否该文件拥有 SUID 权限(有 SUID 权限为真) |
| `-g` 文件     | 判断该文件是否存在, 并且是否该文件拥有 SGID 权限(有 SGID 权限为真) |
| `-k` 文件     | 判断该文件是否存在, 并且是否该文件拥有 SBit 权限(有 SBit 权限为真) |

代码示例

```shell
[root@localhost ~]# ll null.txt 
-rw-r--r-- 1 root root 0 Nov 16 18:45 null.txt

#判断文件是拥有写权限的
[root@localhost ~]# [ -w null.txt ] && echo "yes" || echo "no" 
yes
```

### (三) 文件比较

如何进行两个文件之间的比较

| 测试选项              | 英文  | 作用                                                         |
| --------------------- | ----- | ------------------------------------------------------------ |
| 文件1 `-nt` 文件2     | new   | 判断文件1的修改时间是否比文件2的新(如果新则为真)             |
| 文件1 `-ot` 文件2     | old   | 判断文件1的修改时间是否比文件2的旧(如果旧则为真)             |
| **文件1 `-ef` 文件2** | equal | 判断文件1是否和文件2的Inode号一致,可以理解为两个文件是否为同一个文件(**硬链接**) |

代码示例 : 判断两个文件是否是硬链接呢

```shell
#创建个硬链接吧
[root@localhost ~]# ln /root/student.txt /tmp/stu.txt
#用 test 测试下,果然很有用
[root@localhost ~]# [ /root/student.txt -ef /tmp/stu.txt ] && echo "yes" || echo "no" 
yes
```

### (四) 整数比较

如何在两个整数之间进行比较

| 测试选项          | 作用                                     |
| ----------------- | ---------------------------------------- |
| 整数1 `-eq` 整数2 | 判断整数1是否和整数2相等(相等为真)       |
| 整数1 `-ne` 整数2 | 判断整数1是否和整数2不相等(不相等位置)   |
| 整数1 `-gt` 整数2 | 判断整数1是否大于整数2(大于为真)         |
| 整数1 `-lt` 整数2 | 判断整数1是否小于整数2(小于位置)         |
| 整数1 `-ge` 整数2 | 判断整数1是否大于等于整数2(大于等于为真) |
| 整数1 `-le` 整数2 | 判断整数1是否小于等于整数2(小于等于为真) |

代码示例

```shell
#判断 23 是否大于等于 22,当然是了
[root@localhost ~]# [ 23 -ge 22 ] && echo "yes" || echo "no" 
yes
#判断 23 是否小于等于 22,当然不是了
[root@localhost ~]# [ 23 -le 22 ] && echo "yes" || echo "no" 
no
```

### (五) 字符串判断

字符串的判断

| 测试选项         | 英文        | 作用                                     |
| ---------------- | ----------- | ---------------------------------------- |
| `-z` 字符串      | length zero | 判断字符串是否为空(空为真)               |
| `-n` 字符串      | nonzero     | 判断字符串是否为非空(非空为真)           |
| 字串1 `==` 字串2 |             | 判断字符串1是否和字符串2相等(相等为真)   |
| 字串1 `!=` 字串2 |             | 判断字符串1是否和字符串2不相等(不等为真) |

代码示例

```shell
# 判断name变量是否为空
[root@localhost ~]# [ -z "$name" ] && echo "yes" || echo "no" 

# 给变量 aa 和变量 bb 赋值
[root@localhost ~]# [ "$aa" == "$bb" ] && echo "yes" || echo "no" 
```

### (六) 多重条件判断

多重条件判断是什么样子的

| 测试选项         | 作用                                             |
| ---------------- | ------------------------------------------------ |
| 判断1 `-a` 判断2 | 逻辑与, 判断1和判断2都成立, 最终的结果才为真     |
| 判断1 `-o` 判断2 | 逻辑或, 判断1和判断2有一个成立, 最终的结果就为真 |
| `!` 判断         | 逻辑非, 使原始的判断式取反                       |

代码示例 : 

```shell
# 判断变量aa是否为空且是否大于 23
[root@localhost ~]# [ -n "$aa" -a "$aa" -gt 23 ] && echo "yes" || echo "no"

# 本来"-n"选项是变量aa不为空,加"!"后相当于"-z"
[root@localhost ~]# [ ! -n "$aa" ] && echo "yes" || echo "no" 
```

- **注意 : `!` 和 `-n` 之间必须加入空格,否则会报错的。**



## 三 : 条件判断

### (一) 单分支 if 

单分支条件语句最为简单, 就是只有一个判断条件, 如果符合条件则执行某个程序, 否则什么事情都不做; 语法如下 : 

```shell
if [ 条件判断式 ];then
	程序
fi
```

需要注意几个点 : 

- **`if` 语句使用 `fi` 结尾**, 和一般语言使用大括号结尾不同

- `[ 条件判断式 ]` 就是使用 `test` 命令判断,所以**中括号和条件判断式之间必须有空格**

- `then` 后面跟符合条件之后执行的程序, 可以放在 `[]` 之后用 `;` 分割。**也可以换行写入, 就不需要 `;` 了**, 比如单分支 if 语句还可以这样写 : 

  ```shell
  if [ 条件判断式 ]
      then
    		程序
  fi
  ```

代码示例

```shell
[root@localhost ~]# vi sh/if1.sh

#!/bin/bash
# 统计根分区使用率

#把根分区使用率作为变量值赋予变量 rate
rate=$(df -h | grep "/dev/vda1" | awk '{print $5}' | cut -d "%" -f 1)
#判断 rate 的值如果大于等于 80,则执行 then 程序
if [ $rate -ge 80 ]
    then 
    	echo "Warning! /dev/sda3 is full!!"
fi
```

### (二) 双分支 if 条件语句

语法格式

```shell
if [ 条件判断式 ]
    then
    	条件成立时,执行的程序
    else
    	条件不成立时,执行的另一个程序
fi
```

代码示例 : 我们写一个数据备份的例子, 来看看双分支 if 条件语句。

```shell
# 备份 mysql 数据库
[root@localhost ~]# vi sh/bakmysql.sh
#!/bin/bash
#备份 mysql 数据库

#同步系统时间
ntpdate asia.pool.ntp.org &>/dev/null
#把当前系统时间按照"年月日"格式赋予变量 date
date=$(date +%y%m%d)
#统计 mysql 数据库的大小,并把大小赋予 size 变量
size=$(du -sh /var/lib/mysql)
#判断备份目录是否存在,是否为目录
if [ -d /tmp/dbbak ]
	#如果判断为真,执行以下脚本
    then
    	#把当前日期写入临时文件
        echo "Date : $date!" > /tmp/dbbak/dbinfo.txt
        #把数据库大小写入临时文件
        echo "Data size : $size" >> /tmp/dbbak/dbinfo.txt
        #进入备份目录
        cd /tmp/dbbak
        #打包压缩数据库与临时文件,把所有输出丢入垃圾箱(不想看到任何输出)
        tar -zcf mysql-lib-$date.tar.gz /var/lib/mysql dbinfo.txt &>/dev/null
        #删除临时文件
        rm -rf /tmp/dbbak/dbinfo.txt
    else
    	#如果判断为假,则建立备份目录
        mkdir /tmp/dbbak
        #把日期和数据库大小保存如临时文件
        echo "Date : $date!" > /tmp/dbbak/dbinfo.txt
        echo "Data size : $size" >> /tmp/dbbak/dbinfo.txt
        #压缩备份数据库与临时文件
        cd /tmp/dbbak
        tar -zcf mysql-lib-$date.tar.gz dbinfo.txt /var/lib/mysql &>/dev/null
        #删除临时文件
        rm -rf /tmp/dbbak/dbinfo.txt
fi
```

* **注意 :** 解释一下 **`&>/dev/null`** 这个命令, `&>` 输出, `/dev/null` 这个类似回收站, 任何东西丢到这里面都会消失, 所以通常写脚本的时候, 我们习惯加上 `&>/dev/null`, **用于屏蔽命令的提示信息**

实例 : 在工作当中,服务器上的服务经常会宕机。如果我们对服务器监控不好,就会造成服务器中服务宕机了,而管理员却不知道的情况, 这时我们可以写一个脚本来监听本机的服务,如果服务停止或宕机了,可以自动重启这些服务。我们拿 apache 服务来举例 : 

```shell
# 判断 apache 是否启动,如果没有启动则自动启动
[root@localhost ~]# vi sh/autostart.sh

#!/bin/bash
#判断 apache 是否启动,如果没有启动则自动启动

# Author: Bob (E-mail: Bob@163.com)

#使用nmap命令扫描服务器公网ip,并截取 apache 服务的状态,赋予变量 port
port=$(nmap -sT 192.168.4.210 | grep tcp | grep http | awk '{print $2}')
#如果变量 port 的值是"open"
if [ "$port" == "open" ]
	then
		#则证明 apache 正常启动,在正常日志中写入一句话即可
		echo "$(date) httpd is ok!" >> /tmp/autostart-acc.log
	else
		#否则证明 apache 没有启动,自动启动 apache
		/etc/rc.d/init.d/httpd start &>/dev/null
		#并在错误日志中记录自动启动 apche 的时间
		echo "$(date) restart httpd !!" >> /tmp/autostart-err.log
fi
```

- 我们使用 `nmap` 端口扫描命令, 它的原理是给指定服务器所有的端口发送请求, 看它是否回复, `nmap` 命令格式如下 : 

  ```shell
  [root@localhost ~]# nmap -sT 域名或IP(一般用公网IP)
  选项 : 
      -s 扫描
      -T 扫描所有开启的 TCP 端口
  ```

- 这条命令的执行结果如下 : 

  ```shell
  # 可以看到这台服务器开启了如下的服务
  [root@localhost ~]# nmap -sT 192.168.4.210
  Starting Nmap 5.51 ( http://nmap.org ) at 2018-11-25 15:11 CST
  Nmap scan report for 192.168.4.210
  Host is up (0.0010s latency).
  Not shown: 994 closed ports
  PORT STATE SERVICE
  22/tcp open ssh
  80/tcp open http #apache 的状态是 open
  111/tcp open rpcbind
  139/tcp open netbios-ssn
  445/tcp open microsoft-ds
  3306/tcp open mysql
  Nmap done: 1 IP address (1 host up) scanned in 0.49 seconds
  ```

- 知道了 `nmap` 命令的用法,我们在脚本中使用的命令就是为了截取 `http` 的状态,只要状态是 `open` 就证明 apache 启动正常,否则证明 apache 启动错误。来看看脚本中命令的结果 : 

  ```shell
  # 扫描指定计算机,提取包含 tcp 的行,在提取包含 httpd 的行,截取第二列
  [root@localhost ~]# nmap -sT 192.168.4.210 | grep tcp | grep http | awk '{print $2}'
  open
  # 把截取的值赋予变量 port
  ```

### (三) 多分支 if 条件语句

语法格式

```shell
if [ 条件判断式 1 ]
    then
    	当条件判断式 1 成立时,执行程序 1
elif [ 条件判断式 2 ]
    then
        当条件判断式 2 成立时,执行程序 2 
# …省略更多条件…
else
	当所有条件都不成立时,最后执行此程序
fi
```

代码示例 : 判断用户输入的是什么文件

```shell
[root@localhost ~]# vi sh/if-elif.sh
#!/bin/bash
#判断用户输入的是什么文件

#接收键盘的输入,并赋予变量 file
read -p "Please input a filename: " file

#判断 file 变量是否为空
if [ -z "$file" ]
	then
        #如果为空,执行程序 1,也就是输出报错信息
        echo "Error,please input a filename"
        #退出程序,并返回值为 1(把返回值赋予变量$?)
		exit 1
#判断 file 的值是否存在
elif [ ! -e "$file" ]
	then
		#如果不存在,则执行程序 2
		echo "Your input is not a file!"
        #退出程序,把并定义返回值为 2
        exit 2
#判断 file 的值是否为普通文件
elif [ -f "$file" ]
	then
		#如果是普通文件,则执行程序 3
        echo "$file is a regulare file!"
#判断 file 的值是否为目录文件
elif [ -d "$file" ]
	then
		#如果是目录文件,则执行程序 4
		echo "$file is a directory!"
else
	#如果以上判断都不是,则执行程序 5
	echo "$file is an other file!"
fi
```

### (四) case 条件语句

`case` 语句和 `if…elif…else` 语句一样都是多分支条件语句, 不过和 `if` 多分支条件语句不同的是, `case` 语句只能判断一种条件关系, 而 `if` 语句可以判断多种条件关系。case 语句语法如下 : 

```shell
case $变量名 in
    "值1")
    如果变量的值等于值 1,则执行程序 1
    ;;
    "值2")
    如果变量的值等于值 2,则执行程序 2
    ;;
    …省略其他分支…
     *)
    如果变量的值都不是以上的值,则执行此程序
    ;;
esac
```

**注意**以下内容 : 

- `case` 语句, 会取出变量中的值, 然后与语句体中的值逐一比较; 如果数值符合, 则执行对应的程序, 如果数值不符, 则依次比较下一个值。如果所有的值都不符合,则执行 `*)` 中的程序;  `*`  代表所有其他值
- `case` 语句以 `case` 开头, 以 `esac` 结尾; 每一个分支程序之后要通过 `;;` 双分号结尾, 代表该程序段结束(千万不要忘记)。

代码示例

```shell
[root@localhost ~]# vi sh/case.sh
#!/bin/bash
#判断用户输入

#在屏幕上输出"请选择 yes/no",然后把用户选择赋予变量 cho
read -p "Please choose yes/no: " -t 30 cho
#判断变量 cho 的值
case $cho in
	#如果是 yes
    "yes")
    	#执行程序 1
        echo "Your choose is yes!"
        ;;
	#如果是 no
    "no")
		#执行程序 2
        echo "Your choose is no!"
        ;;
	#如果既不是 yes,也不是 no
    *)
    	#则执行此程序
        echo "Your choose is error!"
        ;;
esac
```

## 四 : 循环

### (一) for 循环

`for` 循环是固定循环, 也就是在循环时已经知道需要进行几次的循环, 有时也把 `for` 循环称为**计数循环**。for 的语法有两种 : 

**语法一**

```shell
for 变量 in 值1 值2 值3…
    do
    	程序
    done
```

- 这种语法中 for 循环的次数, 取决于 in 后面值的个数(空格分隔), 有几个值就循环几次, 并且每次循环都把值赋予变量。也就是说,假设 in 后面有三个值, 就会循环三次, 第一次循环会把值1赋予变量, 第二次循环会把值2赋予变量, 以此类推。

**语法二**

```shell
for (( 初始值;循环控制条件;变量变化 ))
    do
    	程序
    done
```

语法二中**需要注意** : 
- 初始值 : 在循环开始时, 需要给某个变量赋予初始值, 如 i=1;  
- 循环控制条件 : 用于指定变量循环的次数,如 i<=100, 则只要 i 的值小于等于 100, 循环就会继续; 
- 变量变化 : 每次循环之后, 变量该如何变化,如 i=i+1; 代表每次循环之后, 变量 i 的值都加 1。

代码示例

```shell
[root@localhost ~]# vi sh/for.sh
#!/bin/bash
# 打印时间
for time in morning noon afternoon evening
	do
		echo "This time is $time!"
	done
```

```shell
[root@localhost ~]# vi sh/auto-tar.sh
#!/bin/bash
# 批量解压缩脚本

# 进入压缩包目录
cd /lamp
# 列出.tar.gz的值,有多少个文件,就会循环多少次,每次循环把文件名赋予变量i
for i in $(ls *.tar.gz)
	do
		#解压缩,并把所有输出都丢弃
		tar -zxf $i &>/dev/null
	done
```

```shell
#!/bin/bash
#从1加到100

s=0
# 定义循环 100 次
for (( i=1;i<=100;i=i+1 ))
    do
    	# 每次循环给变量 s 赋值
        s=$(( $s+$i ))
    done
# 输出1加到100的和
echo "The sum of 1+2+...+100 is : $s"
```

```shell
[root@localhost ~]# vi useradd.sh
#!/bin/bash
# 批量添加指定数量的用户

# 让用户输入用户名,把输入保存入变量 name
read -p "Please input user name: " -t 30 name
# 让用户输入添加用户的数量,把输入保存入变量 num
read -p "Please input the number of users: " -t 30 num 
# 让用户输入初始密码,把输入保存如变量 pass
read -p "Please input the password of users: " -t 30 pass

# 判断三个变量不为空
if [ ! -z "$name" -a ! -z "$num" -a ! -z "$pass" ]
    then
    	# 定义变量的值为后续命令的结果
    	# 后续命令作用是,把变量num的值替换为空。如果能替换为空,证明 num 的值为数字
    	# 如果不能替换为空,证明 num 的值为非数字。我们使用这种方法判断变量 num 的值为数字
    	y=$(echo $num | sed 's/[0-9]//g')
        # 如果变量 y 的值为空,证明 num 变量是数字
        if [ -z "$y" ]
            then
                # 循环 num 变量指定的次数
                for (( i=1;i<=$num;i=i+1 ))
                    do 
                        # 添加用户,用户名为变量 name 的值加变量 i 的数字
                        /usr/sbin/useradd $name$i &>/dev/null
                        # 给用户设定初始密码为变量 pass 的值
                        echo $pass | /usr/bin/passwd --stdin $name$i &>/dev/null
                    done
        fi 
fi
```

```shell
[root@localhost ~]# vi sh/userdel.sh
#!/bin/bash
#批量删除用户

#读取用户信息文件,提取可以登录用户,排除root用户,截取第一列就是用户名
user=$(cat /etc/passwd | grep "/bin/bash" | grep -v "root" |cut -d ":" -f 1)
#循环,有多少个普通用户,循环多少次
for i in $user
    do 
    	#每次循环,删除指定普通用户
        userdel -r $i
    done
```

### (二) while 循环

对 `while` 循环来讲, 只要条件判断式成立, 循环就会一直继续, 直到条件判断式不成立, 循环才会停止

```shell
while [ 条件判断式 ]
    do
    	程序
    done
```

代码示例 : 1 加到 100

```shell
#!/bin/bash
# 从 1 加到 100

# 给变量 i 和变量 s 赋值
i=1
s=0
# 如果变量 i 的值小于等于 100,则执行循环
while [ $i -le 100 ]
    do
        s=$(( $s+$i ))
        i=$(( $i+1 ))
    done
echo "The sum is: $s" 
```

### (三) until 循环

和 `while` 循环相反, `until` 循环时**只要条件判断式不成立则进行循环**, 并执行循环程序; 一旦循环条件成立,则终止循环

```shell
until [ 条件判断式 ]
    do
   		程序
    done
```

还是写从 1 加到 100 这个例子, 注意和 while 循环的区别 : 

```shell
# 从 1 加到 100
[root@localhost ~]# vi sh/until.sh 
#!/bin/bash
#从 1 加到 100

#给变量i和s赋值
i=1
s=0
#循环直到变量i的值大于100,就停止循环
until [ $i -gt 100 ]
    do
        s=$(( $s+$i ))
        i=$(( $i+1 ))
    done
echo "The sum is: $s"
```



## 五 : 退出

### (一) exit 语句

系统是有 `exit` 命令的,用于退出当前用户的登录状态。可是在 Shell 脚本中, `exit` 语句是用来退出当前脚本的。也就是说, 在 Shell 脚本中, 只要碰到了 `exit` 语句, 后续的程序就不再执行, 而直接退出脚本。

exit 的语法如下 : 

```shell
exit [返回值]
```

- 如果 `exit` 命令之后定义了返回值, 那么这个脚本执行之后的返回值就是我们自己定义的返回值; 可以通过查询 `$?` 这个变量来查看返回值; 如果 exit 之后没有定义返回值, 脚本执行之后的返回值是执行 exit 语句之前, 最后执行的一条命令的返回值。


代码示例 

```shell
[root@localhost ~]# vi sh/exit.sh

#!/bin/bash

#接收用户的输入,并把输入赋予num
read -p "Please input a number: " -t 30 num 

#如果变num的值是数字,则把num的值替换为空,否则不替换
#把替换之后的值赋予变量 y
y=$(echo $num | sed 's/[0-9]//g')
#判断变量 y 的值如果不为空,输出报错信息,退出脚本,退出返回值为 18
[ -n "$y" ] && echo "Error! Please input a number!" && exit 18
#如果没有退出脚本,则打印变量 num 中的数字
echo "The number is: $num"
```

* **注意 : 这里的字符串变量要用 `""` 引一下**

### (二) break 语句

再来看看特殊流程控制语句 `break` 的作用, 当程序执行到 `break` 语句时, **会结束整个当前循环**; 而 `continue` 语句也是结束循环的语句, 不过 `continue` 语句单次当前循环, 而下次循环会继续。

代码示例 : 

```shell
[root@localhost ~]# vi sh/break.sh
#!/bin/bash
#演示 break 跳出循环

#循环十次
for (( i=1;i<=10;i=i+1 ))
    do 
    	#如果变量 i 的值等于 4
        if [ "$i" -eq 4 ]
            then
            	#退出整个循环
                break
        fi 
        echo $i
        #输出变量 i 的值
    done
```

测试结果 : 输出1,2,3后停止脚本

### (三) continue 语句

再来看看 continue 语句, continue 也是结束流程控制的语句; 如果在循环中, `continue` 语句只会**结束单次当前循环**

还是用刚刚的脚本,不过退出语句换成 `continue` 语句,看看会发生什么情况 : 

```shell
[root@localhost ~]# vi sh/continue.sh
#!/bin/bash
#演示 continue 语句

for (( i=1;i<=10;i=i+1 ))
    do 
        if [ "$i" -eq 4 ] 
            then
            	#退出语句换成 continue
                continue
        fi 
        echo $i
    done
```

测试结果 : 只有4不会被输出



## 六 : 函数

语法格式

```shell
function 函数名 () {
	程序
}
```

代码示例

```sh
[root@localhost ~]# vi sh/function.sh
#!/bin/bash
#接收用户输入的数字,然后从1加到这个数字

#定义函数 sum
function sum () {
    s=0 
    #循环直到i大于$1为止。$1是函数sum的第一个参数
    #在函数中也可以使用位置参数变量,不过这里的$1指的是函数的第一个参数
    for (( i=0;i<=$1;i=i+1 ))
        do 
        	s=$(( $i+$s ))
        done
	#输出1加到$1 的和
    echo "The sum of 1+2+3...+$1 is : $s"
}

#接收用户输入的数字,并把值赋予变量num
read -p "Please input a number: " -t 30 num 
#把变量num的值替换为空,并赋予变量y
y=$(echo $num | sed 's/[0-9]//g')
#判断变量y是否为空,以确定变量 num 中是否为数字
if [ -z "$y" ]
    then
    	#调用sum函数,并把变量num的值作为第一个参数传递给 sum 函数
        sum $num
else
	#如果变量 num 的值不是数字,则输出报错信息
	echo "Error!! Please input a number!"
fi
```



## 七 : 接收键盘输入

用来从标准输入读取单行数据; 这个命令可以用来读取键盘输入, 当使用重定向的时候, 可以读取文件中的一行数据; 命令格式 : 

```shell
[root@localhost ~]# read [选项] [变量名]
选项:
    -p "message": 在等待 read 输入时, 输出提示信息(message)
    -t second	: read 命令会一直等待用户输入, 使用此选项可以指定等待时间(秒数)
    -n number	: read 命令只接受指定的字符数(number), 就会执行
    -s 			: 隐藏输入的数据, 适用于机密信息的输入
变量名:
    变量名可以自定义, 如果不指定变量名, 会把输入保存入默认变量 REPLY
    如果只提供了一个变量名, 则整个输入行赋予该变量
    如果提供了一个以上的变量名, 则输入行分为若干字, 一个接一个地赋予各个变量, 而命令行上的最后一个变量取得剩余的所有字
```

还是写个例子来解释下 `read` 命令:

```shell
[root@localhost sh]# vi read.sh 
#!/bin/bash
# Author: Bob (E-mail: Bob@163.com)

# 提示"请输入姓名"并等待 30 秒, 把用户的输入保存入变量 name 中
read -t 30 -p "Please input your name: " name

# 看看变量"$name"中是否保存了你的输入
echo "Name is $name"

# 提示"请输入年龄"并等待30秒, 把用户的输入保存入变量age中 
# 年龄是隐私, 所以我们用"-s"选项隐藏输入
read -s -t 30 -p "Please enter your age: " age

# 调整输出格式,如果不输出换行,一会儿的年龄输出不会换行
echo -e "\n"

# 提示"请选择性别"并等待 30 秒, 把用户的输入保存入变量 gender
# 使用"-n 1"选项只接收一个输入字符就会执行(都不用输入回车)
echo "Age is $age"
read -n 1 -t 30 -p "Please select your gender[M/F]: " gender

echo -e "\n"
echo "Sex is $gender"
```



## 八 : 输入输出重定向

### (一) Bash 的标准输入输出

| 设备   | 设备文件名  | 文件描述符 | 类型         |
| ------ | ----------- | ---------- | ------------ |
| 键盘   | /dev/stdin  | 0          | 标准输入     |
| 显示器 | /dev/stdout | 1          | 标准输出     |
| 显示器 | /dev/stderr | 2          | 标准错误输出 |

### (二) 输出重定向

标准输出重定向

| 符号         | 作用                                                     |
| ------------ | -------------------------------------------------------- |
| 命令 > 文件  | 以覆盖的方式, 把命令的正确输出输出到指定的文件或设备当中 |
| 命令 >> 文件 | 以追加的方式, 把命令的正确输出输出到指定的文件或设备当中 |

标准错误输出重定向

| 符号             | 作用                                                     |
| ---------------- | -------------------------------------------------------- |
| 错误命令 2>文件  | 以覆盖的方式, 把命令的错误输出输出到指定的文件或设备当中 |
| 错误命令 2>>文件 | 以追加的方式, 把命令的错误输出输出到指定的文件或设备当中 |

正确输出和错误输出同时保存

| 符号                       | 作用                                                     |
| -------------------------- | -------------------------------------------------------- |
| 命令 > 文件 2>&1           | 以覆盖的方式, 把正确输出和错误输出都保存到同一个文件当中 |
| **命令 >> 文件 2>&1**      | 以追加的方式, 把正确输出和错误输出都保存到同一个文件当中 |
| 命令 &>文件                | 以覆盖的方式, 把正确输出和错误输出都保存到同一个文件当中 |
| **命令 &>>文件**           | 以追加的方式, 把正确输出和错误输出都保存到同一个文件当中 |
| **命令 >>文件1  2>>文件2** | 把正确的输出追加到文件 1 中, 把错误的输出追加到文件 2 中 |

* **注意: 错误输出`2`和`>`之间不能有空格, 至于`>`之后有没有空格无所谓, 为了方便记忆, 错误输出前后都不要加空格了**

### (三) 输入重定向

同时也支持输入重定向

代码示例 : 在文件 var.ini 中统一定义变量, 通过脚本读出

```shell
a=1
b=2
c=3
```

```shell
#!/bin/bash
# 中心思想就是把a=1,b=2,c=3引入进来声明,这里用到read命令做重定向,可以一次读取一行
# eval命令用于重新运算求出参数的内容,还可读取一连串的参数，然后再依参数本身的特性来执行
while read line;do
	eval $line
done < var.ini
echo $a
echo $b
echo $c
```

代码示例 : 统计 err.log 的行数,单词数和字节数

```shell
[root@localhost ~]# wc < err.log
```

### (四) 垃圾桶

解释一下比较常用的 **`&>/dev/null`** 这个命令  

`&>` 输出, `/dev/null` 这个类似回收站, 任何东西丢到这里面都会消失, 所以通常写脚本的时候, 我们习惯加上 `&>/dev/null`, **用于屏蔽命令的提示信息**

```shell
java -jar xxx.jar &>/dev/null
```




