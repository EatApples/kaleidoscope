## RabbitMQ 使用

&#8195;&#8195;RabbitMQ 是实现 AMQP（即 Advanced Message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计）的消息中间件。RabbitMQ 主要是为了实现系统之间的双向解耦（异步）。

&#8195;&#8195;AMQP描述了一套模块化的组件以及这些组件之间进行连接的标准规则。在服务器中，三个主要功能模块连接成一个处理链完成预期的功能：

（1）exchange：交换机，接收发送端（生产者）发送的消息，并根据一定的规则将这些消息路由到消息队列；

（2）queue，消息队列，存储消息，直到这些消息被接收端（消费者）安全处理完为止；

（3）binding，定义了 exchange 和 queue 之间的关联，提供路由规则。

&#8195;&#8195;通常我们谈到队列服务， 会涉及三个概念： 发送端、消息队列和接收端。RabbitMQ 在这个基本概念之上，多做了一层抽象：在发送端和消息队列之间，加入了交换机 （Exchange）。通过设置交换机到消息队列的不同路由规则，除了可以实现点对点模式之外，还能实现发布订阅模式。

### 一 准备工作
#### 1.1 Maven 中引入相关依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```
#### 1.2 配置文件中添加 RabbitMQ 相关属性
```yml
spring.rabbitmq.host: 127.0.0.1
spring.rabbitmq.port: 5672
spring.rabbitmq.username: guest
spring.rabbitmq.password: guest
```

### 二 代码示例及说明

#### 2.1 消息队列的配置
&#8195;&#8195;为保证能正常使用 RabbitMQ 提供的队列服务，首先要进行消息队列的配置。消息队列的配置既可以在发送端配置，也可以在接收端配置，这里建议在接收端配置，由接收端创建队列（交换机）。

##### 2.1.1 队列的配置
&#8195;&#8195;这里在MQ服务器上配置了两个队列：p2p_queue_one 与 p2p_queue_two。这两个队列用于点对点模式。

```java

import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AmqpConfig {

    @Bean("p2p_queue_one")
    Queue p2p_queue_one() {

        return new Queue("p2p_queue_one", true); // 队列持久

    }

    @Bean("p2p_queue_two")
    Queue p2p_queue_two() {

        return new Queue("p2p_queue_two", true); // 队列持久

    }
}

```
##### 2.1.2 交换机的配置
&#8195;&#8195;如果要使用发布订阅模式，则需要配置交换机。

&#8195;&#8195;这里配置了两个交换机： pubsub_exchange_one 与 pubsub_exchange_two，四个队列：pubsub_queue_one，pubsub_queue_two，pubsub_queue_three 与 pubsub_queue_four。
其中：
+ 队列 pubsub_queue_one，pubsub_queue_two 与 交换机 pubsub_exchange_one 绑定，则所有发送到交换机 pubsub_exchange_one 上的消息，会全部路由到 pubsub_queue_one，pubsub_queue_two 这两个队列中；

+ 队列 pubsub_queue_three，pubsub_queue_four 与 交换机 pubsub_exchange_two 绑定，同理，所有发送到交换机 pubsub_exchange_two 上的消息，会全部路由到 pubsub_queue_three，pubsub_queue_four 这两个队列中！

注意：配置的交换机类型是 FanoutExchange；

注意：绑定操作 BindingBuilder.bind(pubsub_queue_one()).to(pubsub_exchange_one()); 是
bind(队列名).to(交换机名)

