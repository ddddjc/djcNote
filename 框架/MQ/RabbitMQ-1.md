# 1. 消息队列

**消息**：是指在应用之间传送数据，消息可以非常简单，比如只包含文本字符串，也可以很复杂

**队列**：可以说是一个数据结构，可以存储数据，先进先出

**消息队列**：是一种应用间的通信方式，消息发送后可以立即返回，有消息系统来确保信息的可靠传递，消息发送者只管把消息发布到MQ而不管谁来取，小婆媳使用者只管从MQ中取消息儿不管谁发布的，这样发布者和使用者都不用知道对方的存在

消息队列主要解决了一下问题：

- **应用耦合**：多应用通过消息队列对统一消息进行处理，避免调用接口失败导致整个过程失败
- **异步处理**：多应用对消息队列中同一消息进行处理，沿用件并发处理消息，相比串行处理，减少处理时间
- **限流削峰**：广泛应用与，秒杀或抢购活动中，避免流量过大导致应用系统挂掉的情况

## 1.1MQ的相关概念

### Producter 生产者

生产者创建消息，然后发布到RabbitMQ中。消息一般可以包含2个部分：`消息体`和`标签（Label）`。消息体也可以称之为payload，在实际应用中，消息体一般是一个带有业务逻辑结构的数据，比如一个JSON字符串。当然可以进一步对这消息进行序列化操作。消息的标签用来表述这条消息，比如一个交换器的名称和一个路由键

### Consumer 消费者

消费者连接到RabbitMQ服务器，并订阅到队列上。当消费者消费一条消息时，只是消费者消费的消息体（payload）。在消息路由的过程中，消息的标签会丢弃，存入到队列中的消息体只有消息体，也就算不知道消息的生产者是谁

### Broker： 消息中间件的服务节点

对于RabbitMQ来说，一个RabbitMQ Broker可以简单地看做一个RabbitMQ服务节点，或者RabbitMQ服务实例。大多数情况下也可以讲一个RabbitMQ Broker看作一台RabbitMQ服务器

### Queue： 队列，是RabbitMQ的内部对象，用于存储消息

RabbitMQ中系哦啊西都只能存出在队列中，多个消费者可以订阅同一个队列，这时队列中的消息会被平均分摊（Round-Robin，即轮训）给多个消费者进行处理，而不是每个消费者都收到所有消息并处理，如下图

![image _27_.png](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307241006682.png)

### Exchange： 交换机

一方面接收生产者发送的消息。另一方面，找到如何处理消息，例如递交给某个特别队列、递交给所有队列、或者将消息丢弃。到底如何操作，取决于交换机的类型，有以下常见三种类型：

- Fanout：广播，将消息交给所有绑定到交换机的队列
- Direct：定向，把消息交给符合指定routing key的队列
- Topic：通配符，把消息交给复合routing pattern（路由模式）的队列

`Exchange(交换关机)`只负责转发消息，不具备存储消息的能力，因此，如果没有任何消息队列与Exchange绑定，或者没有符合路由规则的队列，那么消息将会丢失

## 2.构建

依赖

