
|模式        					|发送                                                           						|接收													|
---------------------------------------------------------------------------------------------------------------------------------------------------------
|默认       						|channel.basicPublish("", ROUTING_KEY, MESSAGE);(EXCHANGE_NAME为空,ROUTING_KEY值为QUEUE_NAME)|没有 bind|
|direct      				|channel.exchangeDeclare(EXCHANGE_NAME, "direct");              channel.exchangeDeclare(EXCHANGE_NAME, "direct");
                channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, MESSAGE);     channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);

|topic       channel.exchangeDeclare(EXCHANGE_NAME, "topic");               channel.exchangeDeclare(EXCHANGE_NAME, "topic");
                channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, MESSAGE);     channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);
                direct的加强版，ROUTING_KEY用(.)隔开，(*，仅匹配一个)、(#，匹配零或多个)

|fanout      channel.exchangeDeclare(EXCHANGE_NAME, "fanout");              channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
                channel.basicPublish(EXCHANGE_NAME, "", MESSAGE);              channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "");
                topic的完全版，ROUTING_KEY为空，意味着只要绑定EXCHANGE_NAME，都能接收

|诡异的模式
    headers     channel.exchangeDeclare(EXCHANGE_NAME, "headers");             channel.exchangeDeclare(EXCHANGE_NAME, "headers");
                发送消息定义一些header                                         接收绑定一些header，忽略ROUTING_KEY，带有这些header的消息都会被接收