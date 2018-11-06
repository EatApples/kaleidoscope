在介绍 ActiveMQ 之前，首先简要介绍一下 JMS 规范。

ActiveMQ 是 JMS 的标准（默认？）实现。

### 1. JMS的基本构件
#### 1.1 连接工厂
接工厂是客户用来创建连接的对象，例如 ActiveMQ 提供的 ActiveMQConnectionFactory。

#### 1.2 连接
JMS Connection 封装了客户与 JMS 提供者之间的一个虚拟的连接。

#### 1.3 会话
JMS Session 是生产和消费消息的一个单线程上下文。会话用于创建消息生产者（producer）、消息消费者（consumer）和消息（message）等。会话提供了一个事务性的上下文，在这个上下文中，一组发送和接收被组合到了一个原子操作中。

#### 1.4 目的地
目的地是客户用来指定它生产的消息的目标和它消费的消息的来源的对象。JMS1.0.2 规范中定义了两种消息传递域：点对点（PTP）消息传递域和发布/订阅消息传递域。

点对点消息传递域的特点如下：

+ 每个消息只能有一个消费者。

+ 消息的生产者和消费者之间没有时间上的相关性。无论消费者在生产者发送消息的时候是否处于运行状态，它都可以提取消息。

发布/订阅消息传递域的特点如下：

+ 每个消息可以有多个消费者。

+ 生产者和消费者之间有时间上的相关性。订阅一个主题的消费者只能消费自它订阅之后发布的消息。JMS规范允许客户创建持久订阅，这在一定程度上放松了时间上的相关性要求。持久订阅允许消费者消费它在未处于激活状态时发送的消息。

在点对点消息传递域中，目的地被成为队列（queue）；在发布/订阅消息传递域中，目的地被成为主题（topic）。

#### 1.5 消息生产者
消息生产者是由会话创建的一个对象，用于把消息发送到一个目的地。

#### 1.6 消息消费者
消息消费者是由会话创建的一个对象，它用于接收发送到目的地的消息。消息的消费可以采用以下两种方法之一：

+ 同步消费。通过调用消费者的receive方法从目的地中显式提取消息。receive方法可以一直阻塞到消息到达。

+ 异步消费。客户可以为消费者注册一个消息监听器，以定义在消息到达时所采取的动作。

#### 1.7 消息
JMS消息由以下三部分组成：

+ 消息头。每个消息头字段都有相应的getter和setter方法。

+ 消息属性。如果需要除消息头字段以外的值，那么可以使用消息属性。

+ 消息体。JMS定义的消息类型有TextMessage、MapMessage、BytesMessage、StreamMessage和ObjectMessage。

### 2. JMS 的可靠性机制

#### 2.1 确认
JMS消息只有在被确认之后，才认为已经被成功地消费了。消息的成功消费通常包含三个阶段：客户接收消息、客户处理消息和消息被确认。

在事务性会话中，当一个事务被提交的时候，确认自动发生。在非事务性会话中，消息何时被确认取决于创建会话时的应答模式（acknowledgement mode）。该参数有以下三个可选值：

+ Session.AUTO_ACKNOWLEDGE。当客户成功的从receive方法返回的时候，或者从MessageListener.onMessage方法成功返回的时候，会话自动确认客户收到的消息。

+ Session.CLIENT_ACKNOWLEDGE。 客户通过消息的acknowledge方法确认消息。需要注意的是，在这种模式中，确认是在会话层上进行：确认一个被消费的消息将自动确认所有已被会话消费的消息。例如，如果一个消息消费者消费了10个消息，然后确认第5个消息，那么所有10个消息都被确认。

+ Session.DUPS_ACKNOWLEDGE。 该选择只是会话迟钝地确认消息的提交。如果 JMS  provider 失败，那么可能会导致一些重复的消息。如果是重复的消息，那么 JMS provider必须把消息头的 JMSRedelivered 字段设置为 true。

#### 2.2 持久性
JMS 支持以下两种消息提交模式：

+ PERSISTENT。指示 JMS provider 持久保存消息，以保证消息不会因为 JMS provider 的失败而丢失。

+ NON_PERSISTENT。不要求 JMS provider 持久保存消息。

#### 2.3 优先级
可以使用消息优先级来指示 JMS provider 首先提交紧急的消息。优先级分10个级别，从0（最低）到9（最高）。如果不指定优先级，默认级别是4。需要注意的是，JMS provider 并不一定保证按照优先级的顺序提交消息。

#### 2.4 消息过期
可以设置消息在一定时间后过期，默认是永不过期。

#### 2.5 临时目的地
可以通过会话上的 createTemporaryQueue 方法和 createTemporaryTopic 方法来创建临时目的地。它们的存在时间只限于创建它们的连接所保持的时间。只有创建该临时目的地的连接上的消息消费者才能够从临时目的地中提取消息。

#### 2.6 持久订阅
首先消息生产者必须使用 PERSISTENT 提交消息。客户可以通过会话上的 createDurableSubscriber 方法来创建一个持久订阅，该方法的第一个参数必须是一个 topic。第二个参数是订阅的名称。

JMS provider 会存储发布到持久订阅对应的 topic 上的消息。如果最初创建持久订阅的客户或者任何其它客户使用相同的连接工厂和连接的客户ID、相同的主题和相同的订阅名再次调用会话上的 createDurableSubscriber 方法，那么该持久订阅就会被激活。JMS provider 会象客户发送客户处于非激活状态时所发布的消息。

