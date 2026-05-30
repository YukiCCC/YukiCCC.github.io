---
title: Redis在spring项目中的运用
date: 2025-03-06 19:28:20
tags: Redis
category: Java
---

Redis 是一种高性能的键值存储数据库，而 Spring Boot 是一个简化了开发过程的 Java 框架。将两者结合，可以轻松地在 Spring Boot 项目中使用 Redis 来实现数据缓存、会话管理和分布式锁等功能。

### 一、添加 Redis 依赖
在 pom.xml 文件中添加 Redis 相关依赖


```
<dependencies>
  <!-- Spring Data Redis -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
</dependencies>
```

### 二、配置 Redis 连接信息
在 application.properties 或 application.yml 配置文件中添加 Redis 连接信息
```
spring.redis.host=127.0.0.1
spring.redis.port=6379
```
根据自己 Redis 服务器配置，修改主机和端口信息

### 三、使用 RedisTemplate 进行操作
#### 1. 创建 RedisTemplate Bean
在配置类中创建 RedisTemplate Bean，用于进行 Redis 操作
```java
@Configuration
public class RedisConfig {

  @Bean
  public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
    RedisTemplate<String, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(connectionFactory);
    template.setKeySerializer(new StringRedisSerializer());
    template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
    return template;
  }
```
上述示例中使用了 JSON 序列化器来对值进行序列化和反序列化，你也可以根据需要选择其他序列化器。

#### 2. 注入 RedisTemplate
在服务类或控制器中注入 RedisTemplate
```java
@Autowired
private RedisTemplate<String, Object> redisTemplate;
```

#### 3. 执行 Redis 操作
使用注入的 RedisTemplate 来进行 Redis 操作，设置键值对、获取值等
```java
// 设置键值对
redisTemplate.opsForValue().set("key", "value");

// 获取值
String value = (String) redisTemplate.opsForValue().get("key");
```
### 四、使用 Spring Cache 简化缓存操作
#### 1. 添加 Spring Cache 依赖
```java
在 pom.xml 文件中添加 Spring Cache 相关依赖

<dependencies>
  <!-- Spring Data Redis -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>

  <!-- Spring Cache -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
  </dependency>
</dependencies>
```
#### 2. 启用缓存支持
在启动类上添加 @EnableCaching 注解，启用缓存支持
```java
@SpringBootApplication
@EnableCaching
public class Application {
  // ...
}
```
#### 3. 使用缓存注解
在服务类或方法上使用 Spring Cache 提供的缓存注解，如 @Cacheable、@CachePut、@CacheEvict
```java
@Service
public class demoService {

  @Cacheable(value = "cacheName", key = "#id")
  public Object getData(String id) {
    // 从数据库或其他数据源获取数据
    return data;
  }

  @CachePut(value = "cacheName", key = "#id")
  public Object updateData(String id, Object newData) {
    // 更新数据库或其他数据源的数据
    return newData;
  }

  @CacheEvict(value = "cacheName", key = "#id")
  public void deleteData(String id) {
    // 删除数据库或其他数据源的数据
  }
}
```
### 五、使用 Redisson 实现分布式锁
#### 1. 添加 Redisson 依赖
在 pom.xml 文件中添加 Redisson 相关依赖
```java
<dependencies>
  <!-- Spring Data Redis -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>

  <!-- Redisson -->
  <dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.16.3</version>
  </dependency>
</dependencies>
```
#### 2. 配置 Redisson
在配置文件中添加 Redisson 配置，例如 application.properties
```
# 单节点 Redisson 配置
spring.redis.redisson.config=classpath:redisson-single-node.yml
```
在 redisson-single-node.yml 配置文件中配置 Redisson 的连接信息

#### 3. 使用 Redisson 获取锁：
在你的代码中使用 Redisson 获取分布式锁：
```java
@Autowired
private RedissonClient redissonClient;

public void doSomething() {
  RLock lock = redissonClient.getLock("lockName");
  try {
    lock.lock();
    // 执行需要加锁的操作
  } finally {
    lock.unlock();
  }
}
```
### 六、完善 Redis 的其他配置
除了基本配置，还可以根据实际需求完善 Redis 的其他配置，例如连接池配置、超时设置等。

#### 一、连接池配置
Redis 使用连接池来管理和复用与 Redis 服务器的连接，以提高连接的效率和性能。

1. 在 配置文件中配置连接池相关参数
打开 Redis 配置文件 redis.conf，找到以下配置项并进行修改
```
# 最大连接数
maxclients 10000

# TCP 连接的队列长度
tcp-backlog 511

# 连接池中的最大空闲连接数
maxidle 100

# 连接池中的最小空闲连接数
minidle 10

# 连接超时时间（毫秒）
timeout 3000

# 是否开启 TCP 连接的保活机制
tcp-keepalive 0
```
2. 通过客户端连接池配置对象进行配置
在 Spring Boot 项目中，可以通过 Redis 连接池配置对象 JedisPoolConfig 进行配置
```java
@Configuration
public class RedisConfig {

  @Bean
  public JedisPoolConfig jedisPoolConfig() {
    JedisPoolConfig poolConfig = new JedisPoolConfig();
    poolConfig.setMaxTotal(10000);
    poolConfig.setMaxIdle(100);
    poolConfig.setMinIdle(10);
    poolConfig.setMaxWaitMillis(3000);
    // 其他配置项设置
    return poolConfig;
  }

  @Bean
  public JedisConnectionFactory jedisConnectionFactory(JedisPoolConfig poolConfig) {
    RedisStandaloneConfiguration config = new RedisStandaloneConfiguration();
    config.setHostName("127.0.0.1");
    config.setPort(6379);
    // 其他 Redis 配置项设置

    JedisClientConfiguration clientConfig = JedisClientConfiguration.builder()
        .usePooling().poolConfig(poolConfig)
        .build();
    
    return new JedisConnectionFactory(config, clientConfig);
  }

  @Bean
  public RedisTemplate<String, Object> redisTemplate(JedisConnectionFactory connectionFactory) {
    RedisTemplate<String, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(connectionFactory);
    template.setKeySerializer(new StringRedisSerializer());
    template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
    return template;
  }
}
```
#### 二、超时设置
超时设置用于控制 Redis 操作的超时时间，以防止长时间的阻塞或无响应操作

1. 配置 Redis 连接超时时间
在 Redis 配置文件中设置 timeout 参数，单位为毫秒，如下设置连接超时时间为 5000 毫秒
```
timeout 5000
```
2. 通过 Redis 客户端配置对象进行配置
通过 JedisClientConfiguration 进行配置，将读取操作的超时时间设置为 5秒
```java
@Bean
public JedisClientConfiguration jedisClientConfiguration() {
  Duration timeout = Duration.ofSeconds(5);

  return JedisClientConfiguration.builder()
      .readTimeout(timeout)
      .build();
}

@Bean
public JedisConnectionFactory jedisConnectionFactory(JedisClientConfiguration clientConfig) {
  // 其他配置项设置
  return new JedisConnectionFactory(config, clientConfig);
}
```
