---
title: RabbitMq入门到实践
date: 2024-12-30 19:07:05
updated: 2024-12-30 19:07:05
tags:
  - 分布式
  - RabbitMq
comments: true
categories:
  - 分布式
  - RabbitMq
thumbnail: https://images.unsplash.com/photo-1682687220509-61b8a906ca19?crop=entropy&cs=srgb&fm=jpg&ixid=M3w2NDU1OTF8MHwxfHJhbmRvbXx8fHx8fHx8fDE3NDMwODM0NjJ8&ixlib=rb-4.0.3&q=85&w=1920&h=1080
---

## 1. centos7 安装 rabbitmq （rpm）

erlang地址：https://packagecloud.io/rabbitmq/erlang

rabbitmq地址：https://packagecloud.io/rabbitmq/rabbitmq-server

### 1.1 更新源

> yum update -y

### 1.2 安装编译器环境

> yum -y install gcc glibc-devel make ncurses-devel openssl-devel xmlto perl wget gtk2-devel binutils-devel

### 1.3 安装erlang语言环境，新增erlang库地址

> curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash

### 1.4 安装erlang语言环境

> sudo yum install erlang-22.3.4.10-1.el7.x86_64

### 1.5 查看版本号

> erl -version

### 1.6 新增rabbitmq packagecloud库地址

> curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash

### 1.7 安装rabbitmq

> yum install rabbitmq-server-3.8.8-1.el7.noarch

### 1.8 启动服务

> systemctl start rabbitmq-server

### 1.9 安装web管理页面

> rabbitmq-plugins enable rabbitmq_management

![image-20220215100441544](images/image-20220215100441544.png)

### 1.10 重启服务

> systemctl restart rabbitmq-server

### 1.11 访问服务

![image-20220215100700103](images/image-20220215100700103.png)

### 1.12 添加用户

> rabbitmqctl add_user root root  #添加用户
>
> rabbitmqctl set_permissions -p / root ".*" ".*" ".*"    #添加权限
>
> rabbitmqctl set_user_tags root administrator  #设置权限为管理员

![image-20220215100858867](images/image-20220215100858867.png)

## 2. centos7安装rabbitmq（源码）

### 2.1 安装依赖环境

> ```
> yum -y install gcc glibc-devel make ncurses-devel openssl-devel xmlto perl wget gtk2-devel binutils-devel
> ```

### 2.2 下载erlang语言环境

> ```
> wget http://erlang.org/download/otp_src_22.0.tar.gz  #下载到对应的路径中
> ```

### 2.3 解压

> ```
> tar -zxvf otp_src_22.0.tar.gz
> mv otp_src_22.0 /usr/local/  #移动到对应的路径下（自定义）
> ```

### 2.4 创建安装路径

> ```
> cd /usr/local/otp_src_22.0/  #切换到解压的路径下面
> mkdir ../erlang
> ./configure --prefix=/usr/local/erlang  #配置erlang install的安装路径
> ```

### 2.5 安装

> make install
>
> /usr/local/erlang/bin #查看是否安装成功

### 2.6 添加环境变量

> ```
> echo 'export PATH=$PATH:/usr/local/erlang/bin' >> /etc/profile
> source /etc/profile
> ```

### 2.7 查看版本号

>  erl --version

### 2.8 下载rabbitmq

> 地址：https://github.com/rabbitmq/rabbitmq-server/releases
>
> wget wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.15/rabbitmq-server-generic-unix-3.7.15.tar.xz  #下载安装包
>
> yum install -y xz  #下载xz工具进行解压

### 2.9 安装rabittmq

> /bin/xz -d rabbitmq-server-3.7.15.tar.xz
>
> tar -xvf rabbitmq-server-3.7.15.tar.xz
>
> mv /usr/local/rabbitmq_server-3.7.15  rabbitmq # 将文件夹进行改名

### 2.10 配置环境变量

> echo 'export PATH=$PATH:/usr/local/rabbitmq/sbin' >> /etc/profile
>
> source /etc/profile

### 2.11 服务启动

> rabbitmq-server -detached

### 2.12 安装WEB管理

> rabbitmq-plugins enable rabbitmq_management

## 3. windows安装rabbitmq

> https://www.cnblogs.com/saryli/p/9729591.html

## 4. 相关概念

​	通常谈到队列服务，会有三个概念：发消息者、队列、收消息者，RabbitMq在这个基础概念之上，多做了一层抽象，在发送消息和接收消息之间加入了交换器（Exchange）。发送消息者跟收消息者没有直接联系，发消息者把消息给交换器，交换器根据调度策略把消息再给队列

- 虚拟主机：一个虚拟机持有一组交换机、队列和绑定。再rabbitmq中，只能以虚拟主机的粒度进行控制权限，如果想要禁止A组访问B组的交换机/队列/绑定，必须为A和B分别创建虚拟主机
- 交换机：Exchange用于转发消息，它不会做存储，如果没有Queue bind绑定到Exchange的话，会直接丢弃Producer发送过来的消息。消息到交换机需要通过 **路由键** 进行转发到具体的队列中
- 绑定：交换机需要和队列相绑定，多对多关系

## 5. 交换机

交换机不存储消息，只做消息的转换，在启用ack模式后，交换机找不到队列会返回错误

- Direct：直接交换机，”先匹配，再投送“，即在绑定时设置一个routing_key，消息的routing_key匹配时才会被交换机进行投送
- Topic：按规则转发消息
- Headers：设置 header attribute参数类型的交换机
- Fanout：转发消息到所有绑定队列

### 5.1 Direct直接交换机

rabbitmq默认交换机，采用全文匹配的方式，例如：T - D1 绑定ey为 QD1，如果发送消息中的路由键匹配到QD1，就会进行转发

