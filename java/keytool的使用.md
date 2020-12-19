### keytool 说明文档

```yml
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

生成证书
keytool -genkey -v -alias {别名} -keyalg {RSA} -keysize {1024} -validity {365} -storetype {PKCS12} -keystore {密钥库存储路径} -keypass {密钥口令} -storepass {密钥库口令} -dname "CN={姓名},OU={组织单位名称},O={组织名称},L={省/市/自治区名称},ST={州或省份名称},C=CN"

导出公钥
keytool -export -alias {别名} -keystore {密钥库} -file {导出文件}

查看公钥
keytool -printcert -v -file {公钥文件}

查看密钥库
keytool -list -rfc -keystore {密钥库}

删除密钥库中的条目
keytool -delete -alias {别名} -keystore {密钥库}

公钥导入授信库
keytool -import -alias {别名} -v -file {公钥文件} -keystore {密钥库}
```

### truststore 与 keystore

Keytool 将密钥（key）和证书（certificates）存在一个称为 keystore 的文件中

在 keystore 里，包含两种数据：

（1）密钥实体（Key entity）— 密钥（secret key）又或者是私钥和配对公钥（采用非对称加密）
包含了私钥，所以是一个需要保密的文件

（2）可信任的证书实体（trusted certificate entries）— 只包含公钥

证书库中的一条证书可以导出数字证书文件，数字证书文件只包括主体信息和对应的公钥

keystore 和 truststore 从其文件格式来看其实是一个东西，只是为了方便管理将其分开。

keystore 中一般保存的是我们的私钥，用来加解密或者为别人做签名。

truststore 中保存的是一些可信任的证书，主要是 java 在代码中访问某个 https 的时候对被访问者进行认证的，以确保其实可信任的。

truststore 里存放的是只包含公钥的数字证书，代表了可以信任的证书，而 keystore 是包含私钥的。
