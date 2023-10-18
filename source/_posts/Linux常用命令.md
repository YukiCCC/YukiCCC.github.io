---
title: Linux常用命令
date: 2023-09-19 23:32:04
tags: Linux
category: Linux
---


## 操作系统

操作系统是实现控制和管理计算机软硬件资源的系统软件。它可以有效地组织多个程序的运行，方便用户操作。常见的操作系统：windows,mac,linux,OS/2,以及各种嵌入式操作系统。目前操作系统的分类：批处理操作系统，分时操作系统，实时操作系统，网络操作系统和分布式操作系统。

操作系统的性能指标：吞吐量，资源利用率，公平性，实时性，可靠性以及安全性。

操作系统功能：

​	存储管理：内存分配，地址映射，内存的保护以及扩充

​	进程管理：进程调度，进程控制，进程之间的通信

​	文件管理：文件存储空间管理、文件操作、目录、读写权限等

​	设备管理：缓冲设备、设备分配、设备驱动

​	用户接口：图形用户接口，命令行接口

## Linux

Linux，全称GNU/Linux，是一种免费使用和自由传播的[类UNIX](https://baike.baidu.com/item/类UNIX/9032872?fromModule=lemma_inlink)操作系统，其内核由[林纳斯·本纳第克特·托瓦兹](https://baike.baidu.com/item/林纳斯·本纳第克特·托瓦兹/1034429?fromModule=lemma_inlink)（Linus Benedict Torvalds）于1991年10月5日首次发布，它主要受到[Minix](https://baike.baidu.com/item/Minix/7106045?fromModule=lemma_inlink)和[Unix](https://baike.baidu.com/item/Unix/219943?fromModule=lemma_inlink)思想的启发，是一个基于[POSIX](https://baike.baidu.com/item/POSIX/3792413?fromModule=lemma_inlink)的多用户、[多任务](https://baike.baidu.com/item/多任务/1011764?fromModule=lemma_inlink)、支持[多线程](https://baike.baidu.com/item/多线程/1190404?fromModule=lemma_inlink)和多[CPU](https://baike.baidu.com/item/CPU/120556?fromModule=lemma_inlink)的[操作系统](https://baike.baidu.com/item/操作系统/192?fromModule=lemma_inlink)。它支持[32位](https://baike.baidu.com/item/32位/5812218?fromModule=lemma_inlink)和[64位](https://baike.baidu.com/item/64位/2262282?fromModule=lemma_inlink)硬件，能运行主要的[Unix](https://baike.baidu.com/item/Unix/219943?fromModule=lemma_inlink)工具软件、应用程序和网络协议。

在国内用的比较多的版本：Ubuntu，Centos，RedHat



# Linux的文件基础

linux系统以文件为基础，系统子目录当中的文件主要是保证系统的正常运行。

/，/home  /usr  /var  /bin /sbin /etc  /dev  /lib

/ : linux系统的根目录（主目录） 

/bin  /sbin: 用于存放linux的系统命令和工具

/boot: linux的启动目录，从此开始启动

/dev：存放linux当中的所有设备文件

/etc: 系统配置文件所在的位置

/lib，/lib64:  存放操作系统的库文件

/home ：用户目录，linux当中每增加一个用户，都会在此目录下相应地增加一个文件，文件名和用户名相同（root除外），给每个用户自己的空间

/mnt:  外部设备的挂载点，可以挂载光驱和软盘

/root： root用户的目录

/usr：通常用来安装各种软件的地方

/var：通常用来放置一些变化的文件

/var/log： 存放系统的日志文件

/run:  Linux系统中用于存放运行文件的临时目录,通常情况下它会包含一些程序运行时需要的文件,例如PID文件、锁文件、socket文件等等。这些文件的作用是协助程序运行时的交互,确保程序能够顺利地完成任务。 

/proc:  Linux 内核提供了一种通过 /proc 文件系统,在运行时访问内核内部数据结构、改变内核设置的机制。proc文件系统是一个伪文件系统,它只存在内存当中,而不占用外存空间。它以文件系统的方式为访问系统内核数据的操作提供接口。

*/srv/:系统服务数据的目录。一些系统服务启动之后,可以在这个目录中保存所需要的数据*。

Linux系统是区分大小写，Linux文件没有扩展名

# 常用命令

## 补全和帮助

tab补全：linux当中的命令比较多，无法记全的情况下，可以使用tab键进行提示。当只有一个匹配命令时，tab键可以帮助不全，当有个匹配，双按tab，会提示命令。

当命令的参数或者option信息不清楚，可以使用man命令或者help进行帮助查看。

​	 man  命令（man ls）

​	命令 --help

## 目录命令

### 目录和路径

主目录和工作目录：每一个用户都有自己的主目录，这个是管理员在创建用户的时候指定的。用户在自己的home目录当中可以进行各种操作，用户对自己的主目录拥有最大的权限。

工作目录：指当前所在的位置，进入linux之后，用户始终都会有一个工作目录。可以使用cd命令切换工作目录。

​		可以使用pwd命令查看当前所在的工作目录的完整路径。

### 目录命令

#### cd

#### ls [options]  

options比较多，无法记住，使用man ls 或者 ls --help方式进行求助

命令比较常，记不住的情况，可以使用tab求助。

```
-a 显示目录中所有文件大小
-k 以KB为单位显示文件大小
-m 以MB为单位显示文件大小
-g 以GB为单位显示文件大小
-h 以易读方式显示文件大小
-s 仅显示总计
-c或--total  除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和
```

几个特殊路径：

​	cd  / :进入根目录

​	cd ~: 进入当前用户的主目录

​	cd .. :返回到上一级目录

​	cd -: 返回上一次所在目录

绝对路径： 从文件的根目录开始的路径，始终以 /开头

相对路径：从当前的工作目录开始的路径，要和工作目录结合起来，才能确定所在的位置。

ls：列举指定目录中的文件和目录

mkdir：创建新目录

rmdir：删除指定的目录（必须是空目录，如果不是空，可以先删除此目录中的对象，再删除，还可以使用rm命令）

## 常用文件命令

touch：创建空文件
cp：复制文件。可将文件复制到不同的目录，也可将指定目录中的文件复制到其他位置
mv：将文件或目录移动至一个新的位置
rm:   删除文件或目录
ln ： 创建链接（类似与windows的快捷方式）
whereis：查找文件。可以查找文件的源、二进制文件或手册
which: 查找二进制文件
find：查找文件
location：查找文件
grep ：所有文本

### touch：创建空文件

用touch命令可以创建一个没有任何内容的空文件。

```shell
touch file01
```

### **cp：复制文件**

类似与DOS命令中的copy
语法：cp 选项  源文件或目录   目标文件或目录 
可以将文件复制到不同的目录，也可以将指定目录中的文件复制到其他位置。
常用参数
-a：相当与-dpr参数
-d：保留链接
-f：强制复制，覆盖目标文件
-i：覆盖时询问用户
-p：保留修改时间和访问权限
-r，-R：递归复制（目录到目录）
-l：创建链接
-v：显示过程

```shell
#当前文件夹下复制一个文件
cp 3.txt 4.txt 
#将当前目录下的txt复制到根目录下
cp *.txt / 
#将data目录（文件）复制到2目录下 
cp -r data data1/  
#将hello文件复制到/opt目录下
cp -p hello.java /opt/hello.java
```

### **mv：移动文件或目录**

将文件或目录移到一个新的位子。也可以用来修改文件名称
mv  选项   源文件或目标  目标文件或目录
常用参数：
-i：交互方式操作，如果mv操作将导致对已存在文件的覆盖，此时系统询问是否重写，要求用户回答y或n
-f：禁止交互操作。覆盖时不会提示。

```shell
mv -i  hello.java  hello1.java
```

### **rm： 删除文件或目录**

​	rm [选项] 文件
​	常用参数：
​	-i : 为了避免误删除文件，可以使用此项，进行用户确认删除
​	-f:  强制删除，使用该选项后将不提示所删除的文件
​	-v:  显示文件的删除速度
​	-r:  删除某个目录以及其中所有的文件和子目录。

```shell
#删除文件
rm aa
#删除目录
rm -rf a
```

<span style="color:blue">在Linux中可以创建链接文件，当使用rm删除链接文件时，只是删除该链接文件，实际的文件仍旧继续存在。</span>

### ln：创建链接文件

​	Linux中的链接类似于windows的快捷方式，分两种：软连接和硬链接。
​	创建软链接，只是在指定的位置上生成一个镜像，不会占用磁盘空间。
​     	语法： ln  -s   目标文件  链接文件名
​	创建硬链接，将在指定的位置上生成一个和源文件大小相同的文件。
​      	语法：    ln   目标文件  链接文件名
​	无论是软链接还是硬链接，链接文件和目录文件都将保持同步变化。
​	<span style="color:blue">不能创建目录的硬链接。</span>

```shell
#创建软链接
ln -s hello  hellolink
#查找文件
find -name hello
```

### whereis：查找文件

whereis用来查找程序的源、二进制文件或手册。
whereis 选项 文件名
常用选项
-b：搜索文件的二进制部分
-m：搜索文件的手册部分
-s：搜索文件的源部分。

不带选项，查找二进制文件和手册的位置

```shell
whereis ls
```

### which：查找二进制文件。

which 选项 文件名
常用选项
-n：指定文件名长度，指定的长度必须大于或等于所有文件中最长的文件名。
-p：与-n参数相同，但此处包含文件路径
-w：指定输出时栏位的宽度
-V：显示版本信息

```shell
which  -V ls
```

### find（会问到）：查找文件

可以按照文件名、文件类型、用户等去查找。
find  路径  选项  [-print] [-exec –ok command] {}\;
常用选项
-name filename :查找名为filename的文件
-perm：按执行权限来查找
-user username：按照文件的所属用户来查找
-group  groupname：按照组来查找
-size n[c] 查长度为n块（n字节）的文件。

```shell
#根据文件名查找
find /home -name 123.sh
#根据文件大小查找 (+大于 -小于)
find /home -size +20000
#根据文件所有者查找：查找所有者为jp的文件
find /home -user jp
```

### locate：查找文件

locate：从已经创建好的一个索引数据库中查找。比find命令的查找速度更快。
locate 选项  文件名
-b, --basename -- 仅匹配路径名的基本名称
-c, --count -- 只输出找到的数量
-d, -- 数据库路径 -- 使用 DB PATH 指定的数据库，而不是默认数据库 /var/lib/mlocate/mlocate.db
-q, -- 安静模式，不会显示任何错误讯息。
一般不使用参数。

locate 与 find 不同: find 是去硬盘找，locate 只在 /var/lib/slocate 资料库中找。
locate 的速度比 find 快，它并不是真的查找，而是查数据库，一般文件数据库在 /var/lib/slocate/slocate.db 中，所以 locate 的查找并不是实时的，而是以数据库的更新为准，一般是系统自己维护，也可以手工升级数据库 ，命令为：updatedb

*常见问题*：
无法执行 stat () `/var/lib/mlocate/mlocate.db': 没有那个文件或目录
解决方案：
执行：updatedb命令

```shell
[root@centos601 桌面]# locate /etc/sh
locate: can not stat () `/var/lib/mlocate/mlocate.db': 没有那个文件或目录
[root@centos601 桌面]# update locate
bash: update: command not found
[root@centos601 桌面]# updatedb
[root@centos601 桌面]# locate /etc/sh
/etc/shadow
/etc/shadow-
/etc/shells
```

### **grep（面试常问）：查找文件**

对查找目标中的具体内容进行查找，是一个强大的文本搜索工具。它是一个管道命令，和其它命令结合使用。
工作方式：在一个或多个文件中搜索字符串模板，可以使用正则表达式进行搜索。

在linux或unix系统中，|就是管道命令，把上一个命令的结果交给管道命令（|）后面的命令处理

```shell
#查找/etc/hosts中包含localhost4的内容
cat /etc/hosts | grep localhost4
```

##   文本内容查看命令

### cat：文件内容查看输出

cat：将文件中的内容输出到设备上，若是多个文件，则按顺序输出。
 cat   选项  文件名
常用选项

- -n：由1开始对所有输出的行数编号
- -b：和-n相似，只是对于空白行不进行编号
- -s：若遇到连续两行以上的空白行，就替换成一行空白行输出。
  若准备将多个文件合并为一个文件，则使用以下方式
  Cat  选项  文件名1  文件名2…… > 新文件

```shell
#将/etc/hosts文件中的内容输出到/opt/a文件中
cat /etc/hosts > /opt/a
#查看/opt/a文件的内容
cat /opt/a
```

### more：分屏显示

more分屏显示 ，可以和其它命令结合使用，也可以单独使用。在 more 这个程序的运行过程中，你有几个按键可以按的：

- 空白键 (space)：代表向下翻一页；
- Enter         ：代表向下翻『一行』；
- /字串         ：代表在这个显示的内容当中，向下搜寻『字串』这个关键字；
- :f            ：立刻显示出档名以及目前显示的行数；
- q             ：代表立刻离开 more ，不再显示该文件内容。
- b 或 [ctrl]-b ：代表往回翻页，不过这动作只对文件有用，对管道无用。

```shell
more /etc/profile
cat day522 | more
```

### less：分屏显示

less分屏显示 ，可以和其它命令结合使用，也可以单独使用。less运行时可以输入的命令有：

- 空白键    ：向下翻动一页；
- 上下箭头：上一行/下一行
- [page down]：向下翻动一页；
- [page up]  ：向上翻动一页；
- /字串     ：向下搜寻『字串』的功能；
- ?字串     ：向上搜寻『字串』的功能；
- n         ：重复前一个搜寻 (与 / 或 ? 有关！)
- N         ：反向的重复前一个搜寻 (与 / 或 ? 有关！)
- q         ：离开 less 这个程序；

```shell
more /etc/profile
```

### head：显示前几行

语法：head [-n number] 文件 

```shell
head -n 3 /etc/profile
```

### **tail（面试常问）**：**最后**几行

语法：tail [-n number] 文件 ，常用来**读取日志文件**

​      -f : 循环读取  tail  -f  文件名

​     -n ：数字  读取最后几行

```shell
tail -n 10 /etc/profile
#循环读取
ping 192.168.66.211 > ping.log &
tail -f ping.log
```

## 4. 用户管理

在Linux中，用户和组的相关信息保存在对应的文件中，一共有三个文件，分别是passwd、shadow和group

- passwd文件
  - 用户信息文件  /etc/目录下。	
  - 系统中的每一个合法用户账号对应于该文件中的一行记录。
  - 每一行由7个部分组成：注册名:口令：用户标识号：组标识号：备注：用户主目录：命令解释程序
    - 注册名：登陆账户，不能重复，区分大小写。
    - 口令：登陆系统的口令。若第一个字符是“*”，表示禁止该账号登陆。
    - 用户标志号：Linux中唯一的用户标识
    - 组标志号：当行用户的默认工作组标记
    - 备注：保存一些用户的信息
    - 用户主目录：个人用户的主目录，该用户登陆后，将该目录作为用户的工作目录
    - 命令解释程序：当前用户登陆系统时运行的程序名称，通常是一个shell程序的全路径名。

- shadow文件
  - 保存用户的口令。/etc/目录下
  - 该文件不能被普通用户读取，只有超级用户root才有权读取。
- group文件
  - 保存在/etc/group当中
  - 每一行数据内容 用户组名称：用户组密码：用户组标识号：用户列表



### 用户组管理

- 添加组：groupadd
  groupadd  policeman 
- 查看组：通过vi或cat命令
  vi  /etc/group
  cat /etc/group
- 删除组：groupdel
  groupdel policeman
- 将现有用户增加到组中
  usermod  -g  组名  用户名
- 从组中删除用户
  gpasswd -d 用户名 组名
- 查看当前用户所在组：groups

### 用户管理

- 创建用户：useradd
  useradd 用户名：创建一个新的用户
  useradd 用户名 -g 组名	：添加用户，并加入到某个组当中
  useradd –m –g 组名 用户名：创建一个新的用户并创建家目录，指定组
- 修改用户：usermod 
  usermod -g 组名 用户名 ：修改用户信息
- 创建密码：password
  passwd 用户名 ：为用户创建密码
- 删除用户：userdel
  userdel 用户名	：删除用户名
  userdel -r 用户名	：删除用户以及用户主目录
  - su   用户名称 切换用户

### 管理员权限

修改 /etc/sudoers文件，给对应的用户增加root权限，用户在执行命令式，使用sudo 命令 即可执行管理员的所有权限。

```shell
## Allow root to run any commands anywhere 
root	ALL=(ALL) 	ALL
#给huangyy增加root权限
huangyy ALL=(ALL) 	ALL
```

## 5. 文件权限

Linux的文件类型大致可以分为5种：
普通文件：用于存储数据、程序等信息的文件。文本文件和二进制文件。
目录文件：由文件系统中的一个目录项组成的文件。用户进行只能对其进行读取，不能进行修改
设备文件：用于与IO设备提供连接的文件，可以分为字符设备文件和块设备文件。每一种I/O设备对应于一个设备文件，存放于/dev/目录中。
链接文件：通过链接文件中指向文件的指针来实现对文件的访问。
管道文件：用于进程间传递数据。Linux对管道的操作与对普通文件的操作相同。



d**rwx** r-x**r-x**  test :  第一组 rwx表示test文件的用户所有者(vagrant,vagrant所在的组vgroup（v1,v2,v3,vagrant）)的权限

​							r-x(黑色) : 文件所有者的同组其他用户的权限（v1,v3,v2）

​                                     **r-x**: vgroup组以外的其他用户的权限							



![权限1](/权限1.png)





操作权限分为三种：
R：读取权限  （４），如果是目录，用户可以浏览目录
W ：写权限　（２），如果是目录，用户可以删除、移动目录内的文件
X: ：执行权限（１），如果是目录，则表示可进入此目录，如果是bash命令，则表示可以执行

<img src="/权限2.png" alt="权限2" style="zoom:50%;" />



### 改变文件权限

使用chmod命令可以修改文件的权限。通过权限字母和操作符表达式的方法来设置权限
 语法：chmod [用户类型] [+|-|=] [权限字符] 文件名
			u=用户权限  g=组权限  o=不同组其他
		+：添加权限；-：取消权限 =：赋予给定权限并取消其他所有权限。
		权限字符：可以使用r、w、x组合，也可以使用s
		使用数字来设置权限
			chmod [数字组合] 文件名
			用3位八进制数来表示文件的3类用户的权限组合
		例如751：表示用户权限为rwx，当前用户组权限：r-x  其他用户组权限为--x

```shell
#赋予abc权限rwxr-xr-x
chmod 755 abc
#同上
chmod u=rwx,g=rx,o=rx abc
#给abc增加组写权限
chmod go+w abc
#给abc去除用户执行权限，增加当前组写权限
chmod u-x,g+w abc
#给所有用户添加读的权限
chmod a+r abc
```

### 修改文件所有者和组

使用chown命令可以修改文件的所有者和组，只有root用户可以更改用户的所有者。只有root用户或文件所有者可以更改文件的组，如果是文件所有者但不是root用户，则只能将组改为当前用户所在组。 
语法：chown 所有者:组 文件

```shell
#修改文件所有者为xiaohuang
#新的用户组为root
chown xiaohuang:root  hello.java
```



## 6. 进程管理

在Linux中，每个执行的程序都称为一个进程。每一个进程都分配一个ID号。每一个进程，都会对应一个父进程。而这个父进程可以复制多个子进程。
每个进程都可能以两种方式存在的，前台与后台。所谓前台进程就是用户目前的屏幕上可以进行操作的。后台进程则是实际在操作，但由于在屏幕上无法看到的进程，通常使用后台方式执行。一般系统的服务都是以后台进程的方式存在，而且都会常驻在系统中，直到关机才结束。

### ps：查看进程

ps命令是用来查看系统中，有哪些正在执行，以及它们执行的状况。可以不加任何参数。
显示详细的进程信息(终端上的所有进程，包括其它用户)：ps -a
以用户的格式显示进程信息：ps -u
显示后台进程运行参数：px –x
查看更全面信息   ps -aux

```SHELL
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root        485  0.0  0.0      0     0 ?        S    Nov02   0:00 [ext4-dio-unwrit]
root        593  0.0  0.0  11248  1372 ?        S<s  Nov02   0:00 /sbin/udevd -d
root       1800  0.1  0.4 260836  8524 ?        Sl   Nov02   0:20 /usr/sbin/vmtoolsd
root       2044  0.0  0.0  27596   832 ?        S<sl Nov02   0:00 auditd
rpc        2179  0.0  0.0  18980   876 ?        Ss   Nov02   0:00 rpcbind
dbus       2199  0.0  0.1  32556  2088 ?        Ssl  Nov02   0:00 dbus-daemon --system
root       2213  0.0  0.2  84960  4864 ?        Ss   Nov02   0:00 NetworkManager --pid-file=/var/run/NetworkManager/NetworkManager.pid
root       2220  0.0  0.1  58120  2608 ?        S    Nov02   0:00 /usr/sbin/modem-manager
rpcuser    2237  0.0  0.0  23352  1380 ?        Ss   Nov02   0:00 rpc.statd
```

USER：用户
PID: 进程的ID
PPID:父进程的ID
%CPU:进程占用的CPU百分比
%MEM：进程占用的内存百分比
VSZ:使用的虚拟内存量（KB)
RSS：该进程占用的固定内存量（KB)
STAT:进程的状态。
TTY:该进程在哪个终端上运行，若与终端无关，则显示？ 若为pst/0等，则表示由网络连接主机进程。
TIME:使用CPU的时间

