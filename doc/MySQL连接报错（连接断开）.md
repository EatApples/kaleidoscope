# 错误提示:
```
The last packet successfully received from the server was 35,427,592 milliseconds ago.  The last packet sent successfully to the server was 1 milliseconds ago.; nested exception is com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure

The last packet successfully received from the server was 35,427,592 milliseconds ago.  The last packet sent successfully to the server was 1 milliseconds ago.
        at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
        at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
        at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
        at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
        at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
        at com.mysql.jdbc.SQLError.createCommunicationsException(SQLError.java:989)
        at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3556)
        at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3456)
        at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3897)
        at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2524)
        at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2677)
        at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2549)
        at com.mysql.jdbc.PreparedStatement.executeInternal(PreparedStatement.java:1861)
        at com.mysql.jdbc.PreparedStatement.executeUpdateInternal(PreparedStatement.java:2073)
        at com.mysql.jdbc.PreparedStatement.executeUpdateInternal(PreparedStatement.java:2009)
        at com.mysql.jdbc.PreparedStatement.executeLargeUpdate(PreparedStatement.java:5098)
        at com.mysql.jdbc.PreparedStatement.executeUpdate(PreparedStatement.java:1994)
        at org.springframework.jdbc.core.JdbcTemplate$2.doInPreparedStatement(JdbcTemplate.java:877)
        at org.springframework.jdbc.core.JdbcTemplate$2.doInPreparedStatement(JdbcTemplate.java:870)
        at org.springframework.jdbc.core.JdbcTemplate.execute(JdbcTemplate.java:633)
        ... 11 common frames omitted
Caused by: java.io.EOFException: Can not read response from server. Expected to read 4 bytes, read 0 bytes before connection was unexpectedly lost.
        at com.mysql.jdbc.MysqlIO.readFully(MysqlIO.java:3008)
        at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3466)
        ... 24 common frames omitted
```
# 错误分析:
 
 数据库连接已经关闭或者失效后仍然在执行操作，导致mysql服务没返回数据 
 
     1，客户端连接池中连接已经失效。但是连接池还没有检测到，当操作数据库时，启用该连接，抛出该错误
     2，mysql服务器，已关闭该连接，但客户端连接池中该连接，尚未检测到。当用该连接操作数据库时，抛出该错

# 处理办法:

     1，每次连接操作数据库时，检测该连接的有效性
     2，缩短监控空闲线程的时间

# 分析应该是第二种情况:

在晚上没有业务发生，也就没有数据库连接操作。
假定数据库连接保持的时间是8小时，从时间上 

>The last packet successfully received from the server was 35,427,592 milliseconds ago

等待时间约10个小时，连接已断开。我们在连接中添加了 autoReconnect=true ，重连后操作恢复。
