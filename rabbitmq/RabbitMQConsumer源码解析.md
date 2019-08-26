## 自底向上解析`RabbitMQ`消费端的源码

### 1. 版本说明

基于`RabbitMQ`客户端版本。

```xml
    <dependency>
			<groupId>com.rabbitmq</groupId>
			<artifactId>amqp-client</artifactId>
			<version>4.0.0</version>
		</dependency>
```

### 2. 消费端的基本使用

```java
		Consumer consumer = new DefaultConsumer(channel) {
			@Override
			public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,
					byte[] body) throws IOException {
            //body为字节流，即消息
            //TODO：业务处理
            //如果autoAck为false，则必须调用如下代码，回复ACK
            //envelope.getDeliveryTag()为本条消息的序号。回复ACK之后，服务器就不再保留这条信息
            //channel.basicAck(envelope.getDeliveryTag(), false);
			}

			@Override
			public void handleShutdownSignal(String consumerTag, ShutdownSignalException sig) {
          //连接断开做的处理
          //客户端有自动重连功能，这里啥都不做，只是记录这个事件
			}
		};
    //创建队列。这是幂等操作（如果已存在则不会再创建）
    channel.queueDeclare(queueName, true, false, false, null);
    //如果使用EXCHANGE，则需要创建交换机
    channel.exchangeDeclare(exchangeName, "fanout", true, false, null);
    //如果使用EXCHANGE，还需要将EXCHANGE与队列绑定
    channel.queueBind(queueName, exchangeName, "");
    //消费端流量控制，只有autoAck为false才有效
    channel.basicQos(prefetchCount);
    //将Consumer与队列绑定
    channel.basicConsume(queueName, autoAck, consumer);

```

下面将从`2`个方向向上溯源。

+ 其一，消费端`DefaultConsumer`中我们重写了`handleDelivery`，即消息处理流程，但除了上文的`Consumer`与`Channel`绑定之外，没有相关的调用逻辑（猜测与`Channel`有关）。所以，这条路就是找`handleDelivery`是怎么被调用的。

+ 其二，我们发现`Consumer`接口指定实例时（`DefaultConsumer`），需要传入一个`Channel`。当然这是为了流量控制时，能调用`channel.basicAck`所致。在`Consumer`初始化之后，需要与`Channel`绑定，即`channel.basicConsume(queueName, autoAck, consumer);`，这才是我们关心的。所以这条路是找`Channel`是怎么建立的。

以下先从`handleDelivery`的调用入手，再看`Channel`的建立。其实到最后，这两条线路在某处相遇了。

### 3. 消费端接口`Consumer`

消费端接口`Consumer`只有一个默认实现`DefaultConsumer`：

```java
public class DefaultConsumer implements Consumer
```

`DefaultConsumer`有一个扩展类`QueueingConsumer`：

```java
public class QueueingConsumer extends DefaultConsumer
```

`QueueingConsumer`给定了`Consumer`的实现，现在已不建议使用。

### 4. `ConsumerWorkService`

发现`Consumer`接口是一个回调接口（注意这句话，后文有解释它为什么是回调接口），内部由 `ConsumerWorkService` 设置的线程池调用。

```java
final public class ConsumerWorkService
```
线程池大小为可用处理器核数的2倍。

```java
    private static final int MAX_RUNNABLE_BLOCK_SIZE = 16;
    private static final int DEFAULT_NUM_THREADS = Runtime.getRuntime().availableProcessors() * 2;
    private final ExecutorService executor;
    private final boolean privateExecutor;
    private final WorkPool<Channel, Runnable> workPool;
    private final int shutdownTimeout;

    public ConsumerWorkService(ExecutorService executor, ThreadFactory threadFactory, int shutdownTimeout) {
        this.privateExecutor = (executor == null);
        this.executor = (executor == null) ? Executors.newFixedThreadPool(DEFAULT_NUM_THREADS, threadFactory)
                                           : executor;
        this.workPool = new WorkPool<Channel, Runnable>();
        this.shutdownTimeout = shutdownTimeout;
    }
```

回调接口：将`Consumer`放到线程池中执行。

```java
public void addWork(Channel channel, Runnable runnable) {
    if (this.workPool.addWorkItem(channel, runnable)) {
        this.executor.execute(new WorkPoolRunnable());
    }
}
```
这里没看见`Consumer`相关的调用啊，别急，往后看。

