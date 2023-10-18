---
title: Redis学习
date: 2023-08-21 21:30:53
tags: Redis
category: Java
---

## 一、Redis简介

### 1. 关于NoSQL

NoSQL的全称是Not only SQL，意即"不仅仅是SQL"，是一项全新的数据库革命性运动

NoSQL，泛指非关系型的数据库，如Redis、MongoDB和memcached等内存数据库。

产生背景：

- 海量数据、数据多样化和实时性
- 高并发、高可扩和高性能

### 2. Redis是什么

Redis是一个开源的高性能键值对（key-value）数据库。它通过提供多种键值数据类型来适应不同场景下的存储需求。

Redis数据就是以key-value形式来存储数据：

- key只能是字符串类型
- value可以是以下五种类型：String、List、Set、Sorted-Sets、Hash

### 3. Redis优势

优点：

- 应用广泛，技术成熟，简单易用
- 基于内存，高性能（Redis读的速度是110000次/s，写的速度是81000次/s ）
- 丰富的数据类型
- 数据持久化
- 支持主从备份和读写分离
- 支持集群

### 4. Redis的应用场景

- 缓存（数据查询、短连接、菜单内容、新闻内容、商品内容等）
- 分布式集群架构中的session分离
- 聊天室的在线好友列表
- 任务队列（秒杀、抢购、12306等）
- 应用排行榜
- 网站访问统计
- 数据过期处理（可以精确到毫秒） 


## 二、安装Redis

### 1. 安装

#### 1.1 Windows

Redis程序文件`redis-win.zip` 				                 	解压到无空格无中文目录下
- redis-server.exe                                                        启用服务的命令
- redis-cli.exe                                                               连接服务端的命令
- 端口号6379

```shell
# 默认连接【本机+6379】
redis-cli
# 指定连接 -h ip   -p 端口
redis-cli -h 127.0.0.1 -p 6379
# 连接好了，退出
exit
# 关闭redis服务器
redis-cli shutdown
```

使用图形化客户端工具 `Redis Desktop Manager.exe`                            

#### 1.2 Linux

​	步骤：

1. 解压redis-3.2.8.tar.gz

   ```bash
   cd ~/software
   tar -zxf redis-3.2.8.tar.gz
   ```

2. 编译

   ```bash
   cd redis-3.2.8
   make
   ```

3. 安装

   ```bash
   mkdir ~/software/redis-bin
   make install PREFIX=~/software/redis-bin/   #PREFIX选项用来指定安装的位置
   ```

4. 启动redis

   ```bash
   cd ~/software/redis-bin/bin/
   ./redis-server #使用默认配置文件启动，默认配置文件所在目录redis-3.2.8/redis.conf
   或
   cp ~/software/redis-3.2.8/redis.conf myredis.conf  #复制默认配置文件到当前目录，并改名
   ./redis-server myredis.conf #使用指定的配置文件启动
   ```

5. 连接redis

   ```bash
   ./redis-cli  #默认连接本机的6379端口(redis默认使用的端口号)
   或
   ./redis-cli -h IP地址 -p 端口号  #连接指定主机、指定端口的redis，如./redis-cli -h localhost -p 6379
   ```

   测试

   ```bash
   127.0.0.1:6379> set name tom
   127.0.0.1:6379> get name
   ```

6. 关闭	

- 方式1：在服务器窗口中按 `Ctrl+C`
- 方式2：在客户端连接后输入 `shutdown` 或 直接输入 `redis-cli shutdown`

​    查看进程

```bash
ps -ef | grep redis	#查看redis的进程信息
或
lsof -i:6379 #查看6379端口的进程信息
```

### 2. 配置

编辑redis.conf配置文件：

```bash
daemonize yes						#配置为守护进程，后台启动

port 6379								#修改监听端口

#让redis支持远程访问，默认只允许本地访问
#bind    127.0.0.1   		#注释掉该行，允许所有主机访问redis
protected-mode no      	#关闭保护模式

requirepass 123				#配置redis密码，使用时需要输入auth 123进行认证，认证后才能操作redis
```