其中STAT常见的值如下：

| 值   | 含义                                 | 值   | 含义                                  |
| ---- | ------------------------------------ | ---- | ------------------------------------- |
| D    | 无法中断的休眠状态（通常 IO 的进程） | R    | 正在运行可中在队列中可过行的          |
| S    | 处于休眠状态                         | T    | 停止或被追踪                          |
| <    | 优先级高的进程                       | X    | 死掉的进程 （基本很少见）             |
| N    | 优先级较低的进程                     | L    | 有些页被锁进内存                      |
| s    | 进程的领导者（在它之下有子进程）     | l    | 多线程，克隆线程（使用 CLONE_THREAD） |
| \+   | 位于后台的进程组                     |      |                                       |

### kill（面试常问）：终止进程

对于前台进程，在推出程序后该进程将自动结束。在前台进程运行过程中，也可按快捷键Ctrl + C 退出。
对于后台进程，需要使用kill命令来终止。

**信号量：15，9，默认为15 ，告诉进程需要终止，并不一定立刻终止，如果是9，表示强制终止进程**

```shell
#终止pid为4217的进程
kill -9  4217
```



### top（面试常问）

查看系统当前正在执行的进程的相关信息，包括进程id，内存的使用率，CPU的占有率等（动态，实时）

load average: 0.00, 0.01, 0.04，分别表示最近一分钟，五分钟，以及十五分钟的负载状况。