### 5. `ConsumerDispatcher`
回调接口将被`ConsumerDispatcher`调用。

```java
final class ConsumerDispatcher
```

`ConsumerDispatcher`获得`ConsumerWorkService`的引用。

```java

    private final ConsumerWorkService workService;

    private final AMQConnection connection;

    private final Channel channel;

    private volatile boolean shuttingDown = false;
    private volatile boolean shutdownConsumersDriven = false;
    private volatile CountDownLatch shutdownConsumersComplete;

    private volatile ShutdownSignalException shutdownSignal = null;

    public ConsumerDispatcher(AMQConnection connection,
                              Channel channel,
                              ConsumerWorkService workService) {
        this.connection = connection;
        this.channel = channel;
        workService.registerKey(channel);
        this.workService = workService;
    }
```

类`ConsumerDispatcher`的`handleDelivery`函数传入`Consumer`接口的实现（委托），封装成`Runnable`，随后调用`addWork`方法，放入线程池中执行。
```java
public void handleDelivery(final Consumer delegate,
                           final String consumerTag,
                           final Envelope envelope,
                           final AMQP.BasicProperties properties,
                           final byte[] body) throws IOException {
    executeUnlessShuttingDown(
    new Runnable() {
        @Override
        public void run() {
            try {
                delegate.handleDelivery(consumerTag,
                        envelope,
                        properties,
                        body);
            } catch (Throwable ex) {
                connection.getExceptionHandler().handleConsumerException(
                        channel,
                        ex,
                        delegate,
                        consumerTag,
                        "handleDelivery");
            }
        }
    });
}

private void executeUnlessShuttingDown(Runnable r) {
    if (!this.shuttingDown) execute(r);
}

private void execute(Runnable r) {
    checkShutdown();
    this.workService.addWork(this.channel, r);
}
```
`Consumer`的`handleDelivery`就在这里回调的。发现`handleDelivery`方法没，所以`Consumer`需要实现`handleDelivery`的处理逻辑。

### 6. `ChannelN`

`ChannelN`将会使用`ConsumerDispatcher`。
```java
public class ChannelN extends AMQChannel implements com.rabbitmq.client.Channel
```

`ChannelN`构造函数中新建了`ConsumerDispatcher`对象。
```java
/** Dispatcher of consumer work for this channel */
private final ConsumerDispatcher dispatcher;

/**
 * Construct a new channel on the given connection with the given
 * channel number. Usually not called directly - call
 * Connection.createChannel instead.
 * @see Connection#createChannel
 * @param connection The connection associated with this channel
 * @param channelNumber The channel number to be associated with this channel
 * @param workService service for managing this channel's consumer callbacks
 * @param metricsCollector service for managing metrics
 */
public ChannelN(AMQConnection connection, int channelNumber,
    ConsumerWorkService workService, MetricsCollector metricsCollector) {
    super(connection, channelNumber);
    this.dispatcher = new ConsumerDispatcher(connection, this, workService);
    this.metricsCollector = metricsCollector;
}

```
`ChannelN`的函数`processDelivery`将调用`dispatcher`的`handleDelivery`方法。

```java
protected void processDelivery(Command command, Basic.Deliver method) {
    Basic.Deliver m = method;

    Consumer callback = _consumers.get(m.getConsumerTag());
    if (callback == null) {
        if (defaultConsumer == null) {
            // No handler set. We should blow up as this message
            // needs acking, just dropping it is not enough. See bug
            // 22587 for discussion.
            throw new IllegalStateException("Unsolicited delivery -" +
                    " see Channel.setDefaultConsumer to handle this" +
                    " case.");
        }
        else {
            callback = defaultConsumer;
        }
    }

    Envelope envelope = new Envelope(m.getDeliveryTag(),
                                     m.getRedelivered(),
                                     m.getExchange(),
                                     m.getRoutingKey());
    try {
        // call metricsCollector before the dispatching (which is async anyway)
        // this way, the message is inside the stats before it is handled
        // in case a manual ack in the callback, the stats will be able to record the ack
        metricsCollector.consumedMessage(this, m.getDeliveryTag(), m.getConsumerTag());
        this.dispatcher.handleDelivery(callback,
                                       m.getConsumerTag(),
                                       envelope,
                                       (BasicProperties) command.getContentHeader(),
                                       command.getContentBody());
    } catch (Throwable ex) {
        getConnection().getExceptionHandler().handleConsumerException(this,
            ex,
            callback,
            m.getConsumerTag(),
            "handleDelivery");
    }
}
```
函数`processDelivery`的第一行`Consumer callback = _consumers.get(m.getConsumerTag());`
注意到`Consumer`的命名为`callback`（这就是回调函数）。