## 三、Redis数据类型

### 1. String类型

#### 1.1 简介

​	字符串类型是Redis中最为基础的数据存储类型，它是以二进制形式存储的，这便意味着该类型可以存储任何格式的数据，如JPEG图像数据或Json对象描述信息等。在Redis中字符串类型的Value最多可以容纳的数据长度是512M。

#### 1.2 操作

- set/get/append/strlen

  ```shell
  $ redis-cli 
  127.0.0.1:6379> select 0					#切换到第1个数据库（默认共有16个数据库，通过数字来进行命名，索引从0开始）
  OK
  127.0.0.1:6379> keys *						#显示所有的键key
  (empty list or set)
  127.0.0.1:6379> set name tom			#设置键
  OK
  127.0.0.1:6379> get name					#获取键对应的值
  "tom"
  127.0.0.1:6379> append mykey "hello"       	#如果该键不存在，则创建，返回当前value的长度
  (integer) 5
  127.0.0.1:6379> append mykey " world"      	#如果该键已经存在，则追加，返回追加后value的长度
  (integer) 11
  127.0.0.1:6379> get mykey                   #获取mykey的值
  "hello world"
  127.0.0.1:6379> strlen mykey               	#获取mykey的长度
  (integer) 11
  #EX和PX表示失效时间，单位为秒和毫秒，两者不能同时使用；NX表示数据库中不存在时才能设置,XX表示存在时才能设置
  127.0.0.1:6379> set mykey "this is test"  EX 5 NX  
  OK
  127.0.0.1:6379> get mykey
  "this is test"
  ```
  ​	注：命令不区分大小写，但key和value区分大小写
  
- incr/decr/incrby/decrby

  ```shell
  127.0.0.1:6379> flushdb   					#清空数据库
  OK
  127.0.0.1:6379> set mykey 20
  OK
  127.0.0.1:6379> incr mykey          		#递增1
  (integer) 21
  127.0.0.1:6379> decr mykey           		#递减1
  (integer) 20
  127.0.0.1:6379> del mykey          			#删除该键
  (integer) 1
  127.0.0.1:6379> decr mykey
  (integer) -1
  127.0.0.1:6379> del mykey
  (integer) 1
  127.0.0.1:6379> incr mykey
  (integer) 1
  127.0.0.1:6379> incrby mykey 5        		#递增5，即步长
  (integer) 15
  127.0.0.1:6379> decrby mykey 10       		#递减10
  (integer) 5
  ```

### 2. List类型

#### 2.1 概述

​	在Redis中，List类型是按照插入顺序排序的字符串链表。和数据结构中的普通链表一样，我们可以在其头部(left)和尾部(right)添加新的元素。在插入时，如果该键并不存在，Redis将为该键创建一个新的链表。与此相反，如果链表中所有的元素均被移除，那么该键也将会被从数据库中删除。List中可以包含的最大元素数量是4294967295。
​	从元素插入和删除的效率视角来看，如果我们是在链表的两头插入或删除元素，这将会是非常高效的操作，即使链表中已经存储了百万条记录，该操作也可以在常量时间内完成。然而需要说明的是，如果元素插入或删除操作是作用于链表中间，那将会是非常低效的。

#### 2.2 操作

- lpush/lpushx/lrange

  ```shell
  127.0.0.1:6379> flushdb
  OK
  #创建键mykey及与其关联的List，然后将参数中的values从左到右依次插入
  127.0.0.1:6379> lpush mykey a b c d
  (integer) 4
  #获取从位置0开始到位置2结束的3个元素
  127.0.0.1:6379> lrange mykey 0 2
  1) "d"
  2) "c"
  3) "b"
  #获取链表中的全部元素，其中0表示第一个元素，-1表示最后一个元素
  127.0.0.1:6379> lrange mykey 0 -1
  1) "d"
  2) "c"
  3) "b"
  4) "a"
  #获取从倒数第3个到倒数第2个的元素
  127.0.0.1:6379> lrange mykey -3 -2
  1) "c"
  2) "b"
  #lpushx表示键存在时才能插入，mykey2键此时并不存在，因此该命令将不会进行任何操作，其返回值为0
  127.0.0.1:6379> lpushx mykey2 e		
  (integer) 0
  #可以看到mykey2没有关联任何List Value
  127.0.0.1:6379> lrange mykey2 0 -1
  (empty list or set)
  #mykey键此时已经存在，所以该命令插入成功，并返回链表中当前元素的数量
  127.0.0.1:6379> lpushx mykey e
  (integer) 5
  #获取该键的List中的第一个元素
  127.0.0.1:6379> lrange mykey 0 0
  1) "e"
  ```


