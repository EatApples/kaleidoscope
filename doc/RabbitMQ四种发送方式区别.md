
|模式 |发送 |接收	|
|:----|:----|:---|
|默认|<pre>channel.basicPublish("", ROUTING_KEY, MESSAGE);<p>EXCHANGE_NAME为空,ROUTING_KEY值为QUEUE_NAME</pre>|<pre>没有 bind</pre>|
|direct|<pre>channel.exchangeDeclare(EXCHANGE_NAME, "direct");<p>channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, MESSAGE);</pre> |<pre>channel.exchangeDeclare(EXCHANGE_NAME, "direct");<p>channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);</pre>|
|topic|<pre>channel.exchangeDeclare(EXCHANGE_NAME, "topic");<p>channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, MESSAGE);<p>direct的加强版，ROUTING_KEY用(.)隔开，(*，仅匹配一个)、(#，匹配零或多个)</pre>|<pre>channel.exchangeDeclare(EXCHANGE_NAME, "topic");<p>channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);</pre>|
|fanout|<pre>channel.exchangeDeclare(EXCHANGE_NAME, "fanout");<p>channel.basicPublish(EXCHANGE_NAME, "", MESSAGE);<p>topic的完全版，ROUTING_KEY为空，意味着只要绑定EXCHANGE_NAME，都能接收</pre>|<pre>channel.exchangeDeclare(EXCHANGE_NAME, "fanout");<p>channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "");</pre>|
|诡异的headers|<pre>channel.exchangeDeclare(EXCHANGE_NAME, "headers");<p>发送消息定义一些header</pre>|<pre>channel.exchangeDeclare(EXCHANGE_NAME, "headers");<p>接收绑定一些header，忽略ROUTING_KEY，带有这些header的消息都会被接收</pre>|
