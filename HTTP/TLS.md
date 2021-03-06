### 作用

不使用 SSL/TLS 的 HTTP 通信，就是不加密的通信。所有信息明文传播，带来了三大风险。

（1） 窃听风险（eavesdropping）：第三方可以获知通信内容。

（2） 篡改风险（tampering）：第三方可以修改通信内容。

（3） 冒充风险（pretending）：第三方可以冒充他人身份参与通信。

SSL/TLS 协议是为了解决这三大风险而设计的，希望达到：

（1） 所有信息都是加密传播，第三方无法窃听。

（2） 具有校验机制，一旦被篡改，通信双方会立刻发现。

（3） 配备身份证书，防止身份被冒充。

### TLS 是怎样实现的

TLS 协议可以分为两部分

（1）记录协议（Record Protocol）

通过使用客户端和服务端协商后的秘钥进行数据加密传输。

（2）握手协议（Handshake Protocol）

客户端和服务端进行协商，确定一组用于数据传输加密的秘钥串。

### 基本的运行过程

SSL/TLS 协议的基本思路是采用公钥加密法，也就是说，客户端先向服务器端索要公钥，然后用公钥加密信息，服务器收到密文后，用自己的私钥解密。

但是，这里有两个问题。

（1）如何保证公钥不被篡改？

解决方法：将公钥放在数字证书中。只要证书是可信的，公钥就是可信的。

现实中，通过 CA（Certificate Authority）来保证 public key 的真实性。CA 也是基于非对称加密算法来工作。有了 CA，B 会先把自己的 public key（和一些其他信息）交给 CA。CA 用自己的 private key 加密这些数据，加密完的数据称为 B 的数字证书。现在 B 要向 A 传递 public key，B 传递的是 CA 加密之后的数字证书。A 收到以后，会通过 CA 发布的 CA 证书（包含了 CA 的 public key），来解密 B 的数字证书，从而获得 B 的 public key。

但是等等，A 怎么确保 CA 证书不被劫持。C 完全可以把一个假的 CA 证书发给 A，进而欺骗 A。CA 的大杀器就是，CA 把自己的 CA 证书集成在了浏览器和操作系统里面。A 拿到浏览器或者操作系统的时候，已经有了 CA 证书，没有必要通过网络获取，那自然也不存在劫持的问题。

现在 A 和 B 都有了 CA 认证的数字证书。在交换 public key 的阶段，直接交换彼此的数字证书就行。而中间人 C，还是可以拦截 A 和 B 的 public key，也可以用 CA 证书解密获得 A 和 B 的 public key。但是，C 没有办法伪造 public key 了。因为 C 不在 CA 体系里面，C 没有 CA 的 private key，所以 C 是没有办法伪造出一个可以通过 CA 认证的数字证书。如果不能通过 CA 认证，A 和 B 自然也不会相信这个伪造的证书。所以，采用 CA 认证以后，A 和 B 的 public key 的真实性得到了保证，A 和 B 可以通过网络交换 public key（实际是被 CA 加密之后的数字证书）。

（2）公钥加密计算量太大，如何减少耗用的时间？

解决方法：每一次对话（session），客户端和服务器端都生成一个"对话密钥"（session key），用它来加密信息。由于"对话密钥"是对称加密，所以运算速度非常快，而服务器公钥只用于加密"对话密钥"本身，这样就减少了加密运算的消耗时间。

因此，SSL/TLS 协议的基本过程是这样的：

（1） 客户端向服务器端索要并验证公钥。

（2） 双方协商生成"对话密钥"。

（3） 双方采用"对话密钥"进行加密通信。

上面过程的前两步，又称为"握手阶段"（handshake）。

所以，在现代，A 和 B 之间要进行安全，省心的网络通信，需要经过以下几个步骤

（1）通过 CA 体系交换 public key

（2）通过非对称加密算法，交换用于对称加密的密钥

（3）通过对称加密算法，加密正常的网络通信

这基本就是 SSL/TLS 的工作过程了。

### 握手阶段的详细过程

"握手阶段"涉及四次通信，我们一个个来看。需要注意的是，"握手阶段"的所有通信都是明文的。