### 5.1 Topic交换机

Topic Exchange转发消息主要是根据通配符。这种交换机会和队列绑定一种格式，通配符就要在这种路由模式和路由键之间匹配后交换机才能转发消息

- 路由键必须是字符串，用句号(.) 隔开，例如：agreements.us
- 路由匹配模式中的（\*），主要用于匹配路由键指定位置的单词。例如：agreements.*.b，就会去匹配第一个arrements，以及第三个 b
- 路由匹配模式中的（#）, 表示多个单词 。例如： agreements.b.# 只要路由键 agreements.b 开头都会匹配

### 5.3 Headers交换机

相比较direct和topic固定的使用routing_key，headers则是一个自定义匹配规则类型，在队列与交换器绑定时会定义一组规则，只要消息中的键值对有一对匹配或者全部匹配时，就会被投送到对应队列

### 5.4 Fanout扇形交换机

Fanout Exchange消息广播的模式，不管路由键或者是路由模式，会把消息发送给绑定给它的全部队列，如果配置了routing_key 会被忽略

## 6. Spring Boot集成Rabbitmq

### 6.1 引入依赖包

使用spring封装的amqp进行操作

```maven
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### 6.2 组件对象

- Queue：抽象队列对象，
  - durable：是否持久化，如果为false，rabbitmq启动时就会自动删除队列
  - exclusive：声明了exclusive属性的队列只对首次声明它的连接可见，并且在连接断开时自动删除
  - autoDelete：所有客户端都断开连接时自动删除（用于临时使用）
- Message
- Exchange
- Binding

### 6.3 消息队列

```java
@Configuration
public class QueueConfiguration {

    @Bean
    public Queue queue() {
        return new Queue(RabbitMqConst.HELLO_QUEUE_NAME);
    }
}
```

### 6.4 生产者

```java
@Component
public class SimpleSender {

    @Resource
    private AmqpTemplate amqpTemplate;

    public void send() {
        String context = "helloMQ:" + new Date().getTime();
        amqpTemplate.convertAndSend(RabbitMqConst.HELLO_QUEUE_NAME, context);
    }
}
```

### 6.5 消费者

```java
@Component
@RabbitListener(queues = RabbitMqConst.HELLO_QUEUE_NAME)
public class HelloReceiver {
    /**
     * 处理方法
     * @param hello
     */
    @RabbitHandler
    public void process(String hello) {
        System.out.println("接收到数据：" + hello);
    }
}
```

### 6.6 测试方法

```java
@SpringBootTest(classes = SpringRabbitmqDemoApplication.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class SpringRabbitmqDemoApplicationTests {

    @Resource
    private SimpleSender simpleSender;

    @Test
    public void hello() {
        simpleSender.send();
    }
}
```

## 7. 交换机测试

**注意：默认所有创建的队列以及交换机都会持久化，测试代码时可以设置为false，autoDelete可以设置为true**

### 7.1 Direct交换机

```java
@Configuration
public class QueueConfiguration {

    /**
     * 创建队列：指定队列名称
     * @return
     */
    @Bean
    public Queue queue() {
        return new Queue(RabbitMqConst.HELLO_QUEUE_NAME);
    }

    /**
     * 创建一个直接交换机，指定交换机的名称
     *
     * @return
     */
    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange(RabbitMqConst.DIRECT_EXCHANGE_NAME);
    }
	
    /**
     * 创建一个绑定对象，主要的目的是将，交换机和队列绑定起来，hello 是routingkey
     * 在发送数据时，只有routingkey对应上了之后才会发送到对应的队列里面
     *
     * @return
     */
    @Bean
    public Binding bindingToDirect() {
        return BindingBuilder.bind(queue()).to(directExchange()).with("hello");
    }
}
```



```java
@Component
public class SimpleSender {

    @Resource
    private AmqpTemplate amqpTemplate;

    public void sendToDirect() {
        String context = "sendDirect:" + new Date().getTime();
        //注意第二个参数需要跟Binding中指定的routingkey对应上
        amqpTemplate.convertAndSend(RabbitMqConst.DIRECT_EXCHANGE_NAME, "hello", context);
    }
}
```



### 7.2 Topic交换机

```java
@Configuration
public class QueueConfiguration {

    /**
     * 创建队列：指定队列名称
     * @return
     */
    @Bean
    public Queue queue() {
        // 构造方法接收三个参数：durable（是否持久化）, exclusive（是否是特殊化，最对第一个连接可见，断开后就删除）, autoDelete（是否自动删除）
        return new Queue(RabbitMqConst.HELLO_QUEUE_NAME, false, false, false);
    }

    /**
     * 创建一个topic交换机
     * @return
     */
    @Bean
    public TopicExchange topicExchange() {
        return new TopicExchange(RabbitMqConst.TOPIC_EXCHANGE_NAME);
    }

    /**
     * 自定routingkey以 * 或 # 进行匹配
     * @return
     */
    @Bean
    public Binding bindingToTopic() {
        return BindingBuilder.bind(queue()).to(topicExchange()).with("hello.*（#）.world");
    }
}
```

### 7.3 Fanout交换机

```java
@Configuration
public class QueueConfiguration {

    /**
     * 创建队列：指定队列名称
     * @return
     */
    @Bean
    public Queue queue() {
        // 构造方法接收三个参数：durable（是否持久化）, exclusive（是否是特殊化，最对第一个连接可见，断开后就删除）, autoDelete（是否自动删除）
        return new Queue(RabbitMqConst.HELLO_QUEUE_NAME, false, false, false);
    }

