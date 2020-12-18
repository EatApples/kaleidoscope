### AK/SK 原理

云主机需要通过使用 Access Key Id / Secret Access Key 加密的方法来验证某个请求的发送者身份。

Access Key Id（AK）用于标示用户，Secret Access Key（SK）是用户用于加密认证字符串和云厂商用来验证认证字符串的密钥，其中 SK 必须保密。 AK/SK 原理使用对称加解密。

### AK/SK 使用机制

云主机接收到用户的请求后，系统将使用 AK 对应的相同的 SK 和同样的认证机制生成认证字符串，并与用户请求中包含的认证字符串进行比对。如果认证字符串相同，系统认为用户拥有指定的操作权限，并执行相关操作；如果认证字符串不同，系统将忽略该操作并返回错误码。

### 流程

客户端：
（1）构建 http 请求（包含 access key）；
（2）使用请求内容和 使用 secret access key 计算的签名(signature)；
（3）发送请求到服务端。

服务端：
（1）根据发送的 access key 查找数据库得到对应的 secret-key；
（2）使用同样的算法将请求内容和 secret-key 一起计算签名（signature），与客户端步骤 2 相同；
（3）对比用户发送的签名和服务端计算的签名，两者相同则认证通过，否则失败。

#### 1. AK/SK 签名认证算法详解

https://support.huaweicloud.com/devg-apisign/api-sign-algorithm.html

#### 2. 签名机制

https://help.aliyun.com/document_detail/25492.html?spm=a2c4g.11186623.6.1271.365e69b1pMJ13j