是否load average大于1就是系统负载比较高。不一定。要看CPU的核数和线程数，如果是单核cpu，值等于1就是满负荷，如果是四核，八核，负载大于1说明负载不算太高。

```
-c: 显示完整的进程命令
-s:  保密模式
-p PID 指定进程显示
-n <次数> 执行循环显示次数
-H 显示线程数
```

### &字符

可以将命令的最后加上“&”，使得程序放到后台运行。

基本用于需要一直运行的服务类进程，比如说tomcat，nginx，mysql

## 7. 防火墙

centos7中采用以下命令对防火墙进行处理。

- 查看防火墙状态：firewall-cmd --state
- 停止防火墙： systemctl stop firewalld.service
- 启动防火墙： systemctl start firewalld.service
- 重启防火墙： systemctl restart firewalld.service
- 永久关闭防火墙：systemctl disable firewalld.service
- 永久关闭后重启：systemctl enable firewalld.service
- 查看开机防火墙：systemctl is-enabled firewalld.service

端口操作：

- 开启端口：firewall-cmd --zone=public --remove-port=80/tcp --permanent
- 刷新：firewall-cmd --reload
- 查看端口状态：firewall-cmd --zone=public --query-port=80/tcp 
  - yes表示端口开放，no表示端口不开放
- 关闭端口：firewall-cmd --zone=public --remove-port=80/tcp --permanent