发现回调函数从`_consumers`获得的，这是一个`Map`。
```java
private final Map<String, Consumer> _consumers =
    Collections.synchronizedMap(new HashMap<String, Consumer>());
```
是谁把`Consumer`放入`_consumers`之中的？往下看。

原来是函数`basicConsume`将把回调接口`Consumer callback`记录下来（放入`_consumers`之中）。所以创建消费者之后，需要进行绑定操作，将消费者与`Channel`相关联：`channel.basicConsume(queueName, autoAck, consumer);`。
```java
/** Public API - {@inheritDoc} */
    @Override
    public String basicConsume(String queue, final boolean autoAck, String consumerTag,
                               boolean noLocal, boolean exclusive, Map<String, Object> arguments,
                               final Consumer callback)
        throws IOException
    {
        BlockingRpcContinuation<String> k = new BlockingRpcContinuation<String>() {
            @Override
            public String transformReply(AMQCommand replyCommand) {
                String actualConsumerTag = ((Basic.ConsumeOk) replyCommand.getMethod()).getConsumerTag();
                _consumers.put(actualConsumerTag, callback);

                // need to register consumer in stats before it actually starts consuming
                metricsCollector.basicConsume(ChannelN.this, actualConsumerTag, autoAck);

                dispatcher.handleConsumeOk(callback, actualConsumerTag);
                return actualConsumerTag;
            }
        };

        rpc(new Basic.Consume.Builder()
             .queue(queue)
             .consumerTag(consumerTag)
             .noLocal(noLocal)
             .noAck(autoAck)
             .exclusive(exclusive)
             .arguments(arguments)
            .build(),
            k);

        try {
            return k.getReply();
        } catch(ShutdownSignalException ex) {
            throw wrap(ex);
        }
    }
```


#### 6.1  `ChannelN`分支
`ChannelN`继承了`AMQChannel`,实现了`Channel`接口。`Channel`就是类`ChannelN`生成的。从这里开始，我们开始`Channel`的向上溯源。
```java
public class ChannelN extends AMQChannel implements com.rabbitmq.client.Channel
```
下面继续跟踪`ChannelN`的函数`processDelivery`将被谁调用。

```java
protected void processDelivery(Command command, Basic.Deliver method)
                              ↓
public boolean processAsync(Command command) throws IOException
                              ↓
public void handleCompleteInboundCommand(AMQCommand command) throws IOException//父类AMQChannel的方法
                              ↓
public void handleFrame(Frame frame) throws IOException//父类AMQChannel的方法
```
跟踪调用链，发现将由`AMQChannel`的`handleFrame`方法的调用。谁会调用`handleFrame`呢？
是类`AMQConnection`!

这里暂且将`handleDelivery`的调用放下，等遇到`AMQConnection`再说。  
#### 6.2 `RecoveryAwareChannelN`分支
`RecoveryAwareChannelN`继承了`ChannelN`，在连接类型是可恢复的时候使用`RecoveryAwareChannelN`。
```java
public class RecoveryAwareChannelN extends ChannelN   
```  

`RecoveryAwareChannelN`还是通过父类`ChannelN`来调用`processDelivery`。
```java
@Override
protected void processDelivery(Command command, AMQImpl.Basic.Deliver method) {
    long tag = method.getDeliveryTag();
    if(tag > maxSeenDeliveryTag) {
        maxSeenDeliveryTag = tag;
    }
    super.processDelivery(command, offsetDeliveryTag(method));
}
```                     

### 7. `ChannelManager`
上一节中，`Channel`就是类`ChannelN`生成的。现在我们开始`Channel`的向上溯源。