```xml
    <dependencies>
<!--        rabbitmq java 客户端-->
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.6.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

##   工作模式

### Routing模式

- 队列与交换机的绑定，不能是任意绑定了，而是要指定一个RoutingKey （路由key）
- 消息的发生方在向Exchange发送消息时，也必须指定消息的RoutingKey
- Exchange不再把消息交给每一个绑定的队列，而是根据消息的Routing Key进行判断，只有队列的RoctingKey与消息的RoutingKey 完全一致，才会接收到消息

### topics 通配符模式

![image-20230724150625516](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307241506594.png)

## 3. RabbitMQ高级特性

### 3.1消息的可靠投递

在使用RabbitMQ的时候，作为消息发送方希望杜绝任何消息丢失或者投递失败场景。RabbitMQ为我们提供了两种方式来控制消息投递的可靠性模式。

- cinfurm 确认模式 
  -  `publisher-confirm-type:参数`，开启确认模式
  - 使用rabbitTemplate.setConfirmCallback设置回调函数。当消息发达到exchange后回调confirm方法。在方法中判断ack，如果为true，则发送成功，如果为false，则发送失败，需要处理。
- returen 退回模式 
  - `publisher-returns: true`，开启回退模式
  - 使用rabbitTemplate.setReturnCallback甚至回退函数，当消息从exchange路由到queue失败后，如果设置了rabbitTemplate.setMandatory(true)参数，则会将消息退回给producer。并执行会调函数returnedMessage。

RabbitMQ整个消息投递的路径为：

producer-->rabbitmq broker--->exchange--->queue--->consumer 

- 消息从producer到exchange则会返回一个confirmCallback。
- 消息从exchange到queue投递失败则会返回一个return Callback.

我们将利用这两个callback 控制消息的可靠性投递

###  3.2 Consumer Ack

ack指Acknowledge，确认。表示消费端收到消息后的确认方式。

有三种确认方式

- 手动确认：acknowledge=“none”
- 自动确认：acknowledge=“manual”
- 根据异常情况确认：acknowledge=“auto”

其中自动确认指当消息一旦被Consumer接收到，则自动确认收到，并将相应message从RabbitMQ的消息缓存中移除。但是在实际业务处理中，很可能消息接收到，业务处理出现异常，那么该消息就会缺失。如果设置了手动确认的方式，则需要在业务处理成功后，调用cannel。basicAack（），手动签收，如果出现异常，则调用cannel.basicNack()方法，让其自动重新发送消息。

### 3.3 消费端限流

当请求特别多的时候，设置限流，每秒从MQ中拉取暂定数量的请求

- 消费端的确认模式为手动确认，acknowledge=“manual”
- 配置prefetch属性为消费端一次性拉取多少消息

### 3.4 TTL

概念：

- TTL 全称Time To Live（存活时间/过期时间)
- 当消息达到存活时间后，还没被消费，会被自动清除
- RabbitMQ可以对消息设置过期时间，也可以对整个队列设置过期时间
- 消息过期后，只有在最顶端时才会判断是否过期以及删除，这样可以提高效率，不用一直扫描每个元素（懒惰）

使用：

- 设置队列过期时间使用参数：x-message-ttl,单位：ms（毫秒），会对整个队列消息统一过期
- 设置消息过期时间使用从参数：expiration。单位：ms（毫秒），当该消息在队列头部时（消费时），会单独判断这一消息是否过期。
- 如果两者都进行了设置，咦时间短的为准。

### 3.5 死信队列

死信队列，DLX，Dead Letter Exchange（死信交换机），当 消息称为dead message后，可以被重新发送到另一个交换机，这个交换机就是DLX。

- 消息成为死信的情况：

  1. 消息长度达到限制

  1. 消费者拒绝消费信息，basicNack/basicReject，并不把消息重新放入目标队列，requeue=false 

  1. 原队列存在消息过期设置，消息达超时时间未被消费

- 当消息称为死信后，如果该队列绑定了死信交换机，则消息会被死信交换机重新路由到死信队列

### 3.6 延迟队列

消息进入队列后，不会立即被消费，只有达到指定的时间后，才会被消费。

需求：

1. 下单后，三十分钟为支付，取消订单，回滚库存
2. 新用户注册成功七天后，发短信问候

实现方式：

1. 定时器：对数据库消耗过高
2. 延迟队列

![image-20230725094128889](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307250941973.png)

RabbitMQ并没有提供延迟队列的功能

但可以使用TTL+死信队列组合实现延迟队列的效果

### 3.7日志与监控

![image-20230725094956524](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307250949600.png)

### 3.8

## 4. RabbitMQ应用问题

### 4.1 消息可靠性保障

### 4.2 消息幂等性

### 5. RabbitMQ 集群搭建