```shell
[root@centos701 etc]# firewall-cmd --zone=public --query-port=80/tcp
no
[root@centos701 etc]# firewall-cmd --zone=public --add-port=80/tcp --permanent
success
[root@centos701 etc]# firewall-cmd --reload
success
[root@centos701 etc]# firewall-cmd --zone=public --query-port=80/tcp
yes  
[root@centos701 etc]# firewall-cmd --zone=public --remove-port=80/tcp --permanent
success
[root@centos701 etc]# firewall-cmd --reload
success
[root@centos701 etc]# firewall-cmd --zone=public --query-port=80/tcp
no


```



## 8. 启动以及关机注销

logout：注销系统
login：回到登录界面
shutdown –h now	：立刻关机
shutdown +5	：5分钟后关机
shutdown 10:30：在10：30关机
shutdown –r now	：立刻关闭系统并重启
reboot	：重新启动系统

# 五. vi的常用指令

vi编辑器是Linux下最有名的编辑器，也是我们学习linux必须掌握的工具，在unix下也可以进行程序的开发。
开发步骤：
vi  文件名
输入i，进入插入模式，
输入Esc键
输入冒号:,再输入wq，保存并退出，如果不保存退出，则输入q!



基本上 vi/vim 共分为三种模式，分别是**命令模式（Command mode）**，**输入模式（Insert mode）**和**底线命令模式（Last line mode）**。 