`ChannelN`其实是被`ChannelManager`管理。
```java
public class ChannelManager
```
`ChannelManager`的`createChannel`返回可用的`ChannelN`。
```java
public ChannelN createChannel(AMQConnection connection, int channelNumber) throws IOException {
    ChannelN ch;
    synchronized (this.monitor) {
        if (channelNumberAllocator.reserve(channelNumber)) {
            ch = addNewChannel(connection, channelNumber);
        } else {
            return null;
        }
    }
    ch.open(); // now that it's been safely added
    return ch;
}

private ChannelN addNewChannel(AMQConnection connection, int channelNumber) {
    if (_channelMap.containsKey(channelNumber)) {
        // That number's already allocated! Can't do it
        // This should never happen unless something has gone
        // badly wrong with our implementation.
        throw new IllegalStateException("We have attempted to "
                + "create a channel with a number that is already in "
                + "use. This should never happen. "
                + "Please report this as a bug.");
    }
    ChannelN ch = instantiateChannel(connection, channelNumber, this.workService);
    _channelMap.put(ch.getChannelNumber(), ch);
    return ch;
}

protected ChannelN instantiateChannel(AMQConnection connection, int channelNumber, ConsumerWorkService workService) {
    return new ChannelN(connection, channelNumber, workService, this.metricsCollector);
}
```

