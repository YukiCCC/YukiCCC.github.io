---
title: rabbitMQ
date: 2023-10-16 09:45:37
tags:
---
## 一、消息队列

## 1. 什么是消息队列

- 消息Message

  网络中的两台计算机或者两个通讯设备之间传递的数据。例如：文本、音乐、视频等内容。

- 队列 Queue

  一种特殊的线性表。只允许在首部删除元素和在尾部增加元素（FIFO)

- 消息队列（Message Queue）

  保存消息的队列。消息传输过程中的容器，具有存储消息的能力，提供生产、消费接口供外部调用做数据的存储和获取。

<img src="/01.01.同步处理.png" alt="01.01.同步处理" style="zoom: 33%;" />

<img src="/01.01.异步处理.png" alt="01.01.异步处理" style="zoom:33%;" />

## 同步调动的问题

耦合度高：每次加入新的需求，都要修改原来的代码

性能下降：调用者需要等待服务提供者响应，如果调用链过长，则响应时间等于每次调用的事件之和

资源浪费：调用链中的每条服务在等待响应中，不能释放请求占用的资源，高并发场景下会极度浪费系统资源

级联失败：如果服务提供者出现问题，所有调用方都会跟着出问题，迅速导致整个微服务群故障。

## 2. 使用消息队列的好处

- 解耦（类似Spring的IOC）
  允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。
- 可恢复性
  系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。
- 缓冲
  有助于控制和优化数据流经过系统的速度， 解决生产消息和消费消息的处理速度不一致的情况。
- 灵活性 & 峰值处理能力（削峰）
  在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。
- 异步通信
  很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。

## 3. 消息队列的两种模式

### 3.1 点对点的消息系统

一对一，消费者主动拉取数据，消息收到后消息清除。	

消息生产者生产消息发送到Queue中，然后消息消费者从Queue中取出并且消费消息。消息被消费以后， queue 中不再有存储，所以消息消费者不可能消费到已经被消费的消息。Queue 支持存在多个消费者，但是对一个消息而言，只会有一个消费者可以消费。最典型的例子就是订单处理系统，多个订单处理器可以同时工作，但是对于一个特定的订单，只有其中一个订单处理器可以拿到该订单进行处理。

<img src="/01.02.点对点消息.png" alt="01.02.点对点消息" style="zoom: 40%;" />

### 3.2 发布-订阅消息系统

一对多，消费者消费数据之后不会清除消息。
消息生产者（发布）将消息发布到 topic 中，同时有多个消息消费者（订阅）消费该消息。和点对点方式不同，发布到 topic 的消息会被所有订阅者消费。有两种方式，一种是队列主动推送模式、一种是消费者主动拉取模式。

<img src="/01.03.发布订阅模式.png" alt="01.03.发布订阅模式" style="zoom:40%;" />

主动推送方式：生产者一生产消息，就发送给消费者。问题：1：要维护订阅的消费者   2：消费者消费能力问题。

消费者拉取方式：维持长轮询，不停地访问是否有新的数据。问题：维持长轮询，即使没有数据，也要不停地询问。

## 4. 常见的消息系统

1. RabbitMQ 支持多协议AMQP、XMPP、SMTP、STOMP。支持负载均衡，数据持久化。同时支持Peer-to-Peer和发布/订阅。可靠性和稳定性

2. Redis基于key-value对的NoSQL数据库，通知支持MQ功能，可做轻量级队列服务使用。就入队而言，Redis对短消息（小于10kb）的性能比RabbitMQ好，长消息性能比RabblitMQ差。

3. Zoom 轻量级，不需要单独的消息服务器或中间件，应用本身扮演该角色，Peer-to-Peer。它本质上是一个库，需要开发人员自己组合多种技术，使用复杂度高。

4. ActiveMQ JMS实现，Peer-to-Peer，支持持久化、XA（分布式）事务

5. Kafka 高性能跨语言的分布式发布/订阅信息系统，数据持久化、全分布式，同时支持在线和离线处理。

6. MetaQ/RocketMQ 纯java实现，发布/订阅信息系统，支持本地事务和XA分布式事务。

   ![消息队列](/消息队列.png)

## 5. 常见使用场景

- 任务异步处理

  将不需要同步处理的并且耗时长的操作由消息队列通知消息接收方进行异步处理。提高应用程序的响应时间。

- 应用程序解耦合

  MQ相当于一个中介，生产方通过MQ与消费方交互，它将应用程序进行解耦合。

## 二、RabbitMQ基础