- lpop/llen

  ```shell
  127.0.0.1:6379> flushdb
  OK
  127.0.0.1:6379> lpush mykey a b c d
  (integer) 4
  #取出链表头部的元素，该元素在链表中就已经不存在了
  127.0.0.1:6379> lpop mykey
  "d"
  127.0.0.1:6379> lpop mykey
  "c"
  #在执行lpop命令两次后，链表头部的两个元素已经被弹出，此时链表中元素的数量是2
  127.0.0.1:6379> llen mykey
  (integer) 2
  ```

- lrem/lindex/lset/ltrim

  ```shell
  127.0.0.1:6379> flushdb
  OK
  #准备测试数据
  127.0.0.1:6379> lpush mykey a b c d a c
  (integer) 6
  #从头部(left)向尾部(right)操作链表，删除2个值等于a的元素，返回值为实际删除的数量
  127.0.0.1:6379> lrem mykey 2 a
  (integer) 2
  #查看删除后链表中的全部元素
  127.0.0.1:6379> lrange mykey 0 -1
  1) "c"
  2) "d"
  3) "c"
  4) "b"
  #获取索引为1(头部的第二个元素)的元素值
  127.0.0.1:6379> lindex mykey 1
  "d"
  #将索引为1(头部的第二个元素)的元素值设置为新值e
  127.0.0.1:6379> lset mykey 1 e
  OK
  #查看是否设置成功
  127.0.0.1:6379> lindex mykey 1
  "e"
  #索引值6超过了链表中元素的数量，该命令返回nil
  127.0.0.1:6379> lindex mykey 6
  (nil)
  #设置的索引值6超过了链表中元素的数量，设置失败，该命令返回错误信息。
  127.0.0.1:6379> lset mykey 6 h
  (error) ERR index out of range
  #仅保留索引值0到2之间的3个元素，注意第0个和第2个元素均被保留。
  127.0.0.1:6379> ltrim mykey 0 2
  OK
  #查看trim后的结果
  127.0.0.1:6379> lrange mykey 0 -1
  1) "c"
  2) "e"
  3) "c"
  ```

### 3. Set类型

#### 3.1 概述

​	在Redis中，我们可以将Set类型看作为没有排序的字符集合，也可以在该类型的数据值上执行添加、删除或判断某一元素是否存在等操作。Set可包含的最大元素数量是4294967295。
​	和List类型不同的是，Set集合中不允许出现重复的元素，这一点和Java中的set容器是完全相同的。换句话说，如果多次添加相同元素，Set中将仅保留该元素的一份拷贝。和List类型相比，Set类型在功能上还存在着一个非常重要的特性，即在服务器端完成多个Sets之间的聚合计算操作，如unions并、intersections交和differences差。由于这些操作均在服务端完成，因此效率极高，而且也节省了大量的网络IO开销。

#### 3.2 操作