### 8. `AMQConnection`
`ChannelManager`的`createChannel`将被`AMQConnection`调用。
```java
public class AMQConnection extends ShutdownNotifierComponent implements Connection, NetworkConnection
```
`AMQConnection`的`createChannel`将返回一个可用的`Channel`。
```java
/** Public API - {@inheritDoc} */
@Override
public Channel createChannel(int channelNumber) throws IOException {
    ensureIsOpen();
    ChannelManager cm = _channelManager;
    if (cm == null) return null;
    Channel channel = cm.createChannel(this, channelNumber);
    metricsCollector.newChannel(channel);
    return channel;
}

/** Public API - {@inheritDoc} */
@Override
public Channel createChannel() throws IOException {
    ensureIsOpen();
    ChannelManager cm = _channelManager;
    if (cm == null) return null;
    Channel channel = cm.createChannel(this);
    metricsCollector.newChannel(channel);
    return channel;
}
```
上面函数的`_channelManager`又是什么时候初始化的呢？在`public void start()`函数中。
```java
/**
 * Start up the connection, including the MainLoop thread.
 * Sends the protocol
 * version negotiation header, and runs through
 * Connection.Start/.StartOk, Connection.Tune/.TuneOk, and then
 * calls Connection.Open and waits for the OpenOk. Sets heart-beat
 * and frame max values after tuning has taken place.
 * @throws IOException if an error is encountered
 * either before, or during, protocol negotiation;
 * sub-classes {@link ProtocolVersionMismatchException} and
 * {@link PossibleAuthenticationFailureException} will be thrown in the
 * corresponding circumstances. {@link AuthenticationFailureException}
 * will be thrown if the broker closes the connection with ACCESS_REFUSED.
 * If an exception is thrown, connection resources allocated can all be
 * garbage collected when the connection object is no longer referenced.
 */
public void start()
        throws IOException, TimeoutException {
    initializeConsumerWorkService();
    initializeHeartbeatSender();
    this._running = true;
    // Make sure that the first thing we do is to send the header,
    // which should cause any socket errors to show up for us, rather
    // than risking them pop out in the MainLoop
    AMQChannel.SimpleBlockingRpcContinuation connStartBlocker =
        new AMQChannel.SimpleBlockingRpcContinuation();
    // We enqueue an RPC continuation here without sending an RPC
    // request, since the protocol specifies that after sending
    // the version negotiation header, the client (connection
    // initiator) is to wait for a connection.start method to
    // arrive.
    _channel0.enqueueRpc(connStartBlocker);
    try {
        // The following two lines are akin to AMQChannel's
        // transmit() method for this pseudo-RPC.
        _frameHandler.setTimeout(handshakeTimeout);
        _frameHandler.sendHeader();
    } catch (IOException ioe) {
        _frameHandler.close();
        throw ioe;
    }

    this._frameHandler.initialize(this);

    AMQP.Connection.Start connStart;
    AMQP.Connection.Tune connTune = null;
    try {
        connStart =
                (AMQP.Connection.Start) connStartBlocker.getReply(handshakeTimeout/2).getMethod();

        _serverProperties = Collections.unmodifiableMap(connStart.getServerProperties());

        Version serverVersion =
                new Version(connStart.getVersionMajor(),
                                   connStart.getVersionMinor());

        if (!Version.checkVersion(clientVersion, serverVersion)) {
            throw new ProtocolVersionMismatchException(clientVersion,
                                                              serverVersion);
        }

        String[] mechanisms = connStart.getMechanisms().toString().split(" ");
        SaslMechanism sm = this.saslConfig.getSaslMechanism(mechanisms);
        if (sm == null) {
            throw new IOException("No compatible authentication mechanism found - " +
                                          "server offered [" + connStart.getMechanisms() + "]");
        }

        LongString challenge = null;
        LongString response = sm.handleChallenge(null, this.username, this.password);

        do {
            Method method = (challenge == null)
                                    ? new AMQP.Connection.StartOk.Builder()
                                              .clientProperties(_clientProperties)
                                              .mechanism(sm.getName())
                                              .response(response)
                                              .build()
                                    : new AMQP.Connection.SecureOk.Builder().response(response).build();

            try {
                Method serverResponse = _channel0.rpc(method, handshakeTimeout/2).getMethod();
                if (serverResponse instanceof AMQP.Connection.Tune) {
                    connTune = (AMQP.Connection.Tune) serverResponse;
                } else {
                    challenge = ((AMQP.Connection.Secure) serverResponse).getChallenge();
                    response = sm.handleChallenge(challenge, this.username, this.password);
                }
            } catch (ShutdownSignalException e) {
                Method shutdownMethod = e.getReason();
                if (shutdownMethod instanceof AMQP.Connection.Close) {
                    AMQP.Connection.Close shutdownClose = (AMQP.Connection.Close) shutdownMethod;
                    if (shutdownClose.getReplyCode() == AMQP.ACCESS_REFUSED) {
                        throw new AuthenticationFailureException(shutdownClose.getReplyText());
                    }
                }
                throw new PossibleAuthenticationFailureException(e);
            }
        } while (connTune == null);
    } catch (TimeoutException te) {
        _frameHandler.close();
        throw te;
    } catch (ShutdownSignalException sse) {
        _frameHandler.close();
        throw AMQChannel.wrap(sse);
    } catch(IOException ioe) {
        _frameHandler.close();
        throw ioe;
    }

    try {
        int channelMax =
            negotiateChannelMax(this.requestedChannelMax,
                                connTune.getChannelMax());
        _channelManager = instantiateChannelManager(channelMax, threadFactory);

        int frameMax =
            negotiatedMaxValue(this.requestedFrameMax,
                               connTune.getFrameMax());
        this._frameMax = frameMax;

        int heartbeat =
            negotiatedMaxValue(this.requestedHeartbeat,
                               connTune.getHeartbeat());

        setHeartbeat(heartbeat);

        _channel0.transmit(new AMQP.Connection.TuneOk.Builder()
                            .channelMax(channelMax)
                            .frameMax(frameMax)
                            .heartbeat(heartbeat)
                          .build());
        _channel0.exnWrappingRpc(new AMQP.Connection.Open.Builder()
                                  .virtualHost(_virtualHost)
                                .build());
    } catch (IOException ioe) {
        _heartbeatSender.shutdown();
        _frameHandler.close();
        throw ioe;
    } catch (ShutdownSignalException sse) {
        _heartbeatSender.shutdown();
        _frameHandler.close();
        throw AMQChannel.wrap(sse);
    }

    // We can now respond to errors having finished tailoring the connection
    this._inConnectionNegotiation = false;
}
```
`instantiateChannelManager`将生成一个`ChannelManager`实例。
```java
protected ChannelManager instantiateChannelManager(int channelMax, ThreadFactory threadFactory) {
    ChannelManager result = new ChannelManager(this._workService, channelMax, threadFactory, this.metricsCollector);
    configureChannelManager(result);
    return result;
}

protected void configureChannelManager(ChannelManager channelManager) {
    channelManager.setShutdownExecutor(this.shutdownExecutor);
    channelManager.setChannelShutdownTimeout((int) ((requestedHeartbeat * CHANNEL_SHUTDOWN_TIMEOUT_MULTIPLIER) * 1000));
}

```

在`6.1`节中我们留下了一个问题：在谁会调用`handleFrame`呢？
回答就是是类`AMQConnection`!

