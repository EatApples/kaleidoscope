### 1. Maven 依赖
```xml
<dependency>
  <groupId>com.jcraft</groupId>
  <artifactId>jsch</artifactId>
  <version>0.1.54</version>
</dependency>
```

### 2. 通过 ChannelExec 交互，上下文没有关联
```java
/**
 * 通过 ChannelExec 交互，上下文没有关联
 *
 * @param host
 * @param port
 * @param user
 * @param password
 * @param commands
 * @return
 */
public String[] ChannelExecCommand(String host, int port, String user, String password, String[] commands) {

    String[] responses = new String[commands.length];
    JSch jsch = new JSch();
    Session session = null;
    String reponse = "FAIL";
    try {
        session = jsch.getSession(user, host, port);
        session.setConfig("StrictHostKeyChecking", "no");
        session.setPassword(password);
        session.connect();
        for (int i = 0; i < commands.length; i++) {
            ChannelExec channelExec = (ChannelExec) session.openChannel("exec");
            try {
                InputStream in = channelExec.getInputStream();
                channelExec.setCommand(commands[i]);
                LOGGER.info("command:[" + commands[i] + "]");
                channelExec.setErrStream(System.err);
                channelExec.connect();
                reponse = getStringFromStream(in);
                // suspend(2000);
            }
            catch (Exception e) {
                LOGGER.error("run command fail:", e);
                reponse = exception2String(e);
            }
            finally {
                channelExec.disconnect();
                responses[i] = reponse;
                LOGGER.info("response:[" + reponse + "]");
            }
        }
    }
    catch (Exception e) {
        LOGGER.error("run command fail:", e);
    }
    finally {
        if (session != null) {
            session.disconnect();
        }
    }
    return responses;
}
```
### 3. 通过 ChannelShell 交互，上下文语义关联
```java
/**
 * 通过 ChannelShell 交互，上下文语义关联
 *
 * @param host
 * @param port
 * @param user
 * @param password
 * @param commands
 * @return
 */
public String ChannelShellCommand(String host, int port, String user, String password, String[] commands) {

    String reponse = "FAIL";
    JSch jsch = new JSch();
    Session session = null;

    try {
        session = jsch.getSession(user, host, port);
        session.setConfig("StrictHostKeyChecking", "no");
        session.setPassword(password);
        session.connect();
        ChannelShell channel = (ChannelShell) session.openChannel("shell");
        channel.connect();
        // 从远程端到达的所有数据都能从这个流中读取到
        InputStream inputStream = channel.getInputStream();
        // 写入该流的所有数据都将发送到远程端。
        OutputStream outputStream = channel.getOutputStream();
        // 使用PrintWriter流的目的就是为了使用println这个方法
        // 好处就是不需要每次手动给字符串加\n
        PrintWriter printWriter = new PrintWriter(outputStream);

        for (int i = 0; i < commands.length; i++) {
            LOGGER.info("command:[" + commands[i] + "]");
            printWriter.println(commands[i]);

        }
        // 加上个就是为了，结束本次交互
        printWriter.println("exit");
        printWriter.flush();
        // 获取结果
        reponse = getStringFromStream(inputStream);

    }
    catch (Exception e) {
        LOGGER.error("run command fail:", e);
    }
    finally {
        if (session != null) {
            session.disconnect();
        }
    }
    LOGGER.info("response:[" + reponse + "]");
    return reponse;
}
```
### 4. 将本地文件名为src的文件，上传到目标服务器
```java
/**
 * 将本地文件名为src的文件上传到目标服务器，目标文件名为dst，若dst为目录，则目标文件名将与src文件名相同。 采用默认的传输模式：OVERWRITE
 *
 * @param host
 * @param port
 * @param user
 * @param password
 * @param src
 * @param dst
 *            如果 dst 是目录（注意，目录必须存在），则目标文件名与源文件名相同，文件的绝对路径为 dst/源文件名。 如果 dst 是文件名，文件的绝对路径为 dst。
 * @return
 */
public String ChannelSftpCommand(String host, int port, String user, String password, String src, String dst) {

    String reponse = "FAIL";
    JSch jsch = new JSch();
    Session session = null;

    try {
        session = jsch.getSession(user, host, port);
        session.setConfig("StrictHostKeyChecking", "no");
        session.setPassword(password);
        // 设置登陆超时时间
        session.connect(10000);
        // 创建sftp通信通道
        ChannelSftp sftp = (ChannelSftp) session.openChannel("sftp");
        sftp.connect(10000);
        /**
         * @see http://www.cnblogs.com/longyg/archive/2012/06/25/2556576.html
         */
        sftp.put(src, dst);
        reponse = "SUCCESS";
    }
    catch (Exception e) {
        LOGGER.error("run command fail:", e);
    }
    finally {
        if (session != null) {
            session.disconnect();
        }
    }
    LOGGER.info("response:[" + reponse + "]");
    return reponse;
}

```

### 相关资料
#### 1. java连接SSH服务器并执行shell命令
http://www.cnblogs.com/flyingzl/articles/2145032.html

#### 2. java使用ssh连接Linux并执行命令
https://blog.csdn.net/angel_xiaa/article/details/52355625