    @Bean
    public FanoutExchange fanoutExchange() {
        return new FanoutExchange(RabbitMqConst.FANOUT_EXCHANGE_NAME);
    }
	/**
     * 扇形交换机不需要指定routingkey，会转发给绑定的所有队列
     * @return
     */
    @Bean
    public Binding bindingToFanout() {
        return BindingBuilder.bind(queue()).to(fanoutExchange());
    }
}

	public void sendToFanout() {
        String context = "sendFanout:" + new Date().getTime();
        //发送时，如果指定了routingkey也会失效
        amqpTemplate.convertAndSend(RabbitMqConst.FANOUT_EXCHANGE_NAME, "hello.1.world", context);
    }
```

### 7.4 Headers交换机

```java
@Configuration
public class QueueConfiguration {

    /**
     * 创建队列：指定队列名称
     * @return
     */
    @Bean
    public Queue queue() {
        // 构造方法接收三个参数：durable（是否持久化）, exclusive（是否是特殊化，最对第一个连接可见，断开后就删除）, autoDelete（是否自动删除）
        return new Queue(RabbitMqConst.HELLO_QUEUE_NAME, false, false, false);
    }

    @Bean
    public HeadersExchange headersExchange() {
        return new HeadersExchange(RabbitMqConst.HEADERS_EXCHANGE_NAME);
    }

    /**
     * 设置头中数据是否匹配，只有匹配时才进行转发
     * @return
     */
    @Bean
    public Binding bindingToHeaders() {
        //匹配单个KV值
        Binding single = BindingBuilder.bind(queue()).to(headersExchange()).where("msg").matches("hello");
        //多个KV值进行匹配
        Map<String, Object> map = new HashMap<>();
        map.put("msg1", "hello1");
        map.put("msg2", "hello2");
        Binding many = BindingBuilder.bind(queue()).to(headersExchange()).whereAny(map).match();
        return many;
    }
}

public void sendToHeaders() {
        String context = "sendHeaders:" + new Date().getTime();

        MessageProperties properties = MessagePropertiesBuilder
            .newInstance()
            .setHeader("msg2", "hello2")
            .setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN)
            .build();

        Message message = MessageBuilder
            .withBody(context.getBytes())
            .andProperties(properties)
            .build();
        amqpTemplate.send(RabbitMqConst.HEADERS_EXCHANGE_NAME, "", message);
    }
```

- whereAny：匹配多个KV中的任意一个
- whereAll：所有的KV都必须匹配到

## 8. 消息确认机制

```yml
spring:
  rabbitmq:
    host: 192.168.92.190
    port: 5672
    username: root
    password: root
    publisher-confirms: true  # 消息发送到交换机是否成功的回调函数
    publisher-returns: true   # 消息发送到对应队列失败调用的回调
    virtual-host: /
    listener:
      simple:
        acknowledge-mode: manual # 采用手动答应
        concurrency: 1 # 指定最小的消费者数量
        max-concurrency: 1 # 指定最大的消费者数量
        retry:
          enabled: true #开启消息重试
```

### 8.1 生产者

```java
@Component
public class SimpleSender {

    @Resource
    private AmqpTemplate amqpTemplate;

    public void send() {
        RabbitTemplate rabbitTemplate = (RabbitTemplate) this.amqpTemplate;
        //交换机投递到对应队列失败时，根据当前Mandatory 判断是否需要调用 ReturnCallback，配置文件中需要配置publisher-returns:true
        rabbitTemplate.setMandatory(true);

        rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            if (ack) {
                System.out.println("消息成功发送到exchange....");
            } else {
                System.out.println("消息发送exchange失败，原因："+ cause);
            }
        });
		//交换机转发消息到对应队列失败时调用
        rabbitTemplate.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
            String correlationId = message.getMessageProperties().getCorrelationId();
            System.out.printf("消息：%s，发送失败，应答码：%s，原因：%s，交换机：%s，路由键：%s%n", correlationId, replyCode, replyText, exchange, routingKey);
        });

        String context = "helloMQ:" + new Date().getTime();
        this.amqpTemplate.convertAndSend(RabbitMqConst.DIRECT_EXCHANGE_NAME, "hello", context);
    }
}
```

- ConfirmCallback：消息确认函数，当前函数主要的目的是，消息是否发送到交换机时回调的函数
- ReturnCallback：消息返回回调函数，当前消息成功投递到交换机，但是没有投递到对应的队列时调用

### 8.2 消费者

```java
/**
     * 处理方法
     * @param
     */
    @RabbitHandler
    @RabbitListener(queues = RabbitMqConst.HELLO_QUEUE_NAME)
    public void process(Message message, Channel channel) throws IOException {
        System.out.println("接收到数据：" + new String(message.getBody()));
        //消费者手动确认消息，如果没有确认，broker会将数据存储后继续发送
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), true);
    }
```

## 9. 死信队列

​	DLX 全称（Dead-Letter-Exchange）,称之为死信交换器，当消息变成一个死信之后，如果这个消息所在的队列存在 **x-dead-letter-exchange** 参数，那么它会被发送到 **x-dead-letter-exchange** 对应值的交换器上，这个交换器就称之为死信交换器，与这个死信交换器绑定的队列就是死信队列

- 消息被拒绝了 Basic.Reject或Basic.Nack）并且设置 requeue 参数的值为 false
- 消息过期了
- 队列达到最大长度

### 9.1 队列配置

```java
@Configuration
public class DeadQueueConfiguration {


    /**
     * 创建一个正常的队列，设置参数死信队列交换机的名称
     * @return
     */
    @Bean
    public Queue queue() {
        return QueueBuilder
            .nonDurable(RabbitMqConst.TEST_DLX_QUEUE_NAME)
            .withArgument("x-dead-letter-exchange", RabbitMqConst.DLX_EXCHANGE)
            .build();
    }

