---
title: MySQL面试题
date: 2023-10-05 18:22:44
tags: 面试技巧
category: MySQL
---

## MySQL执行引擎



[![图片](http://118.25.100.99/img/img_interview_file1/10.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[图片](javascript:void(0))



## MySQL分页优化



### mysql使用limit分页优化方案

#### 测试实验

mysql分页直接用limit start, count分页语句：

```csharp
select * from product limit start, count
```

当起始页较小时，查询没有性能问题，我们分别看下从10， 100， 1000， 10000开始分页的执行时间（每页取20条），如下：

```csharp
select * from product limit 10, 20 0.016秒
select * from product limit 100, 20 0.016秒
select * from product limit 1000, 20 0.047秒
select * from product limit 10000, 20 0.094秒
```

我们已经看出随着起始记录的增加，时间也随着增大， 这说明分页语句limit跟起始页码是有很大关系的，
那么我们把起始记录改为40w看下（也就是记录的一半左右）

```csharp
select * from product limit 400000, 20 3.229秒
```

再看我们取最后一页记录的时间

```csharp
select * from product limit 866613, 20 37.44秒
```

像这种分页最大的页码页显然这种时间是无法忍受的。

从中我们也能总结出两件事情：

- limit语句的查询时间与起始记录的位置成正比。
- mysql的limit语句是很方便，但是对记录很多的表并不适合直接使用。

#### 对limit分页问题的性能优化方法

利用表的覆盖索引来加速分页查询

我们都知道，利用了索引查询的语句中如果只包含了那个索引列（覆盖索引），那么这种情况会查询很快。

因为利用索引查找有优化算法，且数据就在查询索引上面，不用再去找相关的数据地址了，这样节省了很多时间。

另外Mysql中也有相关的索引缓存，在并发高的时候利用缓存就效果更好了。

在我们的例子中，我们知道id字段是主键，自然就包含了默认的主键索引。现在让我们看看利用覆盖索引的查询效果如何：
这次我们之间查询最后一页的数据（利用覆盖索引，只包含id列），如下：

```csharp
select id from product limit 866613, 20
```

查询时间为0.2秒，相对于查询了所有列的37.44秒，提升了大概100多倍的速度。

那么如果我们也要查询所有列，有两种方法，

- id>=的形式：

```bash
SELECT * FROM product 
WHERE ID > =(select id from product limit 866613, 1) limit 20
```

查询时间为0.2秒，简直是一个质的飞跃啊。

- 利用join

```csharp
SELECT * FROM product a 
JOIN (select id from product limit 866613, 20) b ON a.ID = b.id
```

查询时间也很短，赞！
其实两者用的都是一个原理嘛，所以效果也差不多。

## MVCC多版本并发控制



### 前提概要

------

#### 什么是MVCC?

> **MVCC**
> **`MVCC`**，全称`Multi-Version Concurrency Control`，即多版本并发控制。MVCC是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问，在编程语言中实现事务内存。
> [mvcc - @百度百科](https://baike.baidu.com/item/MVCC/6298019?fr=aladdin)

**MVCC**在**MySQL InnoDB**中的实现主要是为了提高数据库并发性能，用更好的方式去处理读-写冲突，做到即使有读写冲突时，也能做到不加锁，非阻塞并发读

------

#### 什么是当前读和快照读？

在学习MVCC多版本并发控制之前，我们必须先了解一下，什么是MySQL InnoDB下的`当前读`和`快照读`?

- **当前读**
  像select lock in share mode(`共享锁`), select for update ; update, insert ,delete(`排他锁`)这些操作都是一种当前读，为什么叫当前读？就是它读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁
- **快照读**
  像`不加锁`的select操作就是快照读，即不加锁的非阻塞读；快照读的前提是隔离级别不是串行级别，串行级别下的快照读会退化成当前读；之所以出现快照读的情况，是基于提高并发性能的考虑，快照读的实现是基于多版本并发控制，即MVCC,可以认为MVCC是行锁的一个变种，但它在很多情况下，避免了加锁操作，降低了开销；既然是基于多版本，即快照读可能读到的并不一定是数据的最新版本，而有可能是之前的历史版本

**说白了MVCC就是为了实现读-写冲突不加锁，而这个读指的就是`快照读`, 而非当前读，当前读实际上是一种加锁的操作，是悲观锁的实现**

------

#### 当前读，快照读和MVCC的关系

- 准确的说，MVCC多版本并发控制指的是 **“维持一个数据的多个版本，使得读写操作没有冲突”** 这么一个概念。仅仅是一个理想概念
- 而在MySQL中，实现这么一个MVCC理想概念，**我们就需要MySQL提供具体的功能去实现它，而快照读就是MySQL为我们实现MVCC理想模型的其中一个具体非阻塞读功能**。而相对而言，当前读就是悲观锁的具体功能实现
- 要说的再细致一些，快照读本身也是一个抽象概念，再深入研究。MVCC模型在MySQL中的具体实现则是由 **`3个隐式字段`**，**`undo日志`** ，**`Read View`** 等去完成的，具体可以看下面的MVCC实现原理

------

#### MVCC能解决什么问题，好处是？

**数据库并发场景有三种，分别为：**

- `读-读`：不存在任何问题，也不需要并发控制
- `读-写`：有线程安全问题，可能会造成事务隔离性问题，可能遇到脏读，幻读，不可重复读
- `写-写`：有线程安全问题，可能会存在更新丢失问题，比如第一类更新丢失，第二类更新丢失

**MVCC带来的好处是？**
多版本并发控制（MVCC）是一种用来解决`读-写冲突`的**无锁并发控制**，也就是为事务分配单向增长的时间戳，为每个修改保存一个版本，版本与事务时间戳关联，读操作只读该事务开始前的数据库的快照。 所以MVCC可以为数据库解决以下问题

- 在并发读写数据库时，可以做到在读操作时不用阻塞写操作，写操作也不用阻塞读操作，提高了数据库并发读写的性能
- 同时还可以解决脏读，幻读，不可重复读等事务隔离问题，但不能解决更新丢失问题

**小结一下咯**
总之，MVCC就是因为大牛们，不满意只让数据库采用悲观锁这样性能不佳的形式去解决读-写冲突问题，而提出的解决方案，所以**在数据库中，因为有了MVCC，所以我们可以形成两个组合：**

- `MVCC + 悲观锁`
  MVCC解决读写冲突，悲观锁解决写写冲突
- `MVCC + 乐观锁`
  MVCC解决读写冲突，乐观锁解决写写冲突

这种组合的方式就可以最大程度的提高数据库并发性能，并解决读写冲突，和写写冲突导致的问题

### MVCC的实现原理

------

MVCC的目的就是多版本并发控制，在数据库中的实现，就是为了解决`读写冲突`，它的实现原理主要是依赖记录中的 **`3个隐式字段`**，**`undo日志`** ，**`Read View`** 来实现的。所以我们先来看看这个三个point的概念

#### **隐式字段**

每行记录除了我们自定义的字段外，还有数据库隐式定义的`DB_TRX_ID`,`DB_ROLL_PTR`,`DB_ROW_ID`等字段

- `DB_TRX_ID`
  6byte，最近修改(`修改/插入`)事务ID：记录创建这条记录/最后一次修改该记录的事务ID
- `DB_ROLL_PTR`
  7byte，回滚指针，指向这条记录的上一个版本（存储于rollback segment里）
- `DB_ROW_ID`
  6byte，隐含的自增ID（隐藏主键），如果数据表没有主键，InnoDB会自动以`DB_ROW_ID`产生一个聚簇索引
- 实际还有一个删除flag隐藏字段, 既记录被更新或删除并不代表真的删除，而是删除flag变了

[![在这里插入图片描述](http://118.25.100.99/img/img_interview_file1/63.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[在这里插入图片描述](javascript:void(0))


如上图，`DB_ROW_ID`是数据库默认为该行记录生成的唯一隐式主键，`DB_TRX_ID`是当前操作该记录的事务ID,而`DB_ROLL_PTR`是一个回滚指针，用于配合undo日志，指向上一个旧版本



------

#### **undo日志**

undo log主要分为两种：

- **insert undo log**
  代表事务在`insert`新记录时产生的`undo log`, 只在事务回滚时需要，并且在事务提交后可以被立即丢弃
- **update undo log**
  事务在进行`update`或`delete`时产生的`undo log`; 不仅在事务回滚时需要，在快照读时也需要；所以不能随便删除，只有在快速读或事务回滚不涉及该日志时，对应的日志才会被`purge`线程统一清除

> **purge**
>
> - 从前面的分析可以看出，为了实现InnoDB的MVCC机制，更新或者删除操作都只是设置一下老记录的deleted_bit，并不真正将过时的记录删除。
> - 为了节省磁盘空间，InnoDB有专门的purge线程来清理deleted_bit为true的记录。为了不影响MVCC的正常工作，purge线程自己也维护了一个read view（这个read view相当于系统中最老活跃事务的read view）;如果某个记录的deleted_bit为true，并且DB_TRX_ID相对于purge线程的read view可见，那么这条记录一定是可以被安全清除的。

对MVCC有帮助的实质是`update undo log` ，`undo log`实际上就是存在`rollback segment`中旧记录链，**它的执行流程如下：**

***\*一、\** 比如一个有个事务插入persion表插入了一条新记录，记录如下，`name`为Jerry, `age`为24岁，`隐式主键`是1，`事务ID`和`回滚指针`，我们假设为NULL**

[![img](http://118.25.100.99/img/img_interview_file1/64.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[img](javascript:void(0))



***\*二、\** 现在来了一个`事务1`对该记录的`name`做出了修改，改为Tom**

- 在`事务1`修改该行(记录)数据时，数据库会先对该行加`排他锁`
- 然后把该行数据拷贝到`undo log`中，作为旧记录，既在`undo log`中有当前行的拷贝副本
- 拷贝完毕后，修改该行`name`为Tom，并且修改隐藏字段的事务ID为当前`事务1`的ID, 我们默认从`1`开始，之后递增，回滚指针指向拷贝到`undo log`的副本记录，既表示我的上一个版本就是它
- 事务提交后，释放锁

[![img](http://118.25.100.99/img/img_interview_file1/65.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[img](javascript:void(0))



***\*三、\** 又来了个`事务2`修改`person表`的同一个记录，将`age`修改为30岁**

- 在`事务2`修改该行数据时，数据库也先为该行加锁
- 然后把该行数据拷贝到`undo log`中，作为旧记录，发现该行记录已经有`undo log`了，那么最新的旧数据作为链表的表头，插在该行记录的`undo log`最前面
- 修改该行`age`为30岁，并且修改隐藏字段的事务ID为当前`事务2`的ID, 那就是`2`，回滚指针指向刚刚拷贝到`undo log`的副本记录
- 事务提交，释放锁

[![img](http://118.25.100.99/img/img_interview_file1/66.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[img](javascript:void(0))



从上面，我们就可以看出，不同事务或者相同事务的对同一记录的修改，会导致该记录的`undo log`成为一条记录版本线性表，既链表，`undo log`的链首就是最新的旧记录，链尾就是最早的旧记录（**当然就像之前说的该undo log的节点可能是会purge线程清除掉，向图中的第一条insert undo log，其实在事务提交之后可能就被删除丢失了，不过这里为了演示，所以还放在这里**）

------

#### **Read View(读视图)**

**什么是Read View?**

什么是Read View，说白了Read View就是事务进行`快照读`操作的时候生产的`读视图`(Read View)，在该事务执行的快照读的那一刻，会生成数据库系统当前的一个快照，记录并维护系统当前活跃事务的ID(**当每个事务开启时，都会被分配一个ID, 这个ID是递增的，所以最新的事务，ID值越大**)

所以我们知道 `Read View`主要是用来做可见性判断的, 即当我们某个事务执行快照读的时候，对该记录创建一个`Read View`读视图，把它比作条件用来判断当前事务能够看到哪个版本的数据，既可能是当前最新的数据，也有可能是该行记录的`undo log`里面的某个版本的数据。

`Read View`遵循一个可见性算法，主要是将`要被修改的数据`的最新记录中的`DB_TRX_ID`（即当前事务ID）取出来，与系统当前其他活跃事务的ID去对比（由Read View维护），如果`DB_TRX_ID`跟Read View的属性做了某些比较，不符合可见性，那就通过`DB_ROLL_PTR`回滚指针去取出`Undo Log`中的`DB_TRX_ID`再比较，即遍历链表的`DB_TRX_ID`（从链首到链尾，即从最近的一次修改查起），直到找到满足特定条件的`DB_TRX_ID`, **那么这个DB_TRX_ID所在的旧记录就是当前事务能看见的最新`老版本`**

**那么这个判断条件是什么呢？**
[![在这里插入图片描述](http://118.25.100.99/img/img_interview_file1/67.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[在这里插入图片描述](javascript:void(0))


我们这里盗窃[@呵呵一笑百媚生](https://www.zhihu.com/question/66320138/answer/241418502)一张源码图，如上，它是一段MySQL判断可见性的一段源码，即`changes_visible`方法（**不完全哈，但能看出大致逻辑**），该方法展示了我们拿DB_TRX_ID去跟Read View某些属性进行怎么样的比较



在展示之前，我先简化一下Read View，我们可以把Read View简单的理解成有三个全局属性

> - `trx_list`（名字我随便取的）
>   **一个数值列表，用来维护Read View生成时刻系统正活跃的事务ID**
> - `up_limit_id`
>   **记录trx_list列表中事务ID最小的ID**
> - `low_limit_id`
>   **ReadView生成时刻系统尚未分配的下一个事务ID，也就是`目前已出现过的事务ID的最大值+1`**

- 首先比较`DB_TRX_ID < up_limit_id`, 如果小于，则当前事务能看到`DB_TRX_ID` 所在的记录，如果大于等于进入下一个判断
- 接下来判断 `DB_TRX_ID 大于等于 low_limit_id` , 如果大于等于则代表`DB_TRX_ID` 所在的记录在`Read View`生成后才出现的，那对当前事务肯定不可见，如果小于则进入下一个判断
- 判断`DB_TRX_ID` 是否在活跃事务之中，`trx_list.contains(DB_TRX_ID)`，如果在，则代表我`Read View`生成时刻，你这个事务还在活跃，还没有Commit，你修改的数据，我当前事务也是看不见的；如果不在，则说明，你这个事务在`Read View`生成之前就已经Commit了，你修改的结果，我当前事务是能看见的

------

#### 整体流程

我们在了解了`隐式字段`，`undo log`， 以及`Read View`的概念之后，就可以来看看MVCC实现的整体流程是怎么样了

**整体的流程是怎么样的呢？我们可以模拟一下**

- 当`事务2`对某行数据执行了`快照读`，数据库为该行数据生成一个`Read View`读视图，假设当前事务ID为`2`，此时还有`事务1`和`事务3`在活跃中，`事务4`在`事务2`快照读前一刻提交更新了，所以Read View记录了系统当前活跃事务1，3的ID，维护在一个列表上，假设我们称为`trx_list`

| 事务1    | 事务2    | 事务3    | 事务4        |
| :------- | :------- | :------- | :----------- |
| 事务开始 | 事务开始 | 事务开始 | 事务开始     |
| …        | …        | …        | 修改且已提交 |
| 进行中   | 快照读   | 进行中   |              |
| …        | …        | …        |              |

- Read View不仅仅会通过一个列表`trx_list`来维护`事务2`执行`快照读`那刻系统正活跃的事务ID，还会有两个属性`up_limit_id`（**记录trx_list列表中事务ID最小的ID**），`low_limit_id`(**记录trx_list列表中事务ID最大的ID，也有人说快照读那刻系统尚未分配的下一个事务ID也就是`目前已出现过的事务ID的最大值+1`，我更倾向于后者** [>>>资料传送门 | 呵呵一笑百媚生的回答](https://www.zhihu.com/question/66320138/answer/241418502)) ；所以在这里例子中`up_limit_id`就是1，`low_limit_id`就是4 + 1 = 5，trx_list集合的值是1,3，`Read View`如下图

[![img](http://118.25.100.99/img/img_interview_file1/68.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[img](javascript:void(0))



- 我们的例子中，只有`事务4`修改过该行记录，并在`事务2`执行`快照读`前，就提交了事务，所以当前该行当前数据的`undo log`如下图所示；我们的事务2在快照读该行记录的时候，就会拿该行记录的`DB_TRX_ID`去跟`up_limit_id`,`low_limit_id`和`活跃事务ID列表(trx_list)`进行比较，判断当前`事务2`能看到该记录的版本是哪个。

[![img](http://118.25.100.99/img/img_interview_file1/69.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[img](javascript:void(0))



- 所以先拿该记录`DB_TRX_ID`字段记录的事务ID `4`去跟`Read View`的的`up_limit_id`比较，看`4`是否小于`up_limit_id`(1)，所以不符合条件，继续判断 `4` 是否大于等于 `low_limit_id`(5)，也不符合条件，最后判断`4`是否处于`trx_list`中的活跃事务, 最后发现事务ID为`4`的事务不在当前活跃事务列表中, 符合可见性条件，所以`事务4`修改后提交的最新结果对`事务2`快照读时是可见的，所以`事务2`能读到的最新数据记录是`事务4`所提交的版本，而事务4提交的版本也是全局角度上最新的版本

[![在这里插入图片描述](http://118.25.100.99/img/img_interview_file1/70.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[在这里插入图片描述](javascript:void(0))



- 也正是Read View生成时机的不同，从而造成RC,RR级别下快照读的结果的不同

### MVCC相关问题

------

#### RR是如何在RC级的基础上解决不可重复读的？

##### **当前读和快照读在RR级别下的区别：**

```
表1:
```

| 事务A                       | 事务B                                      |
| :-------------------------- | :----------------------------------------- |
| 开启事务                    | 开启事务                                   |
| 快照读(无影响)查询金额为500 | 快照读查询金额为500                        |
| 更新金额为400               |                                            |
| 提交事务                    |                                            |
|                             | select `快照读`金额为500                   |
|                             | select lock in share mode`当前读`金额为400 |

在上表的顺序下，事务B的在事务A提交修改后的快照读是旧版本数据，而当前读是实时新数据400

```
表2:
```

| 事务A                         | 事务B                                      |
| :---------------------------- | :----------------------------------------- |
| 开启事务                      | 开启事务                                   |
| 快照读（无影响）查询金额为500 |                                            |
| 更新金额为400                 |                                            |
| 提交事务                      |                                            |
|                               | select `快照读`金额为400                   |
|                               | select lock in share mode`当前读`金额为400 |

而在`表2`这里的顺序中，事务B在事务A提交后的快照读和当前读都是实时的新数据400，这是为什么呢？

- 这里与上表的唯一区别仅仅是`表1`的事务B在事务A修改金额前`快照读`过一次金额数据，而`表2`的事务B在事务A修改金额前没有进行过快照读。

**所以我们知道事务中快照读的结果是非常依赖该事务首次出现快照读的地方，即某个事务中首次出现快照读的地方非常关键，它有决定该事务后续快照读结果的能力**

**我们这里测试的是`更新`，同时`删除`和`更新`也是一样的，如果事务B的快照读是在事务A操作之后进行的，事务B的快照读也是能读取到最新的数据的**

------

#### RC,RR级别下的InnoDB快照读有什么不同？

正是`Read View`生成时机的不同，从而造成RC,RR级别下快照读的结果的不同

- 在RR级别下的某个事务的对某条记录的第一次快照读会创建一个快照及Read View, 将当前系统活跃的其他事务记录起来，此后在调用快照读的时候，还是使用的是同一个Read View，所以只要当前事务在其他事务提交更新之前使用过快照读，那么之后的快照读使用的都是同一个Read View，所以对之后的修改不可见；
- 即RR级别下，快照读生成Read View时，Read View会记录此时所有其他活动事务的快照，这些事务的修改对于当前事务都是不可见的。而早于Read View创建的事务所做的修改均是可见
- 而在RC级别下的，事务中，每次快照读都会新生成一个快照和Read View, 这就是我们在RC级别下的事务中可以看到别的事务提交的更新的原因

**总之在RC隔离级别下，是每个快照读都会生成并获取最新的Read View；而在RR隔离级别下，则是同一个事务中的第一个快照读才会创建Read View, 之后的快照读获取的都是同一个Read View。**

## MySQL中innodb引擎，主键索引和非主键索引在数据存储方面有什么差异



在 InnoDB 里，主键索引的叶子节点存的是整行数据。主键索引也被称为聚簇索引（clustered index）。非主键索引的叶子节点内容是主键的值。在 InnoDB 里，非主键索引也被称为二级索引（secondary index）。

## 分库分表



分库：因为单个物理数据库的资源跟连接数是有限制的，所以在数据量或者并发量较大的情况下，需要将一个数据库拆分成多个数据库。

分表：垂直分表、水平分表。

## 什么是垂直分表，什么是水平分表



垂直分表：适用于一个表有很多个字段的情况，将原先一个表中的某些字段拆出来，单独放到一个或者多个表。

水平分表：根据具体的切分规则，将数据划分到不同的表上面。

## 水平切分的切分策略



范围切分：比如根据日期的范围进行切分，2019年的数据放一个表，2020年的数据放一个表。

哈希切分：根据数据的某个字段（通常是id）哈希值，进行取模，由此决定某条数据应该被路由到哪张表里。

## 分布式id怎么生成



一般有以下几种方案：

redis自增；

雪花算法；

MySQL自增主键；

uuid；

号段模式（每次生成一个id区间，并缓存在本地，用完了再去取新的id区间）

## 分库分表有什么问题？



分布式事务、跨库join、count、全局排序

## 分库分表如何不停机做数据迁移？



### 需求说明

> 类似订单表，用户表这种未来规模上亿甚至上十亿百亿的海量数据表，在项目初期为了快速上线，一般只是单表设计，不需要考虑分库分表。随着业务的发展，单表容量超过千万甚至达到亿级别以上，这时候就需要考虑分库分表这个问题了，而**不停机分库分表迁移**，这应该是分库分表最基本的需求，毕竟互联网项目不可能挂个广告牌**"今晚10:00~次日10:00系统停机维护"**，这得多low呀，以后跳槽面试，你跟面试官说这个迁移方案，面试官怎么想呀？

### 借鉴codis

笔者正好曾经碰到过这个问题，并借鉴了**codis**一些思想实现了不停机分库分表迁移方案；codis不是这篇文章的重点，这里只提及借鉴codis的地方--rebalance：

> 当迁移过程中发生数据访问时，Proxy会发送“SLOTSMGRTTAGSLOT”迁移命令给Redis，强制将客户端要访问的Key立刻迁移，然后再处理客户端的请求。（ SLOTSMGRTTAGSLOT 是codis基于redis定制的）

### 分库分表

明白这个方案后，了解**不停机分库分表迁移**就比较容易了，接下来详细介绍笔者当初对`installed_app`表的实施方案；即用户已安装的APP信息表；

#### 确定sharding column

确定`sharding column`绝对是分库分表最最最重要的环节，没有之一。sharding column直接决定整个分库分表方案最终是否能成功落地；一个合适的sharding column的选取，基本上能让与这个表相关的绝大部分流量接口都能通过这个sharding column访问分库分表后的单表，而不需要跨库跨表，最常见的sharding column就是`user_id`，笔记这里选取的也是`user_id`；

#### 分库分表方案

根据自身的业务选取最合适的sharding column后，就要确定分库分表方案了。笔者采用`主动迁移`与`被动迁移`相结合的方案：

1. **主动迁移**就是一个独立程序，遍历需要分库分表的`installed_app`表，将数据迁移到分库分表后的目标表中。
2. **被动迁移**就是与`installed_app`表相关的业务代码自身将数据迁移到分库分表后对应的表中。

接下来详细介绍这两个方案；

##### 主动迁移

主动迁移就是一个独立的外挂迁移程序，其作用是遍历需要分库分表的`installed_app`表，将这里的数据**复制**到分库分表后的目标表中，由于`主动迁移`和`被动迁移`会一起运行，所以需要处理主动迁移和被动迁移碰撞的问题，笔者的`主动迁移`伪代码如下：

```java
public void migrate(){
    // 查询出当前表的最大ID, 用于判断是否迁移完成
    long maxId = execute("select max(id) from installed_app");
    long tempMinId = 0L;
    long stepSize = 1000;
    long tempMaxId = 0L;
    do{
        try {
            tempMaxId = tempMinId + stepSize;
            // 根据InnoDB索引特性, where id>=? and id<?这种SQL性能最高
            String scanSql = "select * from installed_app where id>=#{tempMinId} and id<#{tempMaxId}";
            List<InstalledApp> installedApps = executeSql(scanSql);
            Iterator<InstalledApp> iterator = installedApps.iterator();
            while (iterator.hasNext()) {
                InstalledApp installedApp = iterator.next();
                // help GC
                iterator.remove();
                long userId = installedApp.getUserId();
                String status = executeRedis("get MigrateStatus:${userId}");
                if ("COMPLETED".equals(status)) {
                    // migration finish, nothing to do
                    continue;
                }
                if ("MIGRATING".equals(status)) {
                    // "被动迁移" migrating, nothing to do
                    continue;
                }
                // 迁移前先获取锁: set MigrateStatus:18 MIGRATING ex 3600 nx
                String result = executeRedis("set MigrateStatus:${userId} MIGRATING ex 86400 nx");
                if ("OK".equals(result)) {
                    // 成功获取锁后, 先将这个用户所有已安装的app查询出来[即迁移过程以用户ID维度进行迁移]
                    String sql = "select * from installed_app where user_id=#{user_id}";
                    List<InstalledApp> userInstalledApps = executeSql(sql);
                    // 将这个用户所有已安装的app迁移到分库分表后的表中(有user_id就能得到分库分表后的具体的表)
                    shardingInsertSql(userInstalledApps);
                    // 迁移完成后, 修改缓存状态
                    executeRedis("setex MigrateStatus:${userId} 864000 COMPLETED");
                } else {
                    // 如果没有获取到锁, 说明被动迁移已经拿到了锁, 那么迁移交给被动迁移即可[这种概率很低]
                    // 也可以加强这里的逻辑, "被动迁移"过程不可能持续很长时间, 可以尝试循环几次获取状态判断是否迁移完
                    logger.info("Migration conflict. userId = {}", userId);
                }
            }
            if (tempMaxId >= maxId) {
                // 更新max(id)，最终确认是否遍历完成
                maxId = execute("select max(id) from installed_app");
            }
            logger.info("Migration process id = {}", tempMaxId);
        }catch (Throwable e){
            // 如果执行过程中有任何异常(这种异常只可能是redis和mysql抛出来的), 那么退出, 修复问题后再迁移
            // 并且将tempMinId的值置为logger.info("Migration process id="+tempMaxId);日志最后一次记录的id, 防止重复迁移
            System.exit(0);
        }
        tempMinId += stepSize;
    }while (tempMaxId < maxId);
}
```

这里有几点需要注意：

1. 第一步查询出max(id)是为了尽量减少max(id)的查询次数，假如第一次查询max(id)为10000000，那么直到遍历的id到10000000以前，都不需要再次查询max(id)；
2. 根据`id>=? and id遍历，而不要根据`id>=? limit n`或者`limit m, n`进行遍历，因为limit性能一般，且会随着遍历越往后，性能越差。而`id>=? and id这种遍历方式即使会有一些踩空，也没有任何影响，且整个性能曲线非常平顺，不会有任何抖动；迁移程序毕竟是辅助程序，不能对业务程序有过多的影响；
3. 根据id区间范围查询出来的`List`要转换为`Iterator`，每迭代处理完一个userId，要remove掉，否则可能导致GC异常，甚至OOM；

##### 被动迁移

被动迁移就是在正常与`installed_app`表相关的业务逻辑前插入了迁移逻辑，以新增用户已安装APP为例，其伪代码如下：

```java
// 被动迁移方法是公用逻辑，所以与`installed_app`表相关的业务逻辑前都需要调用这个方法；
public void migratePassive(long userId)throws Exception{
    String status = executeRedis("get MigrateStatus:${userId}");
    if ("COMPLETED".equals(status)) {
        // 该用户数据已经迁移完成, nothing to do
        logger.info("user's installed app migration completed. user_id = {}", userId);
    }else if ("MIGRATING".equals(status)) {
        // "被动迁移" migrating, 等待直到迁移完成; 为了防止死循环, 可以增加最大等待时间逻辑
        do{
            Thread.sleep(10);
            status = executeRedis("get MigrateStatus:${userId}");
        }while ("COMPLETED".equals(status));
    }else {
        // 准备迁移
        String result = executeRedis("set MigrateStatus:${userId} MIGRATING ex 86400 nx");
        if ("OK".equals(result)) {
            // 成功获取锁后, 先将这个用户所有已安装的app查询出来[即迁移过程以用户ID维度进行迁移]
            String sql = "select * from installed_app where user_id=#{user_id}";
            List<InstalledApp> userInstalledApps = executeSql(sql);
            // 将这个用户所有已安装的app迁移到分库分表后的表中(有user_id就能得到分库分表后的具体的表)
            shardingInsertSql(userInstalledApps);
            // 迁移完成后, 修改缓存状态
            executeRedis("setex MigrateStatus:${userId} 864000 COMPLETED");
        }else {
            // 如果没有获取到锁, 应该是其他地方先获取到了锁并正在迁移, 可以尝试等待, 直到迁移完成
        }
    }
}
// 与`installed_app`表相关的业务--新增用户已安装的APP
public void addInstalledApp(InstalledApp installedApp) throws Exception{
    // 先尝试被动迁移
    migratePassive(installedApp.getUserId());
    // 将用户已安装app信息(installedApp)插入到分库分表后的目标表中
    shardingInsertSql(installedApp);
}
```

无论是CRUD中哪种操作，先根据缓存中`MigrateStatus:${userId}`的值进行判断：

1. 如果值为COMPLETED，表示已经迁移完成，那么将请求转移到分库分表后的表中进行处理即可；
2. 如果值为MIGRATING，表示正在迁移中，可以循环等待直到值为COMPLETED即迁移完成后，再将请求转移到分库分表后的表中进行处理处理；
3. 否则值为空，那么尝试获取锁再进行数据迁移。迁移完成后，将缓存值更新为COMPLETED，最后再将请求转移到分库分表后的表中进行处理处理；

#### 方案完善

当所有数据迁移完成后，CRUD操作还是会先根据缓存中`MigrateStatus:${userId}`的值进行判断，数据迁移完成后这一步已经是多余的。可以加个总开关，当所有数据迁移完成后，将这个开关的值通过类似TOPIC的方式发送，所有服务接收到TOPIC后将开关local cache化。那么接下来服务的CRUD都不需要先根据缓存中`MigrateStatus:${userId}`的值进行判断；

#### 遗留工作

迁移完成后，将`主动迁移`程序下线，并将`被动迁移`程序中对`migratePassive()`的调用全部去掉，并可以集成一些第三方分库分表中间件，例如`sharding-jdbc`，可以参考sharding-jdbc集成实战

### 回顾总结

回顾这个方案，最大的缺点就是如果碰到sharding column（例如userId）的总记录数比较多，且主动迁移正在进行中，被动迁移与主动迁移碰撞，那么被动迁移可能需要等待较长时间。

不过根据DB性能，一般批量插入1000条数据都是10ms级别，并且同一sharding column的记录分库分表后只属于一张表，不涉及跨表。所以，只要在迁移前先通过sql统计待迁移表中没有这类异常sharding column即可放心迁移；

笔者当初迁移`installed_app`表时，用户最多也只拥有不超过200个APP，所以不需要过多考虑碰撞带来的性能问题；没有万能的方案，但是有适合自己的方案；

如果有那种上万条记录的sharding column，可以把这些sharding column先缓存起来，迁移程序在夜间上线，优先迁移这些缓存的sharding column的数据，就可以尽可能的降低迁移程序对这些用户的体验。当然你也可以使用你想出来的更好的方案。

## innodb里删除一条记录，内部是怎么处理的？



记录头信息里的delete_mask标记位设置为1（表示该记录已删除），同时将记录从记录行链表中断开，并加入到垃圾链表中，垃圾链表的空间后续可以复用。

## 两大类索引



> 使用的存储引擎：MySQL5.7 InnoDB

#### 聚簇索引

```
如果表设置了主键，则主键就是聚簇索引* 如果表没有主键，则会默认第一个NOT NULL，且唯一（UNIQUE）的列作为聚簇索引* 以上都没有，则会默认创建一个隐藏的row_id作为聚簇索引
```

> InnoDB的聚簇索引的叶子节点存储的是行记录（其实是页结构，一个页包含多行数据），InnoDB必须要有至少一个聚簇索引。
>
> 由此可见，使用聚簇索引查询会很快，因为可以直接定位到行记录。

#### 普通索引

> 普通索引也叫二级索引，除聚簇索引外的索引，即非聚簇索引。
>
> InnoDB的普通索引叶子节点存储的是主键（聚簇索引）的值，而MyISAM的普通索引存储的是记录指针。

## 示例



#### 建表

```
mysql> create table user(    -> id int(10) auto_increment,    -> name varchar(30),    -> age tinyint(4),    -> primary key (id),    -> index idx_age (age)    -> )engine=innodb charset=utf8mb4;
```

> id 字段是聚簇索引，age 字段是普通索引（二级索引）

#### 填充数据

```
insert into user(name,age) values('张三',30);insert into user(name,age) values('李四',20);insert into user(name,age) values('王五',40);insert into user(name,age) values('刘八',10);
mysql> select * from user;+----+--------+------+| id | name  | age |+----+--------+------+| 1 | 张三  |  30 || 2 | 李四  |  20 || 3 | 王五  |  40 || 4 | 刘八  |  10 |+----+--------+------+
```

#### 索引存储结构

> id 是主键，所以是聚簇索引，其叶子节点存储的是对应行记录的数据

[![图片](http://118.25.100.99/img/img_mysql/12.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[图片](javascript:void(0))

聚簇索引（ClusteredIndex）



> age 是普通索引（二级索引），非聚簇索引，其叶子节点存储的是聚簇索引的的值

[![图片](http://118.25.100.99/img/img_mysql/11.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[图片](javascript:void(0))

普通索引（SecondaryIndex）



> 如果查询条件为主键（聚簇索引），则只需扫描一次B+树即可通过聚簇索引定位到要查找的行记录数据。
>
> 如：select * from user where id = 1;

[![图片](http://118.25.100.99/img/img_mysql/10.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[图片](javascript:void(0))

聚簇索引查找过程



> 如果查询条件为普通索引（非聚簇索引），需要扫描两次B+树，第一次扫描通过普通索引定位到聚簇索引的值，然后第二次扫描通过聚簇索引的值定位到要查找的行记录数据。
>
> 如：select * from user where age = 30;

```
1. 先通过普通索引 age=30 定位到主键值 id=12. 再通过聚集索引 id=1 定位到行记录数据
```

[![图片](http://118.25.100.99/img/img_mysql/9.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[图片](javascript:void(0))

普通索引查找过程第一步



[![图片](http://118.25.100.99/img/img_mysql/8.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[图片](javascript:void(0))

普通索引查找过程第二步



## 回表查询



> 先通过普通索引的值定位聚簇索引值，再通过聚簇索引的值定位行记录数据，需要扫描两次索引B+树，它的性能较扫一遍索引树更低。

## 索引覆盖



> 只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，速度更快。
>
> 例如：select id,age from user where age = 10;

## 如何实现覆盖索引



> 常见的方法是：将被查询的字段，建立到联合索引里去。
>
> 1、如实现：select id,age from user where age = 10;
>
> explain分析：因为age是普通索引，使用到了age索引，通过一次扫描B+树即可查询到相应的结果，这样就实现了覆盖索引

[![图片](http://118.25.100.99/img/img_mysql/7.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[图片](javascript:void(0))



> 2、实现：select id,age,name from user where age = 10;
>
> explain分析：age是普通索引，但name列不在索引树上，所以通过age索引在查询到id和age的值后，需要进行回表再查询name的值。此时的Extra列的NULL表示进行了回表查询

[![图片](http://118.25.100.99/img/img_mysql/6.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[图片](javascript:void(0))



> 为了实现索引覆盖，需要建组合索引idx_age_name(age,name)

```
drop index idx_age on user;create index idx_age_name on user(`age`,`name`);
```

> explain分析：此时字段age和name是组合索引idx_age_name，查询的字段id、age、name的值刚刚都在索引树上，只需扫描一次组合索引B+树即可，这就是实现了索引覆盖，此时的Extra字段为Using index表示使用了索引覆盖。

[![图片](http://118.25.100.99/img/img_mysql/5.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[图片](javascript:void(0))



## 哪些场景适合使用索引覆盖来优化SQL



#### 全表count查询优化

```
mysql> create table user(    -> id int(10) auto_increment,    -> name varchar(30),    -> age tinyint(4),    -> primary key (id),    -> )engine=innodb charset=utf8mb4;
```

> 例如：select count(age) from user;

[![图片](http://118.25.100.99/img/img_mysql/4.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[图片](javascript:void(0))



> 使用索引覆盖优化：创建age字段索引

```
create index idx_age on user(age);
```

[![图片](http://118.25.100.99/img/img_mysql/3.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[图片](javascript:void(0))



#### 列查询回表优化

> 前文在描述索引覆盖使用的例子就是
>
> 例如：select id,age,name from user where age = 10;
>
> 使用索引覆盖：建组合索引idx_age_name(age,name)即可

#### 分页查询

> 例如：select id,age,name from user order by age limit 100,2;
>
> 因为name字段不是索引，所以在分页查询需要进行回表查询，此时Extra为Using filesort文件排序，查询性能低下。

[![图片](http://118.25.100.99/img/img_mysql/2.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))

[图片](javascript:void(0))



> 使用索引覆盖：建组合索引idx_age_name(age,name)

[![图片](http://118.25.100.99/img/img_mysql/1.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)](javascript:void(0))