<img src="/vim工作模式.png" alt="vim工作模式" style="zoom: 50%;" />





## 1. 命令模式

用户刚刚启动 vi/vim，便进入了命令模式。此状态下敲击键盘动作会被Vim识别为命令，而非输入字符。比如我们此时按下i，并不会输入一个字符，i被当作了一个命令。以下是常用的几个命令：

- i, I字符 ：切换到输入模式，以输入字符。i 为『从目前光标所在处输入』， I 为『在目前所在行的第一个非空格符处开始输入』
- a, A字符 ：进入输入模式(Insert mode)，a 为『从目前光标所在的下一个字符处开始输入』， A 为『从光标所在行的最后一个字符处开始输入』
- o, O字符 ：进入输入模式(Insert mode)，这是英文字母 o 的大小写。o 为『在目前光标所在的下一行处输入新的一行』； O 为在目前光标所在处的上一行输入新的一行！
- r, R：进入取代模式(Replace mode)，r 只会取代光标所在的那一个字符一次；R会一直取代光标所在的文字，直到按下 ESC 为止
- x字符 ：删除当前光标所在处的字符。
- :字符 ：切换到底线命令模式，以在最底一行输入命令。

若想要编辑文本：启动Vim，进入了命令模式，按下i，切换到输入模式。