    /**
     * 创建一个直接交换机
     *
     * @return
     */
    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange(RabbitMqConst.TEST_DLX_EXCHANGE);
    }

    @Bean
    public Binding bindingToDirect() {
        return BindingBuilder.bind(queue()).to(directExchange()).with("hello");
    }

    /**
     * ------------------------------------------------------------------------------- 创建死信队列
     * @return
     */
    @Bean
    public Queue deadQueue() {
        return QueueBuilder
            .nonDurable(RabbitMqConst.DLX_QUEUE_NAME)
            .build();
    }

    @Bean
    public Exchange deadExchange() {
        return ExchangeBuilder
            .topicExchange(RabbitMqConst.DLX_EXCHANGE)
            .build();
    }

    /**
     * 将死信队列绑定到对应的交换机上 routingKey 以 #号进行订阅，任意 routingKey都会被转发到队列上
     */
    @Bean
    public Binding bindingToDead() {
        return BindingBuilder.bind(deadQueue()).to(deadExchange()).with("#").noargs();
    }
}
```



```java
@Component
public class DeadSender {

    @Resource
    private AmqpTemplate amqpTemplate;

    public void send() {
        String context = "dead msg:" + new Date().getTime();

        MessageProperties properties = MessagePropertiesBuilder
            .newInstance()
            //设置过期时间为10秒
            .setExpiration("10000")
            .build();

        Message message = MessageBuilder
            .withBody(context.getBytes())
            .andProperties(properties)
            .build();

        //发送到直接交换机上
        amqpTemplate.convertAndSend(RabbitMqConst.TEST_DLX_EXCHANGE, "hello", message);
    }

}
```

test_dlx_queue：为正常队列，发送消息时，指定10秒钟过期，因为没有开消费者，所以就自动转发到了dlx_queue上面

![image-20220215165156129](images/image-20220215165156129.png)

## 10. 延时队列

### 10.1 下载插件

> wget https://objects.githubusercontent.com/github-production-release-asset-2e65be/924551/bdcc937b-781e-4952-b2e2-a66c8aa94336?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220215%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220215T015044Z&X-Amz-Expires=300&X-Amz-Signature=5f16f24403f3cba33a2e4a99f4eeef41bf3cb5ca5c05bd59468e4f5fd9e8b8f3&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=924551&response-content-disposition=attachment%3B%20filename%3Drabbitmq-server-3.9.8-1.el7.src.rpm&response-content-type=application%2Foctet-stream

找到服务安装路径下面的plugins路径，将下载的 rabbitmq_delayed_message_exchange 插件拷贝进去

> mv  rabbitmq_delayed_message_exchange-3.8.0.ez /usr/lib/rabbitmq/lib/rabbitmq_server-3.8.8/plugins/

 启动：

> rabbitmq-plugins enable rabbitmq_delayed_message_exchange

### 10.2 延迟队列配置

```java
@Configuration
public class DelayConfiguration {

    
    @Bean
    public Exchange delayExchange() {
        Map<String, Object> map = new HashMap<>();
        //指定路由类型为直接交换机
        map.put("x-delayed-type", "direct");
        // 第二个交换机的类型自定义设置为 x-delayed-message 延迟队列指定
        return new CustomExchange(RabbitMqConst.DELAY_EXCHANGE, "x-delayed-message", true, false, map);
    }

    @Bean
    public Queue delayQueue() {
        return QueueBuilder
            .nonDurable(RabbitMqConst.DELAY_QUEUE)
            //设置队列最大长度为1
            .withArgument("x-max-length", 1)
            .build();
    }

    @Bean
    public Binding bindingToDelay() {
        return BindingBuilder.bind(delayQueue()).to(delayExchange()).with("delay.test").noargs();
    }

}
```

### 10.3 生产者

```java
@Component
public class DelaySender {

    @Resource
    private AmqpTemplate amqpTemplate;

    public void send(Integer delay) {
        String context = "delay msg:" + new Date().getTime();

        MessageProperties properties = MessagePropertiesBuilder
            .newInstance()
            .build();

        Message message = MessageBuilder
            .withBody(context.getBytes())
            .andProperties(properties)
            .build();

        //发送到直接交换机上
        amqpTemplate.convertAndSend(RabbitMqConst.DELAY_EXCHANGE, "delay.test", message, p -> {
            //进行消息拦截处理，给消息设置延时时间
            p.getMessageProperties().setDelay(delay);
            return p;
        });
    }

}
```

### 10.4 消费者

```java
@Component
public class DelayReceiver {

    /**
     * 处理方法
     * @param
     */
    @RabbitHandler
    @RabbitListener(queues = RabbitMqConst.DELAY_QUEUE)
    public void process(Message message, Channel channel) throws IOException {
        System.out.println("监听到延时数据，进行处理：" + new String(message.getBody()));
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), true);
    }
}
```

## 11. 实战场景

说明：订单服务进行创建订单后，进行状态变更为对应的下单状态，向延时队列中发送数据在5分钟之内没有进行付款的订单进行超时操作；订单状态入库之后再调用商品库存服务进行对应的商品扣减，商品扣减完成返回数据到订单服务修改订单的状态为已完成

### 1. 涉及服务

- micro-middleware : 中间件服务
- micro-order: 订单服务
- micro-product: 商品服务

### 2. 服务配置文件

#### 2.1 micro-middleware

```yml
spring:
  rabbitmq:
    host: 192.168.92.190
    port: 5672
    username: root
    password: root
    virtual-host: /
  application:
    name: micro-middleware
server:
  port: 9000
```

#### 2.2 micro-order

```yml
server:
  port: 8901 #端口号定义