持久订阅在某个时刻只能有一个激活的订阅者。持久订阅在创建之后会一直保留，直到应用程序调用会话上的 unsubscribe 方法。

#### 2.7 本地事务
在一个JMS客户端，可以使用本地事务来组合消息的发送和接收。JMS Session 接口提供了 commit 和 rollback 方法。事务提交意味着生产的所有消息被发送，消费的所有消息被确认；事务回滚意味着生产的所有消息被销毁，消费的所有消息被恢复并重新提交，除非它们已经过期。

事务性的会话总是牵涉到事务处理中，commit 或 rollback 方法一旦被调用，一个事务就结束了，而另一个事务被开始。关闭事务性会话将回滚其中的事务。

需要注意的是，如果使用请求/回复机制，即发送一个消息，同时希望在同一个事务中等待接收该消息的回复，那么程序将被挂起，因为知道事务提交，发送操作才会真正执行。

需要注意的还有一个，消息的生产和消费不能包含在同一个事务中。

### 3. JMS 规范的变迁
JMS的最新版本的是1.1。它和同 1.0.2 版本之间最大的差别是，JMS1.1 通过统一的消息传递域简化了消息传递。这不仅简化了JMS API，也有利于开发人员灵活选择消息传递域，同时也有助于程序的重用和维护。

以下是不同消息传递域的相应接口：

| JMS公共           | 点对点域               | 发布/订阅域            |
| ----------------- | ---------------------- | ---------------------- |
| ConnectionFactory | QueueConnectionFactory | TopicConnectionFactory |
| Connection        | QueueConnection        | TopicConnection        |
| estination        | Queue                  | Topic                  |
| Session           | QueueSession           | TopicSession           |
| MessageProducer   | QueueSender            | TopicPublisher         |
| MessageConsumer   | QueueReceiver          | TopicSubscriber        |

#### 3.1.Sender.java
```java

import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.DeliveryMode;
import javax.jms.Destination;
import javax.jms.MessageProducer;
import javax.jms.Session;
import javax.jms.TextMessage;
import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;

public class Sender {
    private static final int SEND_NUMBER = 5;

    public static void main(String[] args) {
        // ConnectionFactory ：连接工厂，JMS 用它创建连接
        ConnectionFactory connectionFactory;
        // Connection ：JMS 客户端到JMS Provider 的连接
        Connection connection = null;
        // Session： 一个发送或接收消息的线程
        Session session;
        // Destination ：消息的目的地;消息发送给谁.
        Destination destination;
        // MessageProducer：消息发送者
        MessageProducer producer;
        // TextMessage message;
        // 构造ConnectionFactory实例对象，此处采用ActiveMq的实现jar
        connectionFactory = new ActiveMQConnectionFactory(
                ActiveMQConnection.DEFAULT_USER,
                ActiveMQConnection.DEFAULT_PASSWORD,
                "tcp://localhost:61616");
        try {
            // 构造从工厂得到连接对象
            connection = connectionFactory.createConnection();
            // 启动
            connection.start();
            // 获取操作连接
            session = connection.createSession(Boolean.TRUE,
                    Session.AUTO_ACKNOWLEDGE);

            destination = session.createQueue("FirstQueue");
            // 得到消息生成者【发送者】
            producer = session.createProducer(destination);
            // 设置不持久化，此处学习，实际根据项目决定
            producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
            // 构造消息，此处写死，项目就是参数，或者方法获取
            sendMessage(session, producer);
            session.commit();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (null != connection)
                    connection.close();
            } catch (Throwable ignore) {
            }
        }
    }

    public static void sendMessage(Session session, MessageProducer producer)
            throws Exception {
        for (int i = 1; i <= SEND_NUMBER; i++) {
            TextMessage message = session
                    .createTextMessage("ActiveMq 发送的消息" + i);
            // 发送消息到目的地方
            System.out.println("发送消息：" + "ActiveMq 发送的消息" + i);
            producer.send(message);
        }
    }
}
```

#### 3.2.Receiver.java
```java
import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.Destination;
import javax.jms.MessageConsumer;
import javax.jms.Session;
import javax.jms.TextMessage;
import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;

public class Receiver {
    public static void main(String[] args) {
        // ConnectionFactory ：连接工厂，JMS 用它创建连接
        ConnectionFactory connectionFactory;
        // Connection ：JMS 客户端到JMS Provider 的连接
        Connection connection = null;
        // Session： 一个发送或接收消息的线程
        Session session;
        // Destination ：消息的目的地;消息发送给谁.
        Destination destination;
        // 消费者，消息接收者
        MessageConsumer consumer;
        connectionFactory = new ActiveMQConnectionFactory(
                ActiveMQConnection.DEFAULT_USER,
                ActiveMQConnection.DEFAULT_PASSWORD,
                "tcp://localhost:61616");
        try {
            // 构造从工厂得到连接对象
            connection = connectionFactory.createConnection();
            // 启动
            connection.start();
            // 获取操作连接
            session = connection.createSession(Boolean.FALSE,
                    Session.AUTO_ACKNOWLEDGE);

            destination = session.createQueue("FirstQueue");
            consumer = session.createConsumer(destination);
            while (true) {
                //设置接收者接收消息的时间，为了便于测试，这里定为100s
                TextMessage message = (TextMessage) consumer.receive(100000);
                if (null != message) {
                    System.out.println("收到消息" + message.getText());
                } else {
                    break;
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (null != connection)
                    connection.close();
            } catch (Throwable ignore) {
            }
        }
    }
}
```