## 2. 输入模式

在命令模式下按下i就进入了输入模式。在输入模式中，可以使用以下按键：

- 字符按键以及Shift组合，输入字符大小写转换
- ENTER：回车键，换行
- BACK SPACE：退格键，删除光标前一个字符
- DEL：删除键，删除光标后一个字符
- 方向键：在文本中移动光标
- HOME/END：移动光标到行首/行尾
- Page Up/Page Down：上/下翻页
- Insert：切换光标为输入/替换模式，光标将变成竖线/下划线
- ESC：退出输入模式，切换到命令模式



## 3. 底线命令模式

在命令模式下按下:（英文冒号）就进入了底线命令模式。底线命令模式可以输入单个或多个字符的命令，可用的命令非常多。按ESC键可随时退出底线命令模式。
在底线命令模式中，基本的命令有（已经省略了冒号）：

- q 退出程序，如果文件内容被修改了，会出现错误，要求使用“!”强制退出。
- q! ：强制退出vim，并且不保存文件
- w： 保存文件
- wq：将修改过的文件存储，并且离开vim
- set nu:在文件中每行加入行号
- set nonu：取消行号
- 输入数字：直接输入数字再按enter，就可以将光标定位到改行行首。
- /字符串：利用/字符串来查找特定的内容，如果查找不是想要的的，可以按“n”键盘继续查找。
- ?字符串：同/字符串