- sadd/smembers/sismember/scard

  ```shell
  #由于该键myset之前并不存在，因此参数中的三个成员都被正常插入
  127.0.0.1:6379> sadd myset a b c
  (integer) 3
  #查看集合中的元素，从结果可以，输出的顺序和插入顺序无关(无序的)
  127.0.0.1:6379> smembers myset
  1) "a"
  2) "c"
  3) "b"
  #由于参数中的a在myset中已经存在，因此本次操作仅仅插入了d和e两个新成员（不允许重复）
  127.0.0.1:6379> sadd myset a d e
  (integer) 2
  #查看插入的结果
  127.0.0.1:6379> smembers myset
  1) "a"
  2) "c"
  3) "d"
  4) "b"
  5) "e"
  #判断a是否已经存在，返回值为1表示存在
  127.0.0.1:6379> sismember myset a
  (integer) 1
  #判断f是否已经存在，返回值为0表示不存在
  127.0.0.1:6379> sismember myset f
  (integer) 0
  #获取集合中元素的数量
  127.0.0.1:6379> scard myset
  (integer) 5
  ```

- srandmember/spop/srem/smove

  ```shell
  127.0.0.1:6379> del myset
  (integer) 1
  #准备测试数据
  127.0.0.1:6379> sadd myset a b c d
  (integer) 4
  #查看集合中的元素
  127.0.0.1:6379> smembers myset
  1) "c"
  2) "d"
  3) "a"
  4) "b"
  #随机返回一个成员，成员还在集合中
  127.0.0.1:6379> srandmember myset
  "c"
  #取出一个成员，成员会从集合中删除
  127.0.0.1:6379> spop myset
  "b"
  #查看移出后Set的成员信息
  127.0.0.1:6379> smembers myset
  1) "c"
  2) "d"
  3) "a"
  #从Set中移出a、d和f三个成员，其中f并不存在，因此只有a和d两个成员被移出，返回为2
  127.0.0.1:6379> srem myset a d f
  (integer) 2
  #查看移出后的输出结果
  127.0.0.1:6379> smembers myset
  1) "c"
  
  127.0.0.1:6379> del myset
  (integer) 1
  #为后面的smove命令准备数据
  127.0.0.1:6379> sadd myset a b
  (integer) 2
  127.0.0.1:6379> sadd myset2 c d
  (integer) 2
  #将a从myset移到myset2，从结果可以看出移动成功
  127.0.0.1:6379> smove myset myset2 a
  (integer) 1
  ```

#### 3.3 应用范围

1. 可以使用Redis的Set数据类型跟踪一些唯一性数据，比如访问某一博客的唯一IP地址信息。对于此场景，我们仅需在每次访问该博客时将访问者的IP存入Redis中，Set数据类型会自动保证IP地址的唯一性。
2. 充分利用Set类型的服务端聚合操作方便、高效的特性，可以用于维护数据对象之间的关联关系。比如所有购买某一电子设备的客户ID被存储在一个指定的Set中，而购买另外一种电子产品的客户ID被存储在另外一个Set中，如果此时我们想获取有哪些客户同时购买了这两种商品时，Set的intersections命令就可以充分发挥它的方便和效率的优势了。

### 4. Sorted-Sets类型

#### 4.1 概述

​	Sorted-Sets和Sets类型极为相似，也称为Zset，它们都是字符串的集合，都不允许重复的成员出现在一个Set中。它们之间的主要差别是Sorted-Sets中的每一个成员都会有一个分数(score)与之关联，Redis正是通过分数来为集合中的成员进行从小到大的排序（默认）。然而需要额外指出的是，尽管Sorted-Sets中的成员必须是唯一的，但是分数(score)却是可以重复的。
​	在Sorted-Set中添加、删除或更新一个成员都是非常快速的操作。由于Sorted-Sets中的成员在集合中的位置是有序的，因此，即便是访问位于集合中部的成员也仍然是非常高效的。事实上，Redis所具有的这一特征在很多其它类型的数据库中是很难实现的，换句话说，在该点上要想达到和Redis同样的高效，在其它数据库中进行建模是非常困难的。

#### 4.2 操作