spring:
  # 引入rabbitmq配置信息
  rabbitmq:
    host: 192.168.92.190
    port: 5672
    username: root
    password: root
    publisher-confirms: true #设置回调方法
    publisher-returns: true  #设置队列回调方法
    virtual-host: /
    listener:
      simple:
        acknowledge-mode: manual #采用手动回复消息
        concurrency: 1 # 并发数
        max-concurrency: 1 # 最大并发数
        retry:
          enabled: true # 是否重试
  application:
    name: micro-order
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/order?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull&useSSL=false&allowPublicKeyRetrieval=true
    username: root
    password: root
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    serialization:
      write-dates-as-timestamps: false
mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
    auto-mapping-behavior: full
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  mapper-locations: classpath*:mapper/*Mapper.xml
  global-config:
    db-config:
      logic-not-delete-value: 1
      logic-delete-value: 0
```

#### 2.3 micro-product

```yml
server:
  port: 8902
spring:
  rabbitmq:
    host: 192.168.92.190
    port: 5672
    username: root
    password: root
    publisher-confirms: true
    publisher-returns: true
    virtual-host: /
    listener:
      simple:
        acknowledge-mode: manual
        concurrency: 1
        max-concurrency: 1
        retry:
          enabled: true
  application:
    name: micro-order
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/product?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull&useSSL=false&allowPublicKeyRetrieval=true
    username: root
    password: root
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    serialization:
      write-dates-as-timestamps: false
mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
    auto-mapping-behavior: full
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  mapper-locations: classpath*:mapper/*Mapper.xml
  global-config:
    db-config:
      logic-not-delete-value: 1
      logic-delete-value: 0
```

### 3. RabbitMq队列定义

```java
//主要用于订单数据的投递，
public static final String ORDER_QUEUE_NAME = "order_queue";
public static final String ORDER_EXCHANGE_NAME = "order_exchange";
public static final String ORDER_ROUTINGKEY = "order.create";

//主要用于通知库存服务进行库存扣减
public static final String PRODUCT_QUEUE_NAME = "product_queue";
public static final String PRODUCT_EXCHANGE_NAME = "product_exchange";
public static final String PRODUCT_ROUTINGKEY = "product.create";

//主要用于库存扣减成功后，将数据返回给订单服务
public static final String ORDER_PRODUCT_QUEUE_NAME = "order_product_queue"; 
public static final String ORDER_PRODUCT_EXCHANGE_NAME = "order_product_exchange";
public static final String ORDER_PRODUCT_ROUTINGKEY = "order.product";

//死信队列，长时间未被消费，就投递到死信队列当中
public static final String DLX_EXCHANGE = "dlx_exchange";
public static final String DLX_QUEUE = "dlx_queue";

//延时队列，超时没有支付的订单，就执行当前操作
public static final String DELAY_ORDER_EXCHANGE = "delay_order_exchange";
public static final String DELAY_ORDER_QUEUE = "delay_order_queue";
public static final String DELAY_ORDER_ROUTINGKEY = "delay.order.*";
```

#### 注意点

​	中间件服务用于管理所有的队列和交换机，需要通过 RabbitTemplate 手动触发一次之后，所有的队列才会创建，否则，队列是不会创建的，只有在第一次用到之后才会创建

```java
rabbitTemplate.convertAndSend("order_exchange", "order_create", "hello");
```

### 4. 服务设计

#### 4.1 公共实体设计

```
@Data
@ApiModel("创建订单实体类")
public class OrderDTO {
    /**
     * 商品id
     */
    private Long productId;

    /**
     * 用户id
     */
    private Long userId;

    /**
     * 订单金额
     */
    private BigDecimal orderMoney;

    /**
     * 购买数量
     */
    private Long num;
}
```



#### 4.1 micro-middleware

##### 4.1.1 队列配置对象

```java
@Configuration
public class OrderConfiguration {

    public static final String ORDER_QUEUE_NAME = "order_queue";

    public static final String ORDER_EXCHANGE_NAME = "order_exchange";

    public static final String ORDER_ROUTINGKEY = "order.create";

    @Bean
    public Queue orderQueue() {
        //绑定死信队列，订单超时未被消费，则进入死信队列
        return QueueBuilder.durable(ORDER_QUEUE_NAME).withArgument("x-dead-letter-exchange", DLX_EXCHANGE).build();
    }


    @Bean
    public DirectExchange orderExchange() {
        return new DirectExchange(ORDER_EXCHANGE_NAME);
    }

    /**
     * 创建交换机进行绑定
     */
    @Bean
    public Binding bindingOrder() {
        return BindingBuilder.bind(orderQueue()).to(orderExchange()).with(ORDER_ROUTINGKEY);
    }


    /**
     * -------------------------------------------------------------- 创建库存扣减成功后订单状态修改队列
     * @return
     */

    public static final String ORDER_PRODUCT_QUEUE_NAME = "order_product_queue";

    public static final String ORDER_PRODUCT_EXCHANGE_NAME = "order_product_exchange";

    public static final String ORDER_PRODUCT_ROUTINGKEY = "order.product";

    @Bean
    public Queue orderProductQueue() {
        //绑定死信队列，订单超时未被消费，则进入死信队列
        return QueueBuilder.durable(ORDER_PRODUCT_QUEUE_NAME).withArgument("x-dead-letter-exchange", DLX_EXCHANGE).build();
    }


    @Bean
    public DirectExchange orderProductExchange() {
        return new DirectExchange(ORDER_PRODUCT_EXCHANGE_NAME);
    }

    /**
     * 创建交换机进行绑定
     */
    @Bean
    public Binding bindingProductOrder() {
        return BindingBuilder.bind(orderProductQueue()).to(orderProductExchange()).with(ORDER_PRODUCT_ROUTINGKEY);
    }