下面继续跟踪调用链。
```java
类 ChannelN 的父类 AMQChannel 的 handleFrame 函数将调用类 ChannelN 的函数 processDelivery
                  ↓
类 AMQConnection 的 readFrame 函数//这里有2条路径
+-----------------+
|                 ↓1 //处理消息通讯
|类 AMQConnection的 内部类 MainLoop（implements Runnable）在 run 函数中循环调用
|                 ↓1
|类 AMQConnection 的 startMainLoop 函数
|                 ↓1
|类 SocketFrameHandler 的 initialize 函数
|                 ↓1
|类 AMQConnection 的 start 函数
|+----------------+
|↓1
+↓1---------------+
 ↓1               ↓2 //处理读写IO
 ↓1 类 AMQConnection 的 handleReadFrame 函数                  
 ↓1               ↓2
 ↓1 类 NioLoop（implements Runnable）在 run 函数中循环调用                  
 ↓1               ↓2
 ↓1 类 NioLoopContext 的 startIoLoops 函数
 ↓1               ↓2
 ↓1 类 NioLoopContext 的 initStateIfNecessary 函数
 ↓1               ↓2
 ↓1 类 SocketChannelFrameHandlerFactory 的 create 函数
 +----------------+
                  ↓
类 ConnectionFactory 的 newConnection 函数//合而为一                            
```


### 9. `AutorecoveringConnection`
`AMQConnection`的`createChannel`将被`AutorecoveringConnection`调用。
```java
public class AutorecoveringConnection implements RecoverableConnection, NetworkConnection
```
这里的`createChannel`将调用`RecoveryAwareAMQConnection`代理的`createChannel`方法。
```java
/**
 * @see com.rabbitmq.client.Connection#createChannel()
 */
@Override
public Channel createChannel() throws IOException {
    RecoveryAwareChannelN ch = (RecoveryAwareChannelN) delegate.createChannel();
    if (ch == null) {
        return null;
    } else {
        return this.wrapChannel(ch);
    }
}

/**
 * @see com.rabbitmq.client.Connection#createChannel(int)
 */
@Override
public Channel createChannel(int channelNumber) throws IOException {
    return delegate.createChannel(channelNumber);
}

```
因为`RecoveryAwareAMQConnection`是`AMQConnection`的扩展类，本质上是调用`AMQConnection`的`createChannel`。
```java
public class RecoveryAwareAMQConnection extends AMQConnection
```


### 10. `ConnectionFactory`
而最终`ConnectionFactory`将使用`AMQConnection`的实例。如果连接是可恢复的，则使用`AutorecoveringConnection`，否则使用`AMQConnection`。注意到`AMQConnection`实例化后就调用了`start`方法。而`AutorecoveringConnection`的`init`方法将通过调用`RecoveryAwareAMQConnection`的`start`方法，进而调用`AMQConnection`的`start`方法。

```java
/**
 * Create a new broker connection with a client-provided name, picking the first available address from
 * the list provided by the {@link AddressResolver}.
 *
 * If <a href="http://www.rabbitmq.com/api-guide.html#recovery">automatic connection recovery</a>
 * is enabled, the connection returned by this method will be {@link Recoverable}. Future
 * reconnection attempts will pick a random accessible address provided by the {@link AddressResolver}.
 *
 * @param executor thread execution service for consumers on the connection
 * @param addressResolver discovery service to list potential addresses (hostname/port pairs) to connect to
 * @param clientProvidedName application-specific connection name, will be displayed
 *                           in the management UI if RabbitMQ server supports it.
 *                           This value doesn't have to be unique and cannot be used
 *                           as a connection identifier e.g. in HTTP API requests.
 *                           This value is supposed to be human-readable.
 * @return an interface to the connection
 * @throws java.io.IOException if it encounters a problem
 * @see <a href="http://www.rabbitmq.com/api-guide.html#recovery">Automatic Recovery</a>
 */
public Connection newConnection(ExecutorService executor, AddressResolver addressResolver, String clientProvidedName)
    throws IOException, TimeoutException {
    if(this.metricsCollector == null) {
        this.metricsCollector = new NoOpMetricsCollector();
    }
    // make sure we respect the provided thread factory
    FrameHandlerFactory fhFactory = createFrameHandlerFactory();
    ConnectionParams params = params(executor);
    // set client-provided via a client property
    if (clientProvidedName != null) {
        Map<String, Object> properties = new HashMap<String, Object>(params.getClientProperties());
        properties.put("connection_name", clientProvidedName);
        params.setClientProperties(properties);
    }

    if (isAutomaticRecoveryEnabled()) {
        // see com.rabbitmq.client.impl.recovery.RecoveryAwareAMQConnectionFactory#newConnection
        AutorecoveringConnection conn = new AutorecoveringConnection(params, fhFactory, addressResolver, metricsCollector);

        conn.init();
        return conn;
    } else {
        List<Address> addrs = addressResolver.getAddresses();
        IOException lastException = null;
        for (Address addr : addrs) {
            try {
                FrameHandler handler = fhFactory.create(addr);
                AMQConnection conn = new AMQConnection(params, handler, metricsCollector);
                conn.start();
                this.metricsCollector.newConnection(conn);
                return conn;
            } catch (IOException e) {
                lastException = e;
            }
        }
        throw (lastException != null) ? lastException : new IOException("failed to connect");
    }
}

```

