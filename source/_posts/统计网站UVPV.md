---
title: 统计网站UVPV
date: 2025-02-20 18:49:15
tags: Redis
category: Java
---

### 系统活跃度指标

描述系统活跃度的名词：PV、UV、VV、IP
PV(Page View)  页面浏览量。每当一个页面被打卡或被刷新，都会产生一次PV。
UV(Unique Visitor)  独立访客。一天内相同访客多次访问网站，只计算为1个独立访客。
VV(VIsit View) 访客访问的次数。
IP 独立IP访问数

### 为什么选择HyperLogLog统计UV、PV
在说明 HyperLogLog 之前，我们需要先了解一个概念：基数统计。维基百科中的解释是：

cardinality of a set is a measure of the “number of elements“ of the set
它的意思是：一个集合（注意：这里集合的含义是 Object 的聚合，可以包含重复元素）中不重复元素的个数。例如集合 {1,2,3,1,2}，它有5个元素，但它的基数/Distinct 数为3。

Redis 最常用的数据结构有字符串、列表、字典、集合和有序集合。后来，由于 Redis 的广泛应用，Redis 自身也做了很多补充，其中就有 HyperLogLog（2.8.9 版本添加）结构。HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常大时，计算基数所需的空间总是固定的、并且是很小的。

### Redis HyperLogLog 命令
Redis 为 HyperLogLog提供了三个命令：PFADD、PFCOUNT、PFMERGE。我们依次来看看这三个命令的解释和作用。

**PFADD**
将任意数量的元素添加到指定的 HyperLogLog 里面。时间复杂度： 每添加一个元素的复杂度为 O(1) 。如果 HyperLogLog 估计的近似基数（approximated cardinality）在命令执行之后出现了变化， 那么命令返回 1 ， 否则返回 0 。 如果命令执行时给定的键不存在， 那么程序将先创建一个空的 HyperLogLog 结构， 然后再执行命令。
命令行示例
```
# 命令格式：PFADD key element [element …]
# 如果给定的键不存在，那么命令会创建一个空的 HyperLogLog，并向客户端返回 1
127.0.0.1:6379> PFADD ip_20190301 "192.168.0.1" "192.168.0.2" "192.168.0.3"
(integer) 1
# 元素估计数量没有变化，返回 0（因为 192.168.0.1 已经存在）
127.0.0.1:6379> PFADD ip_20190301 "192.168.0.1"
(integer) 0
# 添加一个不存在的元素，返回 1。注意，此时 HyperLogLog 内部存储会被更新，因为要记录新元素
127.0.0.1:6379> PFADD ip_20190301 "192.168.0.4"
(integer) 1
```
**PFCOUNT**
当 PFCOUNT key [key …] 命令作用于单个键时，返回储存在给定键的 HyperLogLog 的近似基数，如果键不存在，那么返回 0，复杂度为 O(1)，并且具有非常低的平均常数时间；
当 PFCOUNT key [key …] 命令作用于多个键时，返回所有给定 HyperLogLog 的并集的近似基数，这个近似基数是通过将所有给定 HyperLogLog 合并至一个临时 HyperLogLog 来计算得出的，复杂度为 O(N)，常数时间也比处理单个 HyperLogLog 时要大得多。
命令行示例
```
# 返回 ip_20190301 包含的唯一元素的近似数量
127.0.0.1:6379> PFCOUNT ip_20190301
(integer) 4
127.0.0.1:6379> PFADD ip_20190301 "192.168.0.5"
(integer) 1
127.0.0.1:6379> PFCOUNT ip_20190301
(integer) 5
127.0.0.1:6379> PFADD ip_20190302 "192.168.0.1" "192.168.0.6" "192.168.0.7"
(integer) 1
# 返回 ip_20190301 和 ip_20190302 包含的唯一元素的近似数量
127.0.0.1:6379> PFCOUNT ip_20190301 ip_20190302
(integer) 7
```
**PFMERGE**
将多个 HyperLogLog 合并（merge）为一个 HyperLogLog，合并后的 HyperLogLog 的基数接近于所有输入 HyperLogLog 的可见集合（observed set）的并集。时间复杂度是 O(N)，其中 N 为被合并的 HyperLogLog 数量，不过这个命令的常数复杂度比较高。
命令格式：PFMERGE destkey sourcekey [sourcekey …]，合并得出的 HyperLogLog 会被储存在 destkey 键里面，如果该键并不存在，那么命令在执行之前，会先为该键创建一个空的 HyperLogLog。
命令行示例
```
# ip_2019030102 是 ip_20190301 与 ip_20190302 并集
127.0.0.1:6379> PFMERGE ip_2019030102 ip_20190301 ip_20190302
OK
127.0.0.1:6379> PFCOUNT ip_2019030102
(integer) 7
```

### SpringBoot 中使用 Redis HyperLogLog
由于 Redis HyperLogLog 只有三个命令，思想和操作也非常简单，这里直接给出代码示例。
```
import com.imooc.ad.Application;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.HyperLogLogOperations;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * <h1>Redis HyperLogLog 测试用例</h1>
 * Created by Qinyi.
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {Application.class},
        webEnvironment = SpringBootTest.WebEnvironment.NONE)
public class RedisHyperLogLogTest {

    /** 注入 StringRedisTemplate, 使用默认配置 */
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
    @SuppressWarnings("all")
        public void testHyperLogLog() {

        HyperLogLogOperations operations = stringRedisTemplate.opsForHyperLogLog();

        // add 方法对应 PFADD 命令
        operations.add("ip_20190301", "192.168.0.1", "192.168.0.2", "192.168.0.3");
        // size 方法对应 PFCOUNT 命令
        System.out.println(operations.size("ip_20190301"));     // 3

        operations.add("ip_20190301", "192.168.0.1", "192.168.0.4");
        System.out.println(operations.size("ip_20190301"));     // 4

        operations.add("ip_20190302", "192.168.0.1", "192.168.0.5");
        System.out.println(operations.size("ip_20190302"));     // 2

        // union 方法对应 PFMERGE 命令
        operations.union("ip_201903", "ip_20190301", "ip_20190302");
        System.out.println(operations.size("ip_201903"));       // 5
    }
}
```