## 1. 什么是AMQP

> AMQP，即Advanced Message Queuing Protocol，一个提供统一消息服务的应用层标准高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。

AMQP是一套公开的消息队列协议，最早在2003年提出，它旨在从协议层定义消息通信数据的标准格式，为的就是解决MQ市场上协议不同意的问题。RabbitMQ就是遵循AMQP标准协议开发的MQ服务。

## 2. 什么是RabbitMQ

RabbitMQ是由erlang语言开发，基于AMQP（Advanced Message Queue 高级消息队列协议）协议实现的消息队列，它是一种应用程序之间的通信方法，消息队列在分布式系统开发中应用非常广泛。

为什么使用RabbitMQ呢？

- 使用简单，功能强大。
- 基于AMQP协议
- 社区活跃，文档完善。
- 高并发性能好，这主要得益于Erlang语言。
- Spring Boot默认已集成RabbitMQ



## 3. RabbitMQ安装

## docker安装

### 查看仓库

```bash
docker search rabbitmq
```



### 拉取镜像

```bash
docker pull rabbitmq
```



### 创建并启动容器

#### 指定用户名/密码启动

```shell
docker run -d --name rabbitmq \
	-p 5672:5672 -p 15672:15672 \
	-v `pwd`/data:/var/lib/rabbitmq \
	--hostname myRabbit \
	-e RABBITMQ_DEFAULT_VHOST=my_vhost  \
	-e RABBITMQ_DEFAULT_USER=admin -e \
	RABBITMQ_DEFAULT_PASS=admin rabbitmq
```

-d 后台运行容器；
	--name 指定容器名；
	-p 指定服务运行的端口（5672：应用访问端口；15672：控制台Web端口号）；
	-v 映射目录或文件；
	--hostname  主机名（RabbitMQ的一个重要注意事项是它根据所谓的 “节点名称” 存储数据，默认为主机名）；
	-e 指定环境变量；
		（RABBITMQ_DEFAULT_VHOST：默认虚拟机名；
		RABBITMQ_DEFAULT_USER：默认的用户名；
		RABBITMQ_DEFAULT_PASS：默认用户名的密码）

#### 不指定用户名密码启动

默认用户名和密码都是guest

```shell
docker run -d --name rabbitmq 	\
      -p 5672:5672 -p 15672:15672   rabbitmq
```



### 安装插件

```bash
docker exec -it rabbitmq bash
rabbitmq-plugins enable rabbitmq_management
```



### 客户端访问

```bash
http://192.168.33.10:15672
```





## 普通安装

### 创建配置文件

- 创建/etc/yum.repos.d/rabbitmq.repo文件，内容如下

```shell
[rabbitmq_erlang]
name=rabbitmq_erlang
baseurl=https://packagecloud.io/rabbitmq/erlang/el/7/$basearch
repo_gpgcheck=1
gpgcheck=1
enabled=1
# PackageCloud's repository key and RabbitMQ package signing key
gpgkey=https://packagecloud.io/rabbitmq/erlang/gpgkey
       https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300

##
## RabbitMQ server
##

[rabbitmq_server]
name=rabbitmq_server
baseurl=https://packagecloud.io/rabbitmq/rabbitmq-server/el/7/$basearch
repo_gpgcheck=1
gpgcheck=1
enabled=1
# PackageCloud's repository key and RabbitMQ package signing key
gpgkey=https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey
       https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300


[rabbitmq_erlang-source]
name=rabbitmq_erlang-source
baseurl=https://packagecloud.io/rabbitmq/erlang/el/7/SRPMS
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/rabbitmq/erlang/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300

[rabbitmq_server-source]
name=rabbitmq_server-source
baseurl=https://packagecloud.io/rabbitmq/rabbitmq-server/el/7/SRPMS
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
```



### 更新yum package （可以不执行）

```
yum update -y
```

### 安装

```shell
yum install socat logrotate -y

yum -y install erlang

yum install erlang rabbitmq-server -y
```

### 服务启动停止

```shell
#启动服务
/sbin/service rabbitmq-server start
#查看状态
/sbin/service rabbitmq-server status
#停止服务
/sbin/service rabbitmq-server stop
```

### 安装插件

```shell
rabbitmq-plugins enable rabbitmq_management
```

### 修改配置

如果linux上安装了浏览器，可以通过guest/guest登录（只能本机）

在/etc/rabbitmq下创建rabbitmq.conf

/etc/rabbitmq/rabbitmq.conf

```properties
loopback_users = none
#如果无效，则使用下面增加用户的方式
```

