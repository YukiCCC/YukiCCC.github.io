---
title: DynamicDataSource多数据
date: 2024-09-30 22:43:00
tags: 数据库
---

## DynamicDataSource多数据
### 多数据源的典型使用场景
1、业务复杂
应用没有拆，数据库拆分了
2、读写分离
解决数据库的读性能瓶颈（读比写性能更高，写锁会影响阻塞，影响性能）

### 引入依赖
```
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>dynamic-datasource-spring-boot-starter</artifactId>
	<version>3.5.1</version>
</dependency>
```

### 配置多数据源
通过yaml配置主数据源，这里就只配置了一个主数据源，后续通过代码来自由的切换数据源。
```

spring:
  datasource:
    dynamic:
      hikari:
        connection-timeout: 5000
        idle-timeout: 30000 # 经过idle-timeout时间如果连接还处于空闲状态, 该连接会被回收
        min-idle: 5 # 池中维护的最小空闲连接数, 默认为 10 个
        max-pool-size: 16 # 池中最大连接数, 包括闲置和使用中的连接, 默认为 10 个
        max-lifetime: 60000 # 如果一个连接超过了时长，且没有被使用, 连接会被回收
        is-auto-commit: true
 
      primary: master #设置默认的数据源或者数据源组,默认值即为master
      strict: true #严格匹配数据源,默认false. true未匹配到指定数据源时抛异常,false使用默认数据源
      datasource:
        master: # 数据源名称
          url: 
          username: 
          password: 
          driver-class-name: com.mysql.cj.jdbc.Driver
          
# 如下，如果你是确定的几个数据源，可以直接都在yaml配置写死即可
#        slave_1:
#          url: 
#          username: 
#          password: 
#          driver-class-name: com.mysql.cj.jdbc.Driver
 
 
 
其中数据库连接池，所有的数据库统一配置，也可以单独配置，例如：
datasource:
        master: # 数据源名称
          url: 
          username: 
          password: 
          driver-class-name: com.mysql.cj.jdbc.Driver
          hikari:
            connection-timeout: 5000
            idle-timeout: 30000 # 经过idle-timeout时间如果连接还处于空闲状态, 该连接会被回收
            min-idle: 5 # 池中维护的最小空闲连接数, 默认为 10 个
            max-pool-size: 16 # 池中最大连接数, 包括闲置和使用中的连接, 默认为 10 个
            max-lifetime: 60000 # 如果一个连接超过了时长，且没有被使用, 连接会被回收
            is-auto-commit: true
```

### 切换数据源DS注解
```

@Service
@DS("common")
public class BookService extends ServiceImpl<BookMapper, Book> {
    @Resource
    private BookMapper bookMapper;
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void save(ReqDto reqDto) {
        bookMapper.save(reqDto);
    }

```