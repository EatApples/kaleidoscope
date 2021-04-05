### 1. keytool 指令说明

```sh
-list                           列出秘钥库中的条目
-rfc                            以RFC样式输出
-genkey                         生成秘钥
-alias <alias>                  要处理的条目的别名
-keyalg <keyalg>                密钥算法名称
-keysize <keysize>              密钥位大小
-sigalg <sigalg>                签名算法名称
-destalias <destalias>          目标别名
-dname <dname>                  唯一判别名，CN：姓名；OU：组织单位名称；O：组织名称；L：省/市/自治区名称；C：国家/地区代码
-startdate <startdate>          证书有效期开始日期/时间
-ext <value>                    X.509 扩展
-validity <valDays>             有效天数
-keypass <arg>                  密钥口令
-keystore <keystore>            密钥库名称
-storepass <arg>                密钥库口令
-storetype <storetype>          密钥库类型
-providername <providername>    提供方名称
-providerclass <providerclass>  提供方类名
-providerarg <arg>              提供方参数
-providerpath <pathlist>        提供方类路径
-v                              详细输出
-protected                      通过受保护的机制的口令
```

### 2. 示例操作

#### 2.1 生成证书

```sh
keytool -genkey -v -alias {别名} -keyalg {RSA} -keysize {1024} -validity {365} -storetype {PKCS12} -keystore {密钥库存储路径} -keypass {密钥口令} -storepass {密钥库口令} -dname "CN={姓名},OU={组织单位名称},O={组织名称},L={省/市/自治区名称},ST={州或省份名称},C=CN"
```

#### 2.2 导出公钥

```sh
keytool -export -alias {别名} -keystore {密钥库} -file {导出文件}
```

#### 2.3 查看公钥

```sh
keytool -printcert -v -file {公钥文件}
```

#### 2.4 查看密钥库

```sh
keytool -list -rfc -keystore {密钥库}
```

#### 2.5 删除密钥库中的条目

```sh
keytool -delete -alias {别名} -keystore {密钥库}
```

#### 2.6 公钥导入授信库

```sh
keytool -import -alias {别名} -v -file {公钥文件} -storetype {PKCS12} -keystore {密钥库}
```

### 3. truststore 与 keystore

Keytool 将密钥（key）和证书（certificates）存在一个称为 keystore 的文件中

在 keystore 里，包含两种数据：

（1）密钥实体（Key entity）— 密钥（secret key）又或者是私钥和配对公钥（采用非对称加密）包含了私钥，所以是一个需要保密的文件

（2）可信任的证书实体（trusted certificate entries）— 只包含公钥

证书库中的一条证书可以导出数字证书文件，数字证书文件只包括主体信息和对应的公钥

keystore 和 truststore 从其文件格式来看其实是一个东西，只是为了方便管理将其分开。

keystore 中一般保存的是我们的私钥，用来加解密或者为别人做签名。

truststore 中保存的是一些可信任的证书，主要是 java 在代码中访问某个 https 的时候对被访问者进行认证的，以确保其实可信任的。

truststore 里存放的是只包含公钥的数字证书，代表了可以信任的证书，而 keystore 是包含私钥的。

总结：

（1）keystore 储存自己的私钥，不对外提供
（2）可以导出自己的公钥提供给别人
（2）可以将别人的公钥存入授信库 truststore

### 4. 双端开启 TLS 一般步骤

#### 4.1 服务端生成证书

```sh
keytool -genkey -v -alias {别名} -keyalg {RSA} -keysize {1024} -validity {365} -storetype {PKCS12} -keystore {密钥库存储路径} -keypass {密钥口令} -storepass {密钥库口令} -dname "CN={姓名},OU={组织单位名称},O={组织名称},L={省/市/自治区名称},ST={州或省份名称},C=CN"

# 示例：
keytool -genkey -v -alias sslServer -keyalg RSA -keysize 1024 -validity 36500 -storetype PKCS12 -keystore ./sslServer.p12 -keypass 123456 -storepass 123456 -dname "CN=XXX@YYY.com,OU=AAA,O=BBB,L=CCC,ST=DDD,C=CN"
```

#### 4.2 客户端生成证书

```sh
keytool -genkey -v -alias {别名} -keyalg {RSA} -keysize {1024} -validity {365} -storetype {PKCS12} -keystore {密钥库存储路径} -keypass {密钥口令} -storepass {密钥库口令} -dname "CN={姓名},OU={组织单位名称},O={组织名称},L={省/市/自治区名称},ST={州或省份名称},C=CN"

# 示例：
keytool -genkey -v -alias sslClient -keyalg RSA -keysize 1024 -validity 36500 -storetype PKCS12 -keystore ./sslClient.p12 -keypass 654321 -storepass 654321 -dname "CN=PPP@QQQ.com,OU=EEE,O=FFF,L=GGG,ST=HHH,C=CN"
```

#### 4.3 导出公钥

```sh
keytool -export -alias {别名} -keystore {密钥库} -file {导出文件}

# 示例：
#（1）导出客户端的公钥：
keytool -export -alias sslClient -keystore ./sslClient.p12  -rfc -file ./sslClient.cer

#（2）导出服务端的公钥：
keytool -export -alias sslServer -keystore ./sslServer.p12  -rfc -file ./sslServer.cer
```

#### 4.4 公钥导入授信库

```sh
keytool -import -alias {别名} -v -file {公钥文件} -storetype {PKCS12} -keystore {密钥库}
# 示例：
#（1）客户端公钥导入服务端授信库
keytool -import -alias sslClient -v -file ./sslClient.cer -storetype PKCS12 -keystore ./sslServerTrust.p12

#（2）服务端公钥导入客户端授信库
keytool -import -alias sslServer -v -file ./sslServer.cer -storetype PKCS12 -keystore ./sslClientTrust.p12
```

#### 4.5 服务端的配置

（1）将生成的

- 服务端证书（里面有私钥） （`sslServer.p12`）
- 服务端的授信库（里面有客户端的公钥）（`sslServerTrust.p12`）

放在 `resource/tls` 目录下。

PS：打包时可能遇到问题，参见[参考资料](https://blog.csdn.net/kevin_mails/article/details/84590449)

（2）在配置文件 `application.yml` 中配置：

```yml
# TLS配置
# 密钥库地址
server.ssl.key-store=classpath:tls/sslServer.p12
server.ssl.key-store-password=123456
server.ssl.key-alias=sslServer
server.ssl.keyStoreType=PKCS12

# 开启双向认证（客户端与服务端互相校验）
# 信任库地址
server.ssl.trust-store=classpath:tls/sslServerTrust.p12
server.ssl.trust-store-password=123456
server.ssl.client-auth=need
server.ssl.trust-store-type=PKCS12
server.ssl.trust-store-provider=SUN
```

#### 4.6 客户端的配置

见[参考资料](https://blog.csdn.net/qq_42078975/article/details/104351153)

### 5. 参考资料

#### 1. 配置 SpringBoot 实现 TLS 双向认证

https://www.jianshu.com/p/e1aaa5e9de17

#### 2.【springboot】配置实现 https 单项认证和双向认证

https://blog.csdn.net/qq_42078975/article/details/104351153

#### 3. spring boot，https，双向 ssl 认证

https://www.cnblogs.com/htuao/p/10091458.html

#### 4. java httpclinet 请求 https 地址报 java.io.IOException: Invalid keystore format 解决办法

https://blog.csdn.net/kevin_mails/article/details/84590449

#### 5. httpClient，Certificate for ip/域名 doesn't match any of the subject altinative names: []问题处理

https://blog.csdn.net/RL_LEEE/article/details/87920903