增加用户，并设置管理员权限（不要执行）

```shell
#增加用户
rabbitmqctl add_user admin admin
#设置权限
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
#设置管理员
rabbitmqctl set_user_tags admin administrator
```

### 输入测试

输入可以访问，如果不行，则可能是插件没有安装，尝试运行一下命令后再访问

用户名：密码 guest/guest

```
http://192.168.33.10:15672
```



## 三、RabbitMQ的工作原理

## 1.RabbitMQ基本结构

![03.01.基本结构](/03.01.基本结构.png)

组成部分说明如下：

-  Broker：消息队列服务进程，此进程包括两个部分：Exchange和Queue。 
- Exchange：消息队列交换机，按一定的规则将消息路由转发到某个队列，对消息进行过虑。 
- Queue：消息队列，存储消息的队列，消息到达队列并转发给指定的消费方。
- Producer：消息生产者，即生产方客户端，生产方客户端将消息发送到MQ。 
- Consumer：消息消费者，即消费方客户端，接收MQ转发的消息。

队列，交换机和绑定（队列和交换机）统称为AMQP实体（AMQP entities）。



消息发布接收流程
1、生产者和Broker建立TCP连接。
2、生产者和Broker建立通道。
3、生产者通过通道消息发送给Broker，由Exchange将消息进行转发。
4、Exchange将消息转发到指定的Queue（队列）

接收消息
1、消费者和Broker建立TCP连接
2、消费者和Broker建立通道
3、消费者监听指定的Queue（队列）
4、当有消息到达Queue时Broker默认将消息推送给消费者。
5、消费者接收到消息。



### HelloWorld基本案例

#### 生产者

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class Producer {
    //队列
    private static final String QUEUE = "helloworld";
    public static void main(String[] args) throws Exception {
        //通过连接工厂创建新的连接和mq建立连接
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("192.168.33.10");
        connectionFactory.setPort(5672);//端口
        connectionFactory.setUsername("admin");
        connectionFactory.setPassword("admin");
        //设置虚拟机，一个mq服务可以设置多个虚拟机，每个虚拟机就相当于一个独立的mq
        connectionFactory.setVirtualHost("/");

        Connection connection = null;
        Channel channel = null;
        //建立新连接
        connection = connectionFactory.newConnection();
        //创建会话通道,生产者和mq服务所有通信都在channel通道中完成
        channel = connection.createChannel();
        //声明队列，如果队列在mq 中没有则要创建
        //参数：String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments
        /**
         * 参数明细
         * 1、queue 队列名称
         * 2、durable 是否持久化，如果持久化，mq重启后队列还在
         * 3、exclusive 是否独占连接，队列只允许在该连接中访问，如果connection连接关闭队列则自动删除,如果将此参数设置true可用于临时队列的创建
         * 4、autoDelete 自动删除，队列不再使用时是否自动删除此队列，如果将此参数和exclusive参数设置为true就可以实现临时队列（队列不用了就自动删除）
         * 5、arguments 参数，可以设置一个队列的扩展参数，比如：可设置存活时间
         */
        channel.queueDeclare(QUEUE, true, false, false, null);
        //发送消息
        //参数：String exchange, String routingKey, BasicProperties props, byte[] body
        /**
         * 参数明细：
         * 1、exchange，交换机，如果不指定将使用mq的默认交换机（设置为""）
         * 2、routingKey，路由key，交换机根据路由key来将消息转发到指定的队列，如果使用默认交换机，routingKey设置为队列的名称
         * 3、props，消息的属性
         * 4、body，消息内容
         */
        //消息内容
        String message = "hello world";
        channel.basicPublish("", QUEUE, null, message.getBytes());
        System.out.println("send to mq " + message);
        //关闭连接，//先关闭通道
        channel.close();
        connection.close();

    }
}