    /**
     * -------------------------------------------------------------- 创建商品库存队列
     * @return
     */
    public static final String PRODUCT_QUEUE_NAME = "product_queue";

    public static final String PRODUCT_EXCHANGE_NAME = "product_exchange";

    public static final String PRODUCT_ROUTINGKEY = "product.create";

    @Bean
    public Queue productQueue() {
        return QueueBuilder.durable(PRODUCT_QUEUE_NAME).build();
    }


    @Bean
    public DirectExchange productExchange() {
        return new DirectExchange(PRODUCT_EXCHANGE_NAME);
    }

    /**
     * 创建交换机进行绑定
     */
    @Bean
    public Binding bindingProduct() {
        return BindingBuilder.bind(productQueue()).to(productExchange()).with(PRODUCT_ROUTINGKEY);
    }


    /**
     * -------------------------------------------------------------- 创建死信队列
     * @return
     */
    public static final String DLX_EXCHANGE = "dlx_exchange";

    public static final String DLX_QUEUE = "dlx_queue";

    @Bean
    public Queue dlxQueue() {
        return QueueBuilder.durable(DLX_QUEUE).build();
    }

    @Bean
    public TopicExchange dlxExchange() {
        return new TopicExchange(DLX_EXCHANGE);
    }

    @Bean
    public Binding bindingToDlx() {
        return BindingBuilder.bind(dlxQueue()).to(dlxExchange()).with("dlx.order.*");
    }


    /**
     * -------------------------------------------------------------- 创建延时队列：主要用于订单超时未支付的情况
     * @return
     */

    public static final String DELAY_ORDER_EXCHANGE = "delay_order_exchange";

    public static final String DELAY_ORDER_QUEUE = "delay_order_queue";

    public static final String DELAY_ORDER_ROUTINGKEY = "delay.order.*";

    @Bean
    public Exchange delayExchange() {
        Map<String, Object> map = new HashMap<>();
        //指定路由类型为直接交换机
        map.put("x-delayed-type", "topic");
        // 第二个交换机的类型自定义设置为 x-delayed-message 延迟队列指定
        return new CustomExchange(DELAY_ORDER_EXCHANGE, "x-delayed-message", true, false, map);
    }

    @Bean
    public Queue delayQueue() {
        return QueueBuilder
            .durable(DELAY_ORDER_QUEUE)
            .build();
    }

    @Bean
    public Binding bindingToDelay() {
        return BindingBuilder.bind(delayQueue()).to(delayExchange()).with(DELAY_ORDER_ROUTINGKEY).noargs();
    }

}
```

##### 4.1.2 接口设计

主要用于手动出发mq创建队列，可以改为启动服务时进行触发创建

```java
@GetMapping("/push")
public boolean push() {
    rabbitTemplate.convertAndSend("order_exchange", "order_create", "hello");
    return true;
}
```

#### 4.2 micro-order

##### 4.2.1 接口设计

```java
@RestController
@RequestMapping("order")
public class OrderController {
    /**
     * 服务对象
     */
    @Resource
    private OrderService orderService;
	
    /**
     * 创建订单后，返回订单的数据
     */
    @PostMapping("/create")
    public String createOrder(@RequestBody OrderDTO orderDTO) {
        return this.orderService.createOrder(orderDTO);
    }
    
	/**
     * 支付订单
     */
    @GetMapping("/pay/{orderId}")
    public boolean payOrder(@PathVariable("orderId") String orderId) {
        return this.orderService.payOrder(orderId);
    }

}
```

##### 4.2.2 实体设计

```
@Data
@ToString
public class Order {
    /**
     * 主键id
     */
    private Long id;

    /**
     * 订单id
     */
    private String orderId;

    /**
     * 商品id
     */
    private Long productId;

    /**
     * 用户id
     */
    private Long userId;

    /**
     * 订单金额
     */
    private BigDecimal orderMoney;

    /**
     * 购买数量
     */
    private Long num;

    /**
     * 订单状态:0 初始化，1 已下单，2 处理中，3 已出库，4 已完成
     */
    private Byte orderStatus;

    /**
     * 创建时间
     */
    private Date createdTime;

	/**
     * 更新时间
     */
    private Date updateTime;
}
```

##### 4.2.3 消费者设计

```java
@Component
public class OrderConsumer {

    @Resource
    private OrderService orderService;

    @RabbitHandler
    @RabbitListener(queues = "delay_order_queue")
    public void process(Message message, Channel channel) {
        try {
            String orderId = new String(message.getBody());
            System.out.println("接收到订单检查订单状态：" + orderId);
            orderService.checkOrder(orderId);
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), true);
        } catch (IOException e) {
            System.out.println("数据处理失败");
        }
    }

    @RabbitHandler
    @RabbitListener(queues = "order_product_queue")
    public void processOrderProduct(Message message, Channel channel) {
        try {
            Order o = JSON.parseObject(message.getBody(), Order.class);
            System.out.println("接收到库存扣减信息：" + o);
            orderService.updateOrderStatus(o.getOrderId(), Integer.valueOf(o.getOrderStatus()));
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), true);
        } catch (IOException e) {
            System.out.println("数据处理失败");
        }
    }


    /**
     * 死信队列，监听到订单没有消费，直接修改为已经超时
     * @param message
     * @param channel
     */
    @RabbitHandler
    @RabbitListener(queues = "dlx_queue")
    public void dlxProcess(Message message, Channel channel) {
        try {
            Order o = (Order) JSON.parseObject(message.getBody(), Order.class);
            System.out.println("订单超时：" + o.getOrderId());
            this.orderService.updateOrderStatus(o.getOrderId(), 4);
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), true);
        } catch (IOException e) {
            System.out.println("数据处理失败");
        }
    }

}
```

##### 4.2.4 Service设计

```java
@Service
public class OrderServiceImpl implements OrderService {

