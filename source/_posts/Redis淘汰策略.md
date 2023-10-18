---
title: Redis淘汰策略
date: 2023-10-17 18:48:26
tags: Redis
category: Java
---
## volatile-lru：
从已设置过期时间的数据集中挑选最近最少使用的数据淘汰。
## volatile-ttl：
从已设置过期时间的数据集中挑选将要过期的数据淘汰。
## volatile-random：
从已设置过期时间的数据集中任意选择数据淘汰。
## volatile-lfu：
从已设置过期时间的数据集挑选使用频率最低的数据淘汰。
## allkeys-lru：
从数据集中挑选最近最少使用的数据淘汰
## allkeys-lfu：
从数据集中挑选使用频率最低的数据淘汰。
## allkeys-random：
从数据集（server.db[i].dict）中任意选择数据淘汰
## no-enviction（驱逐）：
禁止驱逐数据，这也是默认策略。意思是当内存不足以容纳新入数据时，新写入操作就会报错，请求可以继续进行，线上任务也不能持续进行，采用no-enviction策略可以保证数据不被丢失。
## 淘汰策略的意义
MySQL 里有 2000w 数据，redis 中只存 20w 的数据，如何保证 redis 中的数据都是热点数据？
redis 内存数据集大小上升到一定大小的时候，就会施行数据淘汰策略。