- zadd/zrange/zcard/zrank/zcount/zrem/zscore/zincrby

  ```shell
  #添加一个分数为1的成员
  127.0.0.1:6379> zadd myzset 1 "one"
  (integer) 1
  #添加两个分数分别是2和3的两个成员
  127.0.0.1:6379> zadd myzset 2 "two" 3 "three"
  (integer) 2
  #通过索引获取元素，0表示第一个成员，-1表示最后一个成员。WITHSCORES选项表示返回的结果中包含每个成员及其分数，否则只返回成员
  127.0.0.1:6379> zrange myzset 0 -1 WITHSCORES
  1) "one"
  2) "1"
  3) "two"
  4) "2"
  5) "three"
  6) "3"
  #获取myzset键中成员的数量  
  127.0.0.1:6379> zcard myzset
  (integer) 3
  #获取成员one在集合中的索引，0表示第一个位置
  127.0.0.1:6379> zrank myzset one
  (integer) 0
  #成员four并不存在，因此返回nil
  127.0.0.1:6379> zrank myzset four
  (nil)
  #获取符合指定条件的成员数量，分数满足表达式1 <= score <= 2的成员的数量
  127.0.0.1:6379> zcount myzset 1 2
  (integer) 2
  #删除成员one和two，返回实际删除成员的数量
  127.0.0.1:6379> zrem myzset one two
  (integer) 2
  #查看是否删除成功
  127.0.0.1:6379> zcard myzset
  (integer) 1
  #获取成员three的分数。返回值是字符串形式
  127.0.0.1:6379> zscore myzset three
  "3"
  #由于成员two已经被删除，所以该命令返回nil
  127.0.0.1:6379> zscore myzset two
  (nil)
  #将成员three的分数增加2，并返回该成员更新后的分数
  127.0.0.1:6379> zincrby myzset 2 three
  "5"
  #将成员three的分数增加-1，并返回该成员更新后的分数
  127.0.0.1:6379> zincrby myzset -1 three
  "4"
  #查看在更新了成员的分数后是否正确
  127.0.0.1:6379> zrange myzset 0 -1 withscores
  1) "three"
  2) "4"
  ```

- zrangebyscore/zremrangebyscore/zremrangebyrank

  ```shell
  127.0.0.1:6379> del myzset
  (integer) 1
  127.0.0.1:6379> zadd myzset 1 one 2 two 3 three 4 four
  (integer) 4
  #通过分数获取元素，获取分数满足表达式1 <= score <= 2的成员
  127.0.0.1:6379> zrangebyscore myzset 1 2
  1) "one"
  2) "two"
  #-inf表示第一个成员，+inf表示最后一个成员，limit后面的参数用于限制返回成员的数量，
  #2表示从位置索引(0-based)等于2的成员开始，取后面3个成员，类似于MySQL中的limit
  127.0.0.1:6379> zrangebyscore myzset -inf +inf withscores limit 2 3
  1) "three"
  2) "3"
  3) "four"
  4) "4"
  #根据分数删除成员，删除分数满足表达式1 <= score <= 2的成员，并返回实际删除的数量
  127.0.0.1:6379> zremrangebyscore myzset 1 2
  (integer) 2
  #看一下上面的删除是否成功
  127.0.0.1:6379> zrange myzset 0 -1
  1) "three"
  2) "four"
  #根据索引删除成员，删除索引满足表达式0 <= rank <= 1的成员
  127.0.0.1:6379> zremrangebyrank myzset 0 1
  (integer) 2
  #查看上一条命令是否删除成功
  127.0.0.1:6379> zcard myzset
  (integer) 0
  ```

#### 4.3 应用范围

1. 可以用于大型在线游戏的积分排行榜。每当玩家的分数发生变化时，可以执行ZADD命令更新玩家的分数，此后再通过ZRANGE命令获取积分TOP 10的用户信息。当然我们也可以利用ZRANK命令通过username来获取玩家的排行信息。最后我们将组合使用ZRANGE和ZRANK命令快速的获取和某个玩家积分相近的其他用户的信息。
2. Sorted-Sets类型还可用于构建索引数据。

### 5. Hash类型

#### 5.1 概述

 	可以将Redis中的Hash类型看成具有String Key和String Value的map容器。所以该类型非常适合于存储值对象的信息。如Username、Password和Age等。如果Hash中包含很少的字段，那么该类型的数据也将仅占用很少的磁盘空间。每一个Hash可以存储4294967295个键值对。