## 删除命令

命令模式下：

​	x：删除当前字符

​	3x: 删除当前光标开始的3个字符，如果想要删除n个字符，将3替换成n

​	X：删除当前光标的前一个字符

​	dd: 删除当前行

​	dj：删除下一行

​	dk：删除上一行

​	3d：删除当前行开始往后的3行

底线模式下

​	:5, 10d: 将5~10的数据删除掉

​	:5,$d: 将5行以后的数据全部删除掉

## 拷贝命令

行拷贝：

​	yy：拷贝当前行

​	nyy：拷贝当前行开始的n行，5yy，从当前行开始拷贝5行的数据

​	p：再当前光标后粘贴

​	shift +p：在当前行前面进行粘贴

​	:1,5 co 20: 将1~5行copy放到20行之后

部分拷贝

​	yw：拷贝一个单词

​	2yl：拷贝当前光标开始的2个字符

​	3yh：拷贝当前光标前面的3个字符（不包括光标的相关字符）



# 六. 文件压缩和解压命令

tar：将指定目录中的所有文件和目录全部进行备份
gzip和gunzip：压缩和解压缩文件
Zip和unzip：压缩文件和解压文件

## tar命令

tar 是用来建立，还原备份文件的工具程序，它可以加入，解开备份文件内的文件。