    @Resource
    private OrderMapper orderMapper;

    @Resource
    private RabbitTemplate rabbitTemplate;

    /**
     * 创建订单
     */
    @Override
    public String createOrder(OrderDTO orderDTO) {
        //1. 创建订单实体类，然后设置默认值
        Order order = new Order();
        order.setOrderId(UUID.randomUUID().toString().substring(0, 6));
        order.setUpdateTime(new Date());
        order.setCreatedTime(new Date());
        //初始化商品状态
        order.setOrderStatus((byte) 0);
        BeanUtils.copyProperties(orderDTO, order);
        int insert = orderMapper.insert(order);
        //订单插入成功后，发布延时队列信息，用于检测订单是否已经支付
        if (insert > 0) {
            this.rabbitTemplate.convertAndSend("delay_order_exchange", "delay.order.1", order.getOrderId(), p -> {
                //进行消息拦截处理，给消息设置延时时间
                p.getMessageProperties().setDelay(50000);
                return p;
            });
        }
        return order.getOrderId();
    }

    /**
     * 修改订单状态
     * @param orderId
     * @return
     */
    @Override
    public boolean updateOrderStatus(String orderId, Integer status) {
        return this.orderMapper.updateStatus(orderId, status);
    }

    @Override
    public boolean checkOrder(String orderId) {
        Order order = this.orderMapper.selectOneByOrderId(orderId);
        Byte orderStatus = order.getOrderStatus();
        //订单状态小于1，证明还没有支付，将订单状态修改为 已超时
        if (orderStatus < 1) {
            this.updateOrderStatus(orderId, 4);
        }
        return true;
    }

    @Override
    public boolean payOrder(String orderId) {
        //1. 支付订单，修改订单后，发布库存扣除
        UpdateWrapper<Order> updateWrapper = new UpdateWrapper<>();
        updateWrapper.eq("order_id", orderId);
        boolean b = this.updateOrderStatus(orderId, 1);
        if (b) {
            //查询出订单的信息，将订单信息发送到mq中进行库存扣减
            Order order = this.orderMapper.selectOneByOrderId(orderId);
            MessageProperties properties = MessagePropertiesBuilder
                .newInstance()
                .build();
            //将订单的实体进行序列化
            Message message = rabbitTemplate.getMessageConverter().toMessage(JSON.toJSONString(order), properties);
            //发送到订单队列里
            rabbitTemplate.send("product_exchange", "product.create", message);
        }
        return b;
    }
}
```

##### 4.2.5 Mapper设计

```java
public interface OrderMapper extends BaseMapper<Order> {
    int insert(Order record);

    Order selectOneByOrderId(String orderId);

    boolean updateStatus(@Param("orderId") String orderId, @Param("status") Integer status);
}
```

#### 4.3 micro-product

##### 4.3.1 模型设计

```java
@Data
@ApiModel("商品模型")
public class ProductDTO {
    /**
     * 商品名称
     */
    private String productName;

    /**
     * 商品id
     */
    private Long productId;

    /**
     * 商品价格
     */
    private Double productMoney;

    /**
     * 库存数量
     */
    private Long storageNum;
}
```

##### 4.3.2 接口设计

```java
@RestController
@RequestMapping("/product")
public class ProductController {

    @Resource
    private ProductService productService;

    /**
     * 添加商品信息
     * @return
     */
    @PostMapping("/insert")
    public boolean insert(@RequestBody ProductDTO productDTO) {
        return productService.insert(productDTO) > 0;
    }

}
```

##### 4.3.3 Service设计

```java
@Service
public class ProductServiceImpl implements ProductService {

    @Resource
    private ProductMapper productMapper;

    @Resource
    private ProductStorageMapper productStorageMapper;

    @Override
    @Transactional
    public int insert(ProductDTO productDTO) {
        Product product = new Product();
        BeanUtils.copyProperties(productDTO, product);
        //添加商品到库中
        int insert = productMapper.insert(product);
        if (insert > 0) {
            //商品添加成功后，设置库存
            ProductStorage productStorage = new ProductStorage();
            productStorage.setProductId(product.getProductId());
            productStorage.setProductStorageFreeze(0L);
            productStorage.setProductStorage(productDTO.getStorageNum());
            this.productStorageMapper.insert(productStorage);
        }
        return insert;
    }

}
```

##### 4.3.4 消费者设计

```java
@Component
public class ProductConsumer {

    @Resource
    private ProductStorageService productStorageService;

    @RabbitHandler
    @RabbitListener(queues = "product_queue")
    public void process(Message message, Channel channel) {
        try {
            Order order = JSON.parseObject(message.getBody(), Order.class);
            //处理库存
            this.productStorageService.processOrder(order);
            //手动返回ack
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), true);
        } catch (IOException e) {
            System.out.println("订单处理失败");
        }
    }

}
```

### 5. sql设计

#### 5.1 订单表

```sql
create table `order`
(
    id           bigint auto_increment comment '主键id'
        primary key,
    order_id     varchar(32)       null comment '订单id',
    product_id   bigint            null comment '商品id',
    user_id      bigint            null comment '用户id',
    order_money  decimal(10, 5)    null comment '订单金额',
    order_status tinyint default 0 null comment '订单状态:0 初始化，1 已下单，2 处理中，3 已出库，4 已完成，5 已超时',
    created_time datetime          null comment '创建时间',
    update_time  datetime          null
)
    comment '商品订单表';
```

#### 5.2 商品表

```sql
create table product
(
    id            bigint auto_increment comment '主键id'
        primary key,
    product_name  varchar(255)  null comment '商品名称',
    product_id    bigint        null comment '商品id',
    product_money double(10, 5) null comment '商品价格'
)
    comment '商品表';