#### 5.2 操作

- hset/hget/hlen/hexists/hdel/hsetnx

  ```shell
  #给键值为myhash的键设置字段为field1，值为itany
  127.0.0.1:6379> hset myhash field1 "itany"
  (integer) 1
  #获取键值为myhash，字段为field1的值
  127.0.0.1:6379> hget myhash field1
  "itany"
  #myhash键中不存在field2字段，因此返回nil
  127.0.0.1:6379> hget myhash field2
  (nil)
  #给myhash关联的Hashes值添加一个新的字段field2，其值为liu
  127.0.0.1:6379> hset myhash field2 "liu"
  (integer) 1
  #获取myhash键的字段数量
  127.0.0.1:6379> hlen myhash
  (integer) 2
  #判断myhash键中是否存在字段名为field1的字段，由于存在，返回值为1
  127.0.0.1:6379> hexists myhash field1
  (integer) 1
  #删除myhash键中字段名为field1的字段，删除成功返回1
  127.0.0.1:6379> hdel myhash field1
  (integer) 1
  #再次删除myhash键中字段名为field1的字段，由于上一条命令已经将其删除，因为没有删除，返回0
  127.0.0.1:6379> hdel myhash field1
  (integer) 0
  #通过hsetnx命令给myhash添加新字段field1，其值为itany，因为该字段已经被删除，所以该命令添加成功并返回1
  127.0.0.1:6379> hsetnx myhash field1 "itany"
  (integer) 1
  #由于myhash的field1字段已经通过上一条命令添加成功，因为本条命令不做任何操作后返回0
  127.0.0.1:6379> hsetnx myhash field1 "itany"
  (integer) 0
   
  ```

- hincrby

  ```shell
  127.0.0.1:6379> del myhash
  (integer) 1
  #准备测试数据
  127.0.0.1:6379> hset myhash field 5
  (integer) 1
  #给myhash的field字段的值加1，返回加后的结果
  127.0.0.1:6379> hincrby myhash field 1
  (integer) 6
  #给myhash的field字段的值加-1，返回加后的结果
  127.0.0.1:6379> hincrby myhash field -1
  (integer) 5
  #给myhash的field字段的值加-10，返回加后的结果
  127.0.0.1:6379> hincrby myhash field -10
  (integer) -5  
  ```

- hmset/hmget/hgetall/hkeys/hvals

  ```shell
  127.0.0.1:6379> del myhash
  (integer) 1
  #为该键myhash，一次性设置多个字段，分别是field1 = "hello", field2 = "world"
  127.0.0.1:6379> hmset myhash field1 "hello" field2 "world"
  OK
  #获取myhash键的多个字段，其中field3并不存在，因为在返回结果中与该字段对应的值为nil
  127.0.0.1:6379> hmget myhash field1 field2 field3
  1) "hello"
  2) "world"
  3) (nil)
  #返回myhash键的所有字段及其值，从结果中可以看出，他们是逐对列出的
  127.0.0.1:6379> hgetall myhash
  1) "field1"
  2) "hello"
  3) "field2"
  4) "world"
  #仅获取myhash键中所有字段的名字
  127.0.0.1:6379> hkeys myhash
  1) "field1"
  2) "field2"
  #仅获取myhash键中所有字段的值
  127.0.0.1:6379> hvals myhash
  1) "hello"
  2) "world" 
  ```

## 四、Key操作命令

### 1. 命令列表

