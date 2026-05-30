---
title: Mybatis-Plus日志
date: 2024-09-13 21:23:46
tags: Mybatis-plus
---

## Mybatis-Plus日志
在项目中日志中经常会出现如下日志：
```
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@42607e80] was not registered for synchronization because synchronization is not active
JDBC Connection [com.alibaba.druid.proxy.jdbc.ConnectionProxyImpl@56ebc6bb] will not be managed by Spring
==>  Preparing: SELECT app_id, app_code, app_name FROM t_sys_application 
==> Parameters: 
<==    Columns: app_id, app_code, app_name
<==      Total: 2
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@42607e80]
```
在程序中并无代码体现，那么为什么会打印这些日志呢？
是因为Mybatis-plus自带的SQL日志，首先我们学习一下更基础更通用的mybatis示例。
开启的方法是是由org.apache.ibatis.logging.stdout.StdOutImpl控制的根据StdOutImpl.java可看出日志都是System.out.println(s);的控制台输出，配置及源码如下
```
mybatis:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl # 控制台输出日志
logging:
  level:
    com.XXX.dao: debug
```
将使用mybatis的类的level配置为debug，因为mybatis内部仅打印debug级别的SQL日志。
而本项目使用的Mybatis-plus配置起来更复杂一些
```
# mybatis-plus 配置内容
mybatis-plus:
  configuration:
  	### 开启驼峰配置
    map-underscore-to-camel-case: true # 虽然默认为 true ，但是还是显示去指定下。
  global-config:
    db-config:
      id-type: auto # ID 主键自增
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
  # xml扫描，多个目录用逗号或者分号分隔（告诉 Mapper 所对应的 XML 文件位置）
  mapper-locations: classpath*:mapper/*.xml
logging:
  level:
    com.XXX.dao: debug
    cn:
      iocoder:
  		springboot:
    	  lab12:
      		mybatis:
      		  mapper:info# 将Mapper 层设置日志级别为 info，这将记录较为简要的重要信息，通常包括 SQL 执行过程和结果
  config:classpath:log/logback-spring.xml
  file.path
```
这样配置为 MyBatis-Plus 的 SQL 执行提供更细粒度的日志输出，帮助开发者在开发和调试过程中看到详细的 SQL 语句和执行过程，同时在生产环境中记录必要的日志信息用于运维分析。