-A或--catenate 新增文件到已存在的备份文件。
-c或--create 建立新的备份文件。
-C<目的目录>或--directory=<目的目录> 切换到指定的目录。
-f<备份文件>或--file=<备份文件> 指定备份文件。
-v或--verbose 显示指令执行过程。
-x或--extract或--get 从备份文件中还原文件。
-z或--gzip或--ungzip 通过gzip指令处理备份文件。
-Z或--compress或--uncompress 通过compress指令处理备份文件。
--delete 从备份文件中删除指定的文件。
--exclude=<范本样式> 排除符合范本样式的文件。

实际使用时经常联合多个选项一起使用，例如

-zcvf 创建一个压缩文件          
	-zxvf   还原并解压缩文件  

```shell
#归档文件，将tmp文件夹打包成tmp.tgz
tar -zcvf tmp.tgz tmp
#归档文件，排除tmp目录中的w文件，压缩文件名为tmp.tgz,压缩打包放入tmp
tar --exclude=tmp/w  -zcvf tmp.tgz tmp

#创建tmp01目录，并将tmp.tgz解压缩到tmp01目录中。
mkdir tmp01
tar -zxvf tmp.tgz -C tmp01

#将tmp打包成u1.tar
tar -cf u1.tar tmp
#将tmp01打包成u2.tar
tar -cf u2.tar tmp01
#将u1.tar的内容追加到u2.tar当中（u2当中包含tmp和tmp01）
tar -A u1.tar -vf  u2.tar

```

## gzip和gunzip命令

**gzip**

gzip命令用于压缩文件。gzip是个使用广泛的压缩程序，文件经它压缩过后，其名称后面会多出".gz"的扩展名。常用选项如下：

- -d或--decompress或----uncompress 　解开压缩文件。
- -f或--force 　强行压缩文件。不理会文件名称或硬连接是否存在以及该文件是否为符号连接。
- -l或--list 　列出压缩文件的相关信息。
- -v或--verbose 　显示指令执行过程。
- -V或--version 　显示版本信息。

**gunzip命令**

gunzip命令用于解压文件。gunzip是个使用广泛的解压缩程序，它用于解开被gzip压缩过的文件，这些压缩文件预设最后的扩展名为".gz"。事实上gunzip就是gzip的硬连接，因此不论是压缩或解压缩，都可通过gzip指令单独完成。常用选项如下：

- -c或--stdout或--to-stdout 　把解压后的文件输出到标准输出设备。
- -f或-force 　强行解开压缩文件，不理会文件名称或硬连接是否存在以及该文件是否为符号连接。
- -l或--list 　列出压缩文件的相关信息。
- -v或--verbose 　显示指令执行过程。

```shell
#压缩hello.java文件
gzip -vf hello.java
#将hello.java.gz解压缩
gunzip -vf hello.java.gz
```

## zip和unzip

linux 下提供了 zip 和 unzip 程序，zip 是压缩程序，unzip 是解压程序。

zip命令的常用选项

- -g 将文件压缩后附加在既有的压缩文件之后，而非另行建立新的压缩文件。
- -q 不显示指令执行过程。
- -r 递归处理，将指定目录下的所有文件和子目录一并处理。
- -S 包含系统和隐藏文件。
- -v 显示指令执行过程或显示版本信息。

unzip常用的选项

```shell
#将所有的jpg文件压缩成一个z.zip文件
zip z.zip *.jpg
#解压文件
unzip all.zip
```



# JDK安装

将jdk的linux安装文件放入到linux的某个目录下

解压

```shell
tar -zxvf  jdkxxx.tar.gz 
```

设置环境变量

在/etc/profile的最后增加以下内容

```shell
JAVA_HOME=/usr/jdk1.8.0_261
CLASSPATH=.:$JAVA_HOME/lib
PATH=$PATH:$JAVA_HOME/bin

export JAVA_HOME CLASSPATH  PATH
```

重新加载配置文件

```
source /etc/profile
```

进行测试

javac或者java命令就可以测试