| 命令用法           | 解释                                                         |
| ------------------ | ------------------------------------------------------------ |
| keys    pattern    | 获取所有匹配pattern参数的Keys。需要说明的是，在我们的正常操作中应该尽量避免对该命令的调用，因为对于大型数据库而言，该命令是非常耗时的，对Redis服务器的性能打击也是比较大的。pattern支持glob-style的通配符格式，如*表示任意一个或多个字符，?表示任意字符，[abc]表示方括号中任意一个字母。 |
| del key [key...]   | 从数据库删除中参数中指定的keys，如果指定键不存在，则直接忽略。 |
| exists key         | 判断指定键是否存在。                                         |
| persist key        | 如果Key存在过期时间，该命令会将其过期时间消除，使该Key不再有超时，而是可以持久化存储。 |
| expire key seconds | 该命令为参数中指定的Key设定超时的秒数，在超过该时间后，Key被自动的删除。如果该Key在超时之前被修改，与该键关联的超时将被移除。 |
| ttl key            | 获取该键所剩的超时描述。                                     |
| type key           | 获取与参数中指定键关联值的类型，该命令将以字符串的格式返回。 |

### 2. 操作

- keys/del/exists

  ```shell
  127.0.0.1:6379> flushdb
  OK
  #添加String类型的数据
  127.0.0.1:6379> set mykey 2
  OK
  #添加List类型的数据
  127.0.0.1:6379> lpush mylist a b c
  (integer) 3
  #添加Set类型的数据
  127.0.0.1:6379> sadd myset 1 2 3
  (integer) 3
  #添加Sorted-Set类型的数据
  127.0.0.1:6379> zadd myzset 1 "one" 2 "two"
  (integer) 2
  #添加Hash类型的数据
  127.0.0.1:6379> hset myhash username "tom"
  (integer) 1
  #根据参数中的模式，获取当前数据库中符合该模式的所有key，从输出可以看出，该命令在执行时并不区分与Key关联的Value类型
  127.0.0.1:6379> keys my*
  1) "myset"
  2) "mykey"
  3) "myzset"
  4) "myhash"
  5) "mylist"
  #删除了两个Keys
  127.0.0.1:6379> del mykey mylist
  (integer) 2
  #查看刚刚删除的Key是否还存在，从返回结果看，mykey确实已经删除了
  127.0.0.1:6379> exists mykey
  (integer) 0
  #查看一下没有删除的Key，以和上面的命令结果进行比较
  127.0.0.1:6379> exists myset
  (integer) 1
  ```
  
- ttl/persist/expire

  ```shell
  127.0.0.1:6379> flushdb
  OK
  #准备测试数据，将该键的超时设置为100秒
  127.0.0.1:6379> set mykey "hello" ex 100
  OK
  #通过ttl命令查看还剩多少秒
  127.0.0.1:6379> ttl mykey
  (integer) 97
  #立刻执行persist命令，该存在超时的键变成持久化的键，即将该Key的超时去掉
  127.0.0.1:6379> persist mykey
  (integer) 1
  #ttl的返回值告诉我们，该键已经没有超时了
  127.0.0.1:6379> ttl mykey
  (integer) -1
  #为后面的expire命令准备数据
  127.0.0.1:6379> del mykey
  (integer) 1
  127.0.0.1:6379> set mykey "hello"
  OK
  #设置该键的超时被100秒
  127.0.0.1:6379> expire mykey 100
  (integer) 1
  #用ttl命令看当前还剩下多少秒，从结果中可以看出还剩下96秒
  127.0.0.1:6379> ttl mykey
  (integer) 96
  #重新更新该键的超时时间为20秒，从返回值可以看出该命令执行成功
  127.0.0.1:6379> expire mykey 20
  (integer) 1
  #再用ttl确认一下，从结果中可以看出被更新了
  127.0.0.1:6379> ttl mykey
  (integer) 17
  #立刻更新该键的值，以使其超时无效。
  127.0.0.1:6379> set mykey "world"
  OK
  #从ttl的结果可以看出，在上一条修改该键的命令执行后，该键的超时也无效了
  127.0.0.1:6379> ttl mykey
  (integer) -1
  ```

- type

  ```shell
  127.0.0.1:6379> del mykey
  (integer) 1
  #添加不同类型的测试数据
  127.0.0.1:6379> set mykey 2
  OK
  127.0.0.1:6379> lpush mylist a b c
  (integer) 3
  127.0.0.1:6379> sadd myset 1 2 3
  (integer) 3
  127.0.0.1:6379> zadd myzset 1 "one" 2 "two"
  (integer) 2
  127.0.0.1:6379> hset myhash username "tom"
  (integer) 1
  #分别查看数据的类型
  127.0.0.1:6379> type mykey
  string
  127.0.0.1:6379> type mylist
  list
  127.0.0.1:6379> type myset
  set
  127.0.0.1:6379> type myzset
  zset
  127.0.0.1:6379> type myhash
  hash
  ```