```



#### 消费者

```java
import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Consumer {
    //队列
    private static final String QUEUE = "helloworld";

    public static void main(String[] args) throws IOException, TimeoutException {
        //通过连接工厂创建新的连接和mq建立连接
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("192.168.33.10");
        connectionFactory.setPort(5672);//端口
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");
        //设置虚拟机，一个mq服务可以设置多个虚拟机，每个虚拟机就相当于一个独立的mq
        connectionFactory.setVirtualHost("/");

        //建立新连接
        Connection connection = connectionFactory.newConnection();
        //创建会话通道,生产者和mq服务所有通信都在channel通道中完成
        Channel channel = connection.createChannel();

        //监听队列
        //声明队列，如果队列在mq 中没有则要创建
        //参数：String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments
        /**
         * 参数明细
         * 1、queue 队列名称
         * 2、durable 是否持久化，如果持久化，mq重启后队列还在
         * 3、exclusive 是否独占连接，队列只允许在该连接中访问，如果connection连接关闭队列则自动删除,如果将此参数设置true可用于临时队列的创建
         * 4、autoDelete 自动删除，队列不再使用时是否自动删除此队列，如果将此参数和exclusive参数设置为true就可以实现临时队列（队列不用了就自动删除）
         * 5、arguments 参数，可以设置一个队列的扩展参数，比如：可设置存活时间
         */
        channel.queueDeclare(QUEUE,true,false,false,null);

        //实现消费方法
        DefaultConsumer defaultConsumer = new DefaultConsumer(channel){

            /**
             * 当接收到消息后此方法将被调用
             * @param consumerTag  消费者标签，用来标识消费者的，在监听队列时设置channel.basicConsume
             * @param envelope 信封，通过envelope
             * @param properties 消息属性
             * @param body 消息内容
             * @throws IOException
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                //交换机
                String exchange = envelope.getExchange();
                //消息id，mq在channel中用来标识消息的id，可用于确认消息已接收
                long deliveryTag = envelope.getDeliveryTag();
                //消息内容
                String message= new String(body,"utf-8");
                System.out.println("receive message:"+message);
            }
        };

        //监听队列
        //参数：String queue, boolean autoAck, Consumer callback
        /**
         * 参数明细：
         * 1、queue 队列名称
         * 2、autoAck 自动回复，当消费者接收到消息后要告诉mq消息已接收，如果将此参数设置为tru表示会自动回复mq，如果设置为false要通过编程实现回复
         * 3、callback，消费方法，当消费者接收到消息要执行的方法
         */
        channel.basicConsume(QUEUE,true,defaultConsumer);

    }
}
```



## SpringAMQP

### 基本概念

​		AMQP，即Advanced Message Queuing Protocol，一个提供统一消息服务的应用层标准高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。Erlang中的实现有RabbitMQ等。

​		Spring AMQP是基于AMQP协议定义的一套API规范，提供了模板来放和接收消息。包含两个部分，其中spring-amqp是基础抽象，spring-rabbit是底层的默认实现。

特征

- 侦听器容器，用于异步处理入栈消息。
- 用于发送和接收消息的RabbitTemplate
- RabbitAdmin用于自动声明队列，交换和绑定

### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```



### 配置RabbitMQ

```

```



### 编写Producer

```java
@RestController
public class Producer {
    @Autowired
    RabbitTemplate rabbitTemplate;

    @RequestMapping("/simpleQueue")
    public String testSimpleQueue(){
        String queueName ="helloworld";
        String message = "hello,simple";
        rabbitTemplate.convertAndSend(queueName,message);
        return "success";
    }
}
```



### 编写Consumer

```java
@Component
public class Consumer {
    @RabbitListener(queues = "helloworld")
    public  void  listenSimpleQueue(String msg) throws InterruptedException {
        System.out.println("接收的消息：" + msg);
    }
}
```



## 2. 工作模式和交换机

rabbitmq支持以下集中工作模式：

​	workqueue：工作队列，不需要交换机，消息发送指定的某个队列当中去的，队列的消费者共同消费队列中的数据。

​	pub/sub ：发布订阅：扇形交换机

​	routing: 路由模式：直连交换机

​	topic:主题模式：主题交换机

​	rpc: rpc模式

交换机类型

​	Direct exchange（直连交换机）

​	Fanout exchange（扇型交换机）

​	Topic exchange（主题交换机）

​	Headers exchange（头交换机）



### Work queues

![04.消息队列-workqueuel](/04.消息队列-workqueuel.png)

消息产生者将消息放入队列消费者可以有多个,消费者1,消费者2,同时监听同一个队列,消息被消费?C1 C2共同争抢当前的消息队列内容,谁先拿到谁负责消费消息(隐患,高并发情况下,默认会产生某一个消息被多个消费者共同使用,可以设置一个开关(syncronize,与同步锁的性能不一样) 保证一条消息只能被一个消费者使用)。队列中的数据过多时会被丢弃。提高消息处理的速度，避免消息的队列

应用场景:红包;大项目中的资源调度(任务分配系统不需知道哪一个任务执行系统在空闲,直接将任务扔到消息队列中,空闲的系统自动争抢)

- 生产者

  ```java
  @RequestMapping("/workQueue")
  public String testWorkQueue() throws InterruptedException {
      String queueName ="helloworld";
      String message = "hello,simple";
      for (int i=1;i<50;i++){
          rabbitTemplate.convertAndSend(queueName,message + ",index:" + i);
          Thread.sleep(20);
      }
  
      return "success";
  }
  ```

- 消费者

  ```java
  @RabbitListener(queues = "helloworld")
  public  void  listenWorkQueue1(String msg) throws InterruptedException {
  	System.out.println("消费者1接收消息：" + msg);
  	Thread.sleep(20);
  }
  
  @RabbitListener(queues = "helloworld")
  public  void  listenWorkQueue2(String msg) throws InterruptedException {
  	System.out.println("消费者2接收消息：" + msg);
  	Thread.sleep(50);
  }
  ```

- 消息预取机制：当消息到达的时候，rabbitmq内部会预先取得消息，再后续再处理。
  preFetch消息预取限制，可以控制消息的上限。

  ```yml
  spring:
    rabbitmq:
      host: 192.168.33.10
      port: 5672
      virtual-host: /
      username: guest
      password: guest
      listener:
        simple:
          prefetch: 1
  ```

  

### Publish/Subscribe

交换机是用来发送消息的AMQP实体。交换机拿到一个消息之后将它路由给一个或零个队列。它使用哪种路由算法是由交换机类型和被称作绑定（bindings）的规则所决定的。

发布订阅模式交换机模型：

Direct exchange（直连交换机）

Fanout exchange（扇型交换机）

Topic exchange（主题交换机）

Headers exchange（头交换机）

![04.消息队列-Publish-Subscribe](/04.消息队列-Publish-Subscribe.png)

X代表交换机rabbitMQ内部组件，erlang 消息产生者是代码完成，代码的执行效率不高，消息产生者将消息放入交换机,交换机发布订阅把消息发送到所有消息队列中,对应消息队列的消费者拿到消息进行消费。

应用场景：:邮件群发,群聊天,广播(广告)

#### Fanout Exchange

生产者发送的消息，交换机会将消息路由到所有的队列中。

- 在消费者当中创建交换机，队列，并绑定交换机和队列。
- 定义消费者方法
- 定义生产者方法



创建交换机，队列以及绑定代码

```java
@Configuration
public class FanoutConfig {
    //声明交换机
    @Bean(name="fanoutExchange")
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("fanout.exchange");
    }

    //声明队列
    @Bean(name="fanoutQueue1")
    public Queue fanoutQueue1(){
        return new Queue("fanout.queue1");
    }

    @Bean
    public Binding bindingQueue1(@Qualifier("fanoutQueue1") Queue queue
            ,@Qualifier("fanoutExchange") FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(queue).to(fanoutExchange);
    }

    //声明队列
    @Bean(name="fanoutQueue2")
    public Queue fanoutQueue2(){
        return new Queue("fanout.queue2");
    }
    @Bean
    public Binding bindingQueue2(@Qualifier("fanoutQueue2") Queue queue
            ,@Qualifier("fanoutExchange") FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(queue).to(fanoutExchange);
    }
}

```

消费者代码

```java
@RabbitListener(queues = "fanout.queue1")
public  void  listenFanoutQueue1(String msg) throws InterruptedException {
	System.out.println("消费者1接收消息：" + msg);
	Thread.sleep(20);
}

@RabbitListener(queues = "fanout.queue2")
public  void  listenFanoutQueue2(String msg) throws InterruptedException {
	System.out.println("消费者2接收消息：" + msg);
	Thread.sleep(50);
}
```

生产者代码

```
@RequestMapping("/fanoutQueue")
public String testFanoutQueue() throws InterruptedException {
	String exchange = "fanout.exchange";
	String message = "hello,simple";
	for (int i=1;i<20;i++){
		//交换机，路由名，信息
		rabbitTemplate.convertAndSend(exchange,"",message);
		Thread.sleep(20);
	}
	return "fanoutQueue success";
}
```







### Routing （路由）

Direct Exchange：会将接收到的消息根据规则路由到指定的Queue，因此称为路由模式。

Routing模式要求队列在绑定交换机时要指定routingkey，消息会转发到符合routingkey的队列。

每一个Queue都与Exchange设置一个BindingKey
发布者发布消息时，指定消息的RoutingKey
Exchange将消息路由到BindingKey与消息RountingKey一致的队列。

![04.消息队列-routing](/04.消息队列-routing.png)

消息生产者将消息发送给交换机按照路由判断,路由是字符串(info) 当前产生的消息携带路由字符(对象的方法),交换机根据路由的key,只能匹配上路由key对应的消息队列,对应的消费者才能消费消息;

业务场景：从系统的代码逻辑中获取对应的功能字符串,将消息任务扔到对应的队列中业务场景:error 通知;EXCEPTION;错误通知的功能;传统意义的错误通知;客户通知;利用key路由,可以将程序中的错误封装成消息传入到消息队列中,开发者可以自定义消费者,实时接收错误;



//科技 tec  ，艺术 art



消费者代码

```java
@RabbitListener(bindings = @QueueBinding(
            value=@Queue(name = "direct.queue1"),
            exchange = @Exchange(name="direct.exchange",type = ExchangeTypes.DIRECT),
            key={"eat","jump"}
    ))
    public  void  listenRouteQueue1(String msg) throws InterruptedException {
        System.out.println("消费者1--Direct接收消息--eat：" + msg);
        Thread.sleep(20);
    }

    @RabbitListener(bindings = @QueueBinding(
            value=@Queue(name = "direct.queue2"),
            exchange = @Exchange(name="direct.exchange",type = ExchangeTypes.DIRECT),
            key={"jump"}
    ))
    public  void  listenRouteQueue2(String msg) throws InterruptedException {
        System.out.println("消费者1--Direct接收消息--jump：" + msg);
        Thread.sleep(20);
    }
```



生产者代码

```java
@RequestMapping("/routeQueue")
public String testRouteQueue() throws InterruptedException {
    String exchange = "direct.exchange";
    //交换机，路由名，信息
    rabbitTemplate.convertAndSend(exchange,"eat","...eat.....");
    rabbitTemplate.convertAndSend(exchange,"jump","...jump.....");

    return "routeQueue success";
}
```



### Topics 

TopicExchange与DirectExchange类似，区别在于routingKey必须是多个单词的列表，并且以.分隔。

Queue与exchange指定的BindingKey时，可以使用通配符：

*：代指一个单词

 ‘#’：代指零个或者多个单词 a.#.b  ==>a.x.b    ,a.x.y.z.b



![04.消息队列-topic](/04.消息队列-topic.png)

消费者代码

```java
@RabbitListener(bindings = @QueueBinding(
            value=@Queue(name = "topic.queue1"),
            exchange = @Exchange(name="topic.exchange",type = ExchangeTypes.TOPIC),
            key={"wanho.*"}
    ))
    public  void  listenTopicQueue1(String msg) throws InterruptedException {
        System.out.println("消费者1--topic接收消息--wanho：" + msg);
        Thread.sleep(20);
    }
    @RabbitListener(bindings = @QueueBinding(
            value=@Queue(name = "topic.queue2"),
            exchange = @Exchange(name="topic.exchange",type = ExchangeTypes.TOPIC),
            key={"*.news"}
    ))
    public  void  listenTopicQueue2(String msg) throws InterruptedException {
        System.out.println("消费者2--topic接收消息--news：" + msg);
        Thread.sleep(20);
    }
```

生产者代码

```
@RequestMapping("/topicQueue")
public String testTopicQueue() throws InterruptedException {
    String exchange = "topic.exchange";
    //交换机，路由名，信息
    rabbitTemplate.convertAndSend(exchange,"wanho.news","wanho,hahaha");
    rabbitTemplate.convertAndSend(exchange,"weather.news","sunny");

    return "routeQueue success";
}
```



### Header

header模式与routing不同的地方在于，header模式取消routingkey，使用header中的 key/value（键值对）匹配队列。

接收方

```
@RabbitListener(bindings = @QueueBinding(
            value=@Queue(name = "head.query1"),
            exchange = @Exchange(name="header.exchange",type = ExchangeTypes.HEADERS),
            key = {"sms"}
    ))
    public  void  listenHeadQueue1(byte[] bytes) throws InterruptedException {
    	System.out.println("消费者1--header接收消息--sms：" + new String(bytes));
    	Thread.sleep(20);
    }
```

发送方

```java
@RequestMapping("/headerQueue")
    public String testHeaderQueue() throws InterruptedException {
        String exchange = "header.exchange";
        MessageProperties messageProperties = new MessageProperties();
        messageProperties.setHeader("query","query1");
        //交换机，路由名，信息
        Message message = new Message("info".getBytes(),messageProperties);
        rabbitTemplate.convertAndSend(exchange,"",message);

        return "routeQueue success";
    }
```



### RPC



## 消息转换器

Spring对消息对象的处理是由org.springframework.amqp.support.converter.MessageConverter来处理的，默认实现是SimpleMessageConverter，基于JDK的ObjectOutputStream完成序列化。
如果要修改只需要定义一个MessageConverter类型的Bean即可。推荐用JSON方式序列化。

### 发送方

​      

在publisher服务声明MessageConverter

```
@Bean
public MessageConverter messageConverter(){
	return  new Jackson2JsonMessageConverter();
}
```

发送代码

```
@RequestMapping("/objectQueue")
    public String testObjectQueue(){
        String queueName ="object.queue";
        Map<String,String> map = new HashMap<>();
        map.put("name","dola");
        map.put("age","5");
        rabbitTemplate.convertAndSend(queueName,map);
        return "success";
    }
```

### 接收方

引入依赖和声明MessageConverter参考发送方

接收方代码：需要用和发送消息一致的类型接收。

```java
@RabbitListener(queues = "helloworld")
public  void  listenWorkQueue2(Map<String,String> map) throws InterruptedException {
    System.out.println("name：" + map.get("name"));
    System.out.println("age：" + map.get("age"));
    Thread.sleep(50);
}
```





## 非存储型交换机和队列

存储型交换机和队列：RabbitMQ会将消息保存在磁盘，如果服务器再次启动，会从磁盘将数据读入内存。

非存储型交换机和队列：数据仅仅放在内存当中，当服务重启或者发生宕机之后，数据就会不存在。

**持久化队列和交换机，是要牺牲一部分性能。**

## 设置超时时间以及长度

在创建队列时，使用map参数进行设置

x-message-ttl：超时参数，时间单位是毫秒

x-max-length:     队列中最多能存储的数量

==》死信

## 死信队列

死信：这些信息没有被处理，直接被丢掉了。

​	消息过期

​	队列达到最大的长度

​	消息被拒绝（basicReject/basicNack），并且requeue=false

死信队列：用来处理死信的队列

用来处理死信的交换机称为死信交换机，用来处理死信的队列，称之为死信队列

创建一个普通队列  normalqueue（消费者罢工），队列设置过期时间以及长度，

​	当出现死信的时候，配置死信交换机以及队列

## 配置属性

x-dead-letter-exchange： 配置死信交换机
x-dead-letter-routing-key：死信路由key



## 使用案例

```java
@Configuration
public class DeathConfig {

    @Bean("normal.exchange")
    public FanoutExchange normalExchange(){
        return new FanoutExchange("normal");
    }

    @Bean("normal.queue")
    public Queue normalQueue(){
        Map<String,Object> args = new HashMap<>();
        //超时
        args.put("x-message-ttl",3000);
        //最大长度
        args.put("x-max-length",5);
        //死信交换机(用死信交换机的名字--rabbitmq当中的名字)
        args.put("x-dead-letter-exchange","death");
        //设置死信队列的路由key
        //args.put("x-dead-letter-routing-key","tec");
        return new Queue("normal",true,false,false,args);
    }

    @Bean
    public Binding normalBinding(@Qualifier("normal.exchange")FanoutExchange fanoutExchange
            ,@Qualifier("normal.queue")Queue queue){
        return BindingBuilder.bind(queue).to(fanoutExchange);
    }


    //死信相关
    @Bean("death.exchange")
    public FanoutExchange deathExchange(){
        return new FanoutExchange("death");
    }

    @Bean("death.queue")
    public Queue DeathQueue(){
        return new Queue("death");
    }

    @Bean
    public Binding deathBinding(@Qualifier("death.exchange")FanoutExchange fanoutExchange
            ,@Qualifier("death.queue")Queue queue){
        return BindingBuilder.bind(queue).to(fanoutExchange);
    }
```





## 消息确认

默认情况下，都是自动确认，需要手工确认的情况下，需要配置

## 生产者消息确认

### 配置

```yaml
spring:
  rabbitmq:
    host: 192.168.33.10
    port: 5672
    virtual-host: /
    username: guest
    password: guest
    listener:
      direct:
        acknowledge-mode: manual #配置确认模式为手动确认
    publisher-confirm-type: correlated #生产者需要手工确认
```



### 生产者代码修改

```java
//定义一个特殊的属性对象
    RabbitTemplate.ConfirmCallback confirmCallback = new RabbitTemplate.ConfirmCallback() {
        @Override
        public void confirm(CorrelationData correlationData, boolean b, String s) {
            System.out.println("消息发送id：" +  correlationData.getId()  + "，发送状态：" +  b );
        }
    };

    @RequestMapping("/cfm")
    public String testCfm(){
        String message = "hello,死信消息";

        rabbitTemplate.setConfirmCallback(confirmCallback);
        CorrelationData correlationData = new CorrelationData();
        System.out.println("发送的id:" + correlationData.getId());
        rabbitTemplate.convertAndSend("normal","",message ,correlationData);

        return "cfg success";
    }
```



## 消费者的消息确认

### 进行配置

```yaml
spring:
  rabbitmq:
    host: 192.168.33.10
    port: 5672
    virtual-host: /
    username: guest
    password: guest
    listener:
      simple:
        prefetch: 2 #消息的预取限制
        acknowledge-mode: manual #手工确认
      direct:
        acknowledge-mode: manual  #手工确认
```



### 确认和拒绝

```java
@RabbitListener(queues = "normal")
    public void normalConsumer(Channel channel, Message message) throws IOException {
        System.out.println("正常消费者：" +message);
        long deliveryTag = message.getMessageProperties().getDeliveryTag();
        System.out.println("deliveryTag: " + deliveryTag);
        //消费者确认消息
        channel.basicAck(deliveryTag ,true);
        //拒绝消息,requeue为false，则变成死信
        //channel.basicReject(deliveryTag,false);
        //channel.basicReject(deliveryTag,true);
        //未确认消息重新入队
        //channel.basicRecover(true);
    }
```



## 常见问题

## RabbitMQ有哪几种交换机

Direct exchange（直连交换机）

Fanout exchange（扇型交换机）

Topic exchange（主题交换机）

Headers exchange（头交换机）

## RabbitMQ有几种工作模式

Work queues

Publish/Subscribe

Routing 

Topics 

Header

RPC

## 惰性队列

默认情况下，生产者将消息发送到队列后，默认是存储在内存当中，这样可以将消息发送给消费者。即使是持久化队列，也会在内存当中留一个备份。当需要释放内存的时候，将内存中的数据写入磁盘。

在某些特殊的情况，消息的生产和消费不是同一时间的场合下，可以设置惰性队列，无论队列是否是持久化，默认将数据写入磁盘，目的主要是为了减少内存的消耗。当需要消费消息的时候，再从磁盘读入内如。

拿时间换空间，可以部分解决消息堆积问题。

```
map.put("x-queue-mode","lazy")
```



## 作业

业务：注册用户，给用户发送邮件，用户在邮件中点击链接，确认登录（修改状态位），如果超过24小时不确认，则删除此用户。

​	解决方案：

​				 注册时，发送一条消息（用来发送邮件），邮件确认后，修改状态

​							超时24未确认：

​									采用消息队列，使用扇形交换机，同时发送两个队列，一个用来处理邮件，一个作为延时队列（设置消息的有效时间），此队列没有消费者，指定另外的死信交换机，死信交换机的消费者，根据状态，确定是否要删除用户（未确认，则删除）。**使用直连交换机，两条消息，一条用来发送邮件，一个用来处理过期。**

​									定时任务：查询数据库，到目前位置，注册时间已经到24小时，但是用户状态依旧是未确认的，将这些数据直接删除掉。

​	解决方案2：注册完（写入数据库） --直接发送邮件，通过rabbitmq发送一条消息（用来确认用户是否注册），此消息没有消费者，而是在超时之后，直接转入死信队列，由死信消费者来进行处理。

​	



美团，饿了么订单，用户生成订单后(未付款)，付款后状态发生变化，超过15分钟未付款，则删除订单。

​	解决：生成订单时，直接通过消息队列发送一条消息（15分钟），消息没有对应的消费，转入死信队列，消费者（死信）获取到信息，判断数据状态位，如果为未付款，则删除此订单。



目的：练习rabbitmq收发消息，同时要查阅官网，学习如何发送邮件

1：编写一个注册页面

​	用户名，密码，电话，邮件地址  （存入数据库，默认状态为0--未确认）

   注册处理：

​	写入数据库

​	并且将用户的信息，通过rabbitmq发送到指定的队列当中

2：rabbitmq的消费者

​	从队列当中拿到用户信息（用户名，邮箱）

​	向指定邮箱发送邮件，邮件中带有用户信息，当用户点击连接的时候，要将数据库当中的状态从0（未确认状态）--1(已确认)

3：超时删除

​	如果用户注册后超过24小时未确认，则删除表中的用户信息