#### 1. 客户端发出请求（ClientHello）

首先，客户端（通常是浏览器）先向服务器发出加密通信的请求，这被叫做 ClientHello 请求。

在这一步，客户端主要向服务器提供以下信息。

（1） 支持的协议版本，比如 TLS 1.0 版。

（2） 一个客户端生成的随机数，稍后用于生成"对话密钥"。

（3） 支持的加密方法，比如 RSA 公钥加密。

（4） 支持的压缩方法。

#### 2. 服务器回应（SeverHello）

服务器收到客户端请求后，向客户端发出回应，这叫做 SeverHello。服务器的回应包含以下内容。

（1） 确认使用的加密通信协议版本，比如 TLS 1.0 版本。如果浏览器与服务器支持的版本不一致，服务器关闭加密通信。

（2） 一个服务器生成的随机数，稍后用于生成"对话密钥"。

（3） 确认使用的加密方法，比如 RSA 公钥加密。

（4） 服务器证书。

除了上面这些信息，如果服务器需要确认客户端的身份，就会再包含一项请求，要求客户端提供"客户端证书"。比如，金融机构往往只允许认证客户连入自己的网络，就会向正式客户提供 USB 密钥，里面就包含了一张客户端证书。

#### 3. 客户端回应

客户端收到服务器回应以后，首先验证服务器证书。如果证书不是可信机构颁布、或者证书中的域名与实际域名不一致、或者证书已经过期，就会向访问者显示一个警告，由其选择是否还要继续通信。

如果证书没有问题，客户端就会从证书中取出服务器的公钥。然后，向服务器发送下面三项信息。

（1） 一个随机数。该随机数用服务器公钥加密，防止被窃听。

（2） 编码改变通知，表示随后的信息都将用双方商定的加密方法和密钥发送。

（3） 客户端握手结束通知，表示客户端的握手阶段已经结束。这一项同时也是前面发送的所有内容的 hash 值，用来供服务器校验。

上面第一项的随机数，是整个握手阶段出现的第三个随机数，又称"pre-master key"。有了它以后，客户端和服务器就同时有了三个随机数，接着双方就用事先商定的加密方法，各自生成本次会话所用的同一把"会话密钥"。

#### 4. 服务器的最后回应

服务器收到客户端的第三个随机数 pre-master key 之后，计算生成本次会话所用的"会话密钥"。然后，向客户端最后发送下面信息。

（1）编码改变通知，表示随后的信息都将用双方商定的加密方法和密钥发送。

（2）服务器握手结束通知，表示服务器的握手阶段已经结束。这一项同时也是前面发送的所有内容的 hash 值，用来供客户端校验。

至此，整个握手阶段全部结束。接下来，客户端与服务器进入加密通信，就完全是使用普通的 HTTP 协议，只不过用"会话密钥"加密内容。

### 握手阶段有三点需要注意。

（1）生成对话密钥一共需要三个随机数。

（2）握手之后的对话使用"对话密钥"加密（对称加密），服务器的公钥和私钥只用于加密和解密"对话密钥"（非对称加密），无其他作用。

（3）服务器公钥放在服务器的数字证书之中。

### DH 算法的握手阶段

整个握手阶段都不加密（也没法加密），都是明文的。因此，如果有人窃听通信，他可以知道双方选择的加密方法，以及三个随机数中的两个。整个通话的安全，只取决于第三个随机数（Premaster secret）能不能被破解。

虽然理论上，只要服务器的公钥足够长（比如 2048 位），那么 Premaster secret 可以保证不被破解。但是为了足够安全，我们可以考虑把握手阶段的算法从默认的 RSA 算法，改为 Diffie-Hellman 算法（简称 DH 算法）。

采用 DH 算法后，Premaster secret 不需要传递，双方只要交换各自的参数，就可以算出这个随机数。

第三步和第四步由传递 Premaster secret 变成了传递 DH 算法所需的参数，然后双方各自算出 Premaster secret。这样就提高了安全性。

#### 1. SSL/TLS 协议运行机制的概述

http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html

#### 2. 图解 SSL/TLS 协议

http://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html