## 五、持久化

### 1. 概述

Redis提供了两种数据持久化的方式：

- RDB

  该机制是指在指定的时间间隔内将内存中的数据集快照写入磁盘。    

- AOF

  该机制将以日志的形式记录服务器所处理的每一个写操作

  在Redis服务器启动之初会读取该文件来重新构建数据库，以保证启动后数据库中的数据是完整的。

### 2. RDB

Redis Database：通过单文件的方式来持久化

RDB是默认的持久化方式，默认存储在启动redis服务器时所在当前目录下的dump.rdb文件中，一般都会修改存储在一个固定的目录中

编辑配置文件：

```shell
$ vi myredis.conf
	dbfilename dump.rdb  				#持久化文件的名称
	#dir  ./     						#持久化文件的目录,默认为执行redis-server命令时所在的当前目录
	dir /home/soft01/software/dump/		#修改存储位置为一个固定的目录
```

持久化的时机：

- 在数据库关闭时会持久化（需要注意的是在数据库宕机时不会生成，数据可能会丢失）
- 满足特定条件时会持久化，编辑配置文件：

```shell
$ vi myredis.conf
	save 900 1		#在900秒内，只要有1个key发生变化，就会dump持久化
	save 300 10
	save 60 10000
```

优缺点：

- 缺点：可能会丢失数据
- 优点：效率比较高

### 3. AOF 

Append Only File：通过操作日志的方式来持久化

编辑配置文件：

```shell
 $ vi myredis.conf
 	appendonly yes 						#开启aof模式的持久化
 	appendfilename "appendonly.aof" 	#aof的持久化文件
 	appendfsync everysec   				#每一秒进行一次持久化操作，可取值：always、everysec、no
 	dir /home/soft01/software/dump/	  	#持久化文件的目录，与RDB相同
```

优缺点：

- 缺点：效率比较差
- 优点：丢失数据量比较少

## 六、SpringBoot整合Redis

### 1. 用法

步骤：

1. 创建一个SpringBoot工程，选择以下模块：Spring Data Redis、Web、Lombok、DevTools

2. 配置redis

   ```properties
   spring:
     redis:
       host: localhost
       port: 6379
       password:
       database: 0
       # lettuce连接池配置
       lettuce:
         pool:
           max-wait: 10000
           max-idle: 20
           min-idle: 10
           max-active: 8
   ```

   注：Java中操作Redis主要有两种客户端：jedis和lettuce，SpringBoot中默认使用的是lettuce

3. 基本操作

    使用SpringDataRedis提供的工具类：RedisTemplate

### 2. RedisConfig配置类

```java
/**
 * Redis配置类
 */
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        // 创建RedisTemplate对象
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        // 设置连接工厂
        redisTemplate.setConnectionFactory(redisConnectionFactory);

        // 设置key的序列化方式
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        redisTemplate.setKeySerializer(stringRedisSerializer);
        redisTemplate.setHashKeySerializer(stringRedisSerializer);

        // 设置value的序列化方式
        GenericJackson2JsonRedisSerializer jsonRedisSerializer =new GenericJackson2JsonRedisSerializer();
        redisTemplate.setValueSerializer(jsonRedisSerializer);
        redisTemplate.setHashValueSerializer(jsonRedisSerializer);

        return redisTemplate;
    }
}
```

## 七、Redis应用：验证码

将验证码存储在Redis中

添加依赖，使用captcha生成验证码

```xml
<!--easy-captcha-->
<dependency>
    <groupId>com.github.whvcse</groupId>
    <artifactId>easy-captcha</artifactId>
    <version>1.6.2</version>
</dependency>
```