### 11. 连接的建立
```java
ConnectionFactory factory = new ConnectionFactory();
factory.setHost(Const.RABBIT_HOST);
factory.setPort(Const.RABBIT_PORT);
factory.setUsername(Const.RABBITMQ_USERNAME);
factory.setPassword(Const.RABBITMQ_PASSWORD);
factory.setAutomaticRecoveryEnabled(true);
factory.setHandshakeTimeout(60 * 1000);        
connection = factory.newConnection(getListAddress());
```
连接建立之后就可以根据连接创建`Channel`。
```java
channel = connection.createChannel();
```
回到开头，队列消费者的创建需要传入`Channel`而且需要绑定这个`Channel`，这样首尾内容就相呼应了。

### 12. 总结
#### 12.1 `Consumer`的调用线路（自底向上）
```java
Consumer consumer = new DefaultConsumer(channel)//类 DefaultConsumer 重写 handleDelivery 函数
                  ↓
channel.basicConsume(queueName, autoAck, consumer)//Consumer 与 Channel 关联
                  ↓
接口 Consumer 是一个回调接口，内部由类 ConsumerWorkService 设置的线程池调用
                  ↓
类 ConsumerDispatcher 的 handleDelivery 函数传入 Consumer 接口的实现（委托），封装成 Runnable ，随后调用 addWork 方法，放入 ConsumerWorkService 的线程池中执行
                  ↓
类 ChannelN 的函数 processDelivery 将调用类 dispatcher 的 handleDelivery 函数
                  ↓
类 ChannelN 的父类 AMQChannel 的 handleFrame 函数将调用类 ChannelN 的函数 processDelivery
                  ↓
类 AMQConnection 的 readFrame 函数//这里有2条路径
+-----------------+
|                 ↓1 //处理消息通讯
|类 AMQConnection的 内部类 MainLoop（implements Runnable）在 run 函数中循环调用
|                 ↓1
|类 AMQConnection 的 startMainLoop 函数
|                 ↓1
|类 SocketFrameHandler 的 initialize 函数
|                 ↓1
|类 AMQConnection 的 start 函数
|+----------------+
|↓1
+↓1---------------+
 ↓1               ↓2 //处理读写IO
 ↓1 类 AMQConnection 的 handleReadFrame 函数                  
 ↓1               ↓2
 ↓1 类 NioLoop（implements Runnable）在 run 函数中循环调用                  
 ↓1               ↓2
 ↓1 类 NioLoopContext 的 startIoLoops 函数
 ↓1               ↓2
 ↓1 类 NioLoopContext 的 initStateIfNecessary 函数
 ↓1               ↓2
 ↓1 类 SocketChannelFrameHandlerFactory 的 create 函数
 +----------------+
                  ↓
类 ConnectionFactory 的 newConnection 函数//合而为一            
```

#### 12.2 `Channel`的建立线路（自顶向下）
```java
ConnectionFactory factory = new ConnectionFactory();//连接工厂
                  ↓
connection = factory.newConnection(getListAddress());//创建连接。如果连接是可恢复的，则接口 Connection 的实现类是 AutorecoveringConnection，否则是类 AMQConnection
                  ↓                  
channel = connection.createChannel();//根据连接创建 Channel
                  ↓
类 AMQConnection 的 createChannel 函数创建 Channel//如果连接是可恢复的，类 AutorecoveringConnection 调用类 AMQConnection 的 createChannel 函数
                  ↓
类 AMQConnection 通过类 ChannelManager 的 createChannel 函数创建 ChannelN                  
                  ↓
类 ChannelN 实现了 Channel 接口//如果连接是可恢复的，类 RecoveryAwareChannelN 扩展了类 ChannelN                              
```