``` java
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.FanoutExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AmqpConfig {

    @Bean("pubsub_queue_one")
    Queue pubsub_queue_one() {

        return new Queue("pubsub_queue_one", true); // 队列持久

    }

    @Bean("pubsub_queue_two")
    Queue pubsub_queue_two() {

        return new Queue("pubsub_queue_two", true); // 队列持久

    }

    @Bean("pubsub_queue_three")
    Queue pubsub_queue_three() {

        return new Queue("pubsub_queue_three", true); // 队列持久

    }

    @Bean("pubsub_queue_four")
    Queue pubsub_queue_four() {

        return new Queue("pubsub_queue_four", true); // 队列持久

    }

    //FanoutExchange
    @Bean("pubsub_exchange_one")
    FanoutExchange pubsub_exchange_one() {

        return new FanoutExchange("pubsub_exchange_one");
    }

    @Bean("pubsub_exchange_two")
    FanoutExchange pubsub_exchange_two() {

        return new FanoutExchange("pubsub_exchange_two");
    }

    //bind(queueName).to(exchangeName)
    @Bean
    Binding binding1() {

        return BindingBuilder.bind(pubsub_queue_one()).to(pubsub_exchange_one());
    }

    @Bean
    Binding binding2() {

        return BindingBuilder.bind(pubsub_queue_two()).to(pubsub_exchange_one());
    }

    @Bean
    Binding binding3() {

        return BindingBuilder.bind(pubsub_queue_three()).to(pubsub_exchange_two());
    }

    @Bean
    Binding binding4() {

        return BindingBuilder.bind(pubsub_queue_four()).to(pubsub_exchange_two());
    }

}
```

#### 2.2 发送端
#### 2.2.1 发送端的封装类
&#8195;&#8195;这里通过 @Autowired 注解获得 RabbitTemplate 模板实例，然后封装对点对点模式与发布订阅模式的发送。

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class Producer {

    private static final Logger LOGGER = LoggerFactory.getLogger(Producer.class);
    /**
     * 获得RabbitMQ模版实例
     */
    @Autowired
    private RabbitTemplate instance;

    /**
     * 点对点模式
     *
     * @param queueName
     * @param message
     * @return
     */
    public boolean sendP2P(String queueName, Object message) {

        try {
            instance.convertAndSend("", queueName, message);
        }
        catch (Exception ex) {
            LOGGER.error("", ex);
            return false;
        }
        return true;

    }

    /**
     * 发布订阅模式
     *
     * @param exchangeName
     * @param message
     * @return
     */
    public boolean sendPubSub(String exchangeName, Object message) {

        try {
            instance.convertAndSend(exchangeName, "", message);
        }
        catch (Exception ex) {
            LOGGER.error("", ex);
            return false;
        }
        return true;

    }

}

```
#### 2.2.2 发送端的使用
&#8195;&#8195;在调用类中使用 @Autowired 注解，获得 Producer 对象，就能使用 Producer 封装的发送操作了。

```java
    @Autowired
    @Qualifier("Producer")
    public Producer producer;
```
&#8195;&#8195;例如：
```java
//点对点模式
producer.sendP2P("p2p_queue_one", "STRING");
//发布订阅模式
producer.sendPubSub("pubsub_exchange_one", "STRING");
```
#### 2.2 接收端
&#8195;&#8195;不管是点对点模式还是发布订阅模式，接收端只与队列有关，只从队列中接收消息。
在接收类上使用 @RabbitListener 注解，指定消费的队列名；然后在处理函数上使用 @RabbitHandler 注解，获取消息对象，进行处理。

注意：因为 Spring Boot RabbitMQ 对发送的消息做了封装（消息对象序列化后封装在 org.springframework.amqp.core.Message 对象中），则接收类需要反序列化才能获得消息对象。

注意：这里接收消息的处理函数入口参数必须是 Object 对象，后续再根据具体反序列化后的对象进行相应的处理，否则会报错！

```java
import java.io.UnsupportedEncodingException;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
@RabbitListener(queues = "p2p_queue_one")
public class Consumer {

    private static final Logger LOGGER = LoggerFactory.getLogger(Consumer.class);

    @RabbitHandler
    public void process(Object obj) throws UnsupportedEncodingException {

        if (obj instanceof Message) {
            // 获得消息对象序列化后的字节流，需要反序列化才能获得消息。如果消息是String类型，直接 new String(msg,"UTF-8");
            byte[] msg = ((Message) obj).getBody();
            // do something with your message

        }
        else {
            LOGGER.error("NEVER HAPPEN");
        }

    }

}
```
### 三 参考资料

#### 3.1 Spring Boot中使用RabbitMQ
http://blog.didispace.com/spring-boot-rabbitmq/

#### 3.2 RabbitMQ 官网
http://www.rabbitmq.com/

#### 3.3 Spring AMQP 官方文档
https://docs.spring.io/spring-amqp/docs/latest-ga/reference/html/_reference.html