```

#### 5.3 商品库存表

```sql
create table product_storage
(
    id                     bigint auto_increment
        primary key,
    product_id             bigint           null comment '商品id',
    product_storage        bigint           null comment '商品库存',
    product_storage_freeze bigint default 0 null comment '冻结库存数量'
)
    comment '商品库存表';
```

## 12. 分布式事务（国内开源的 ByteTCC、Himly、TCC-transaction）

### 12. 分布式涉及的异常问题
```java
 机器宕机、网络异常、消息丢失、消息乱序、数据错误、不可靠的TCP、存储数据丢失、其他异常等等
```
### 12.2 CAP定理
```java
这三个指标不可能同时做到。这个结论就叫做 CAP 定理。
C:一 致 性（多个A子系统中修改了值，B系统中也能访问到修改的值，就叫数据一致性）
A:可 用 性（只要用户发送请求，服务器就必须给出回应）
P:分区容错（分布式系统放在多个子网络中，每个子网络就叫一个区，可能因为各种原因导致通信失败）
一致性和可用性，为什么不可能同时成立？答案很简单，因为可能通信失败（即出现分区容错）

分区容错发生在分布式系统内部互访通信，是指分布式网络中部分网络不可用，但系统依然正常对外提供服务。
比如：北京的订单系统，访问上海的库存系统，可能导致失败。如果发生失败，就要在A和C之间做出选择。
要么停止系统进行错误恢复，要么继续服务但是降低一致性，所以说只能保证AP或CP。

2PC跨库事务（JTA/XA）
开源框架（atomikos）实现垮库事务
1，导入jar包 
2，连接池基本属性
3，连接路径账号密码
4，让atomikos代理Jdbc 
5，创建一个全场事务
6，开启事务，编译sql
7，预提交
8，判断 如果 没有异常 就提交事务，则回滚事务，同时atomikos 如果提交 失败的话 它有重试的 还有记录日志的 功能 可以手动 提交(在实际开发中用的并不多)
```
### 12.3 BASE理论
```java
Basically Available（基本可用）
Soft state（软状态）
Eventually consistent（最终一致性）
BASE理论是对CAP中的一致性和可用性进行一个权衡的结果，理论的核心思想就是：我们无法做到强一致，但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性（Eventual consistency）
```
### 12.4 基于可靠消息的最终一致性方案（异步确保型）
```java
1、上游服务，发送消息到可靠消息服务，可靠消息服务保存消息状态为“待确认”
2、上游服务执行自己本地数据库的操作，事务成功修改可靠消息服务中的数据，事务失败找到可靠消息服务的数据删除
3、如果此时消息服务中状态为“已发送”，就将消息发送给MQ（修改和发送必须在一个方法里面，并且开启本地事务）
4、下游服务等着从MQ消费就好，如果消费成功就返回来通知消息服务，并且将消息的状态修改为“已完成”

如何保证上游服务对消息的100%可靠投递
新开一个定时后台线程，对可靠消息服务中的数据不停的进行监控，如果有消息一直为“待确认”，那么就去调用上游服务，去确认这条数据有没有执行成功。

如何保证下游服务对消息的100%可靠接收
在可靠服务中新开一个定时后台线程，对可靠服务中的数据不停的进行监控，如果有消息为“已发送”始终没有变成“已完成”，那么就会认为是出现了问题，就去调用下游服务，来查看下游的服务是否消费成功

```
### 12.5 TCC事务补偿性方案（基于2PC协议的两阶段提交）  同步
```java
核心思想是：针对每个操作，都要注册一个与其对应的确认和补偿（撤销）操作
Try: 阶段主要是对业务系统做检测及资源预留
Confirm: 阶段主要是对业务系统做确认提交，Try阶段执行成功并开始执行 Confirm阶段时，默认 Confirm阶段是不会
         出错的。即：只要Try成功，Confirm一定成功
Cancel: 阶段主要是在业务执行错误，需要回滚的状态下执行的业务取消，预留资源释放
每一个服务里面都有一个事务活动日志磁盘文件或者数据库，会保存下来分布式事务运行的各个阶段和状态。如果出现问题
或者出现了故障，都可以在这里面找到
```
### 12.6 最大努力通知型方案


### 12.7 基于 KV 存储的队列支持的高可用降级方案
```java
上游服务：
  自己封装一个MQ可靠消息服务，自己封装一个MQ客户端组件与故障感知，如果在消息进行投递的时候，多次没有得到回应，那么就启动降级。
解决：可以采用 Redis来进行存储，Redis也支持队列。均匀hash算法来均匀的存储在N个队列中
问题：
  不能对同一个key里面加入大量的value值
  不能往少数key对应的数据结构中写入数据，否则会导致热key产生
通过Zookeeper来触发一个降级开发，让整个MQ这块全部降级
下游服务：
  下游服务消费 MQ 也是通过自行封装的组件来做的，此时那个组件如果从 ZK 感知到降级开关打开了，首先会判断自己是  
  否还能继续从 MQ 消费到数据？如果不能了，就开启多个线程，并发的从 KV 存储的各个预设好的上百个队列中不断的获
  取数据。
自动恢复：
  如果降级开关打开之后，自行封装的组件需要开启一个线程，每隔一段时间尝试给 MQ 投递一个消息看看是否恢复了。
如果 MQ 已经恢复可以正常投递消息了，此时就可以通过 ZK 关闭降级开关，然后可靠消息服务继续投递消息到 MQ，下游服务在确认 KV 存储的各个队列中已经没有数据之后，就可以重新切换为从 MQ 消费消息。
```
