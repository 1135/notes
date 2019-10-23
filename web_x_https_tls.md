>[HTTPS连接最初的若干毫秒 2009经典译文](https://www.infoq.cn/article/HTTPS-Connection-Jeff-Moser)

>[TLS handshake概述 - IBM Knowledge Center](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_7.1.0/com.ibm.mq.doc/sy10660_.htm)

>[传输层安全 - Transport Layer Security - Wikipedia](https://en.wikipedia.org/wiki/Transport_Layer_Security)

>TLS 1.2 [RFC 5246 - The Transport Layer Security (TLS) Protocol Version 1.2](https://tools.ietf.org/html/rfc5246)

### 加密

* 对称加密(Symmetrical Encryption)
  * 1个密钥
    * 通信双方的加密与解密过程 都使用这同一个密钥
  * 对称加密的算法:`Blowfish，AES，RC4，DES，RC5, RC6` 最广泛使用的对称加密算法是AES-128，AES-192和AES-256
  * 对称加密的缺点:主要缺点是所涉及的各方必须在解密数据之前交换用于加密数据的密钥
  * 对称加密的优点:速度快


* 非对称加密(Asymmetrical Encryption) 即 公钥加密([Public-key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography))
  * 2个密钥
    * 公钥 公开的 主要用来加密消息
      * 使用公钥加密的消息只能使用私钥解密  即 私钥是唯一可以解密【公钥加密的消息】的密钥
    * 私钥 保密的 主要用来解密消息
      * 使用私钥加密的消息也可以使用公钥解密
  * 非对称加密的算法:`EIGamal，RSA，DSA，PKCS，Elliptic curve techniques(椭圆曲线)`
  * 非对称加密的优点:现代的算法 消息传输过程的安全性高 所以非对称加密在互联网上应用广泛
  * 对称加密的缺点:速度慢 非对称加密比对称加密花费的时间相对更多

### 非对称加密的安全性的原理

非对称加密算法 - [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem))

RSA算法安全性的核心是:基于n的两个质数分解难题. 即对极大整数n做因数分解的难度等同于RSA的安全性.(n=p×q ,n为极大整数,且p和q都为质数 如13和61)

### 非对称加密应用 - 数字签名(Digital signatures)

数字签名过程:
* 1.发件人计算message得到信息摘要(message digest)m-dig
  * 使用发件人的私钥,加密信息摘要m-dig从而得到数字签名
* 2.发送人发送数字签名
* 3.接收人使用发送人的公钥，解密数字签名，重新生成发送方的消息摘要
* 4.接收人根据接收的消息数据计算消息摘要，并验证两个摘要是否相同


数字签名的作用 - 接受人如果验证了数字签名则可确定:
  * message完整性 传输过程中消息未修改
  * 身份验证(只有发送人才知道私钥，所以谁提供了私钥，谁就被认为是发送人)

### 非对称加密应用 - 数字证书(Digital certificates

公钥基础架构(PKI,[Public key infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure)):用于创建，存储和分发数字证书的系统，用于验证特定公钥是否属于某个实体。PKI创建将公钥映射到实体的数字证书，将这些证书安全地存储在中央存储库中，并在需要时撤消证书。

PKI包括:
* certificate authority (CA)
  * 存储 颁发 签名 数字证书
  * stores, issues and signs the digital certificates)
* registration authority (RA)
  * 验证(请求将其数字证书存储在CA的)实体的身份
  * which verifies the identity of entities requesting their digital certificates to be stored at the CA
* central directory
  * 中央目录，一个安全的地方，可存储和索引密钥
  * a secure location in which to store and index keys
* certificate management system
  * 证书管理系统 用于管理对存储证书的访问 或要颁发的证书的交付
  * managing things like the access to stored certificates or the delivery of the certificates to be issued.
* certificate policy
  * 证明PKI遵循其自身程序的证书策略。其目的是让外人分析PKI的可信度。
  * certificate policy stating the PKI's requirements concerning its procedures. Its purpose is to allow outsiders to analyze the PKI's trustworthiness.

证书颁发机构(CA,[Certificate authority](https://en.wikipedia.org/wiki/Certificate_authority)):作为一个受信任的第三方,颁发数字证书(digital certificates)即 公钥证书([Public key certificate](https://en.wikipedia.org/wiki/Public_key_certificate)),以确保某实体的公钥真正属于该实体。

公钥证书的格式的标准:在密码学中[X.509](https://en.wikipedia.org/wiki/X.509)是定义公钥证书格式的标准。

### SSL/TLS 基本过程

* 握手阶段(TLS handshake)
  * 1 客户端向服务器端索要公钥 并验证
  * 2 双方协商生成一个"对话密钥"

### SSL/TLS连接建立过程 (重点是TLS handshake)

* TCP 3次握手过程(成功之后进行TLS握手)
  * SYN
  * SYN ACK
  * ACK
* TLS/SSL握手过程(TLS handshake)  "握手阶段"涉及的四次通信都是"明文"传输,概述如下
  * (1).确认当前TLS server(Web Server)使用的TLS协议的版本号、加密算法的种类等信息
  * (2).选择加密算法(Select cryptographic algorithms)
  * (3).二者交换数字证书并验证数字证书 从而验证了彼此的身份(Authenticate each other by exchanging and validating digital certificates.)
  * (4).二者使用使用非对称加密技术生成一个共享的密钥 即对称加密的密钥(Use asymmetric encryption techniques to generate a shared secret key.)
* TLS/SSL通信
  * SSL 的握手后，TLS Client和TLS Server双方使用相同的对称密钥"对话密钥"对消息加密，实现加密通信.同时进行通讯完整性的检验


```
      Client                                            Server

# TCP 3次握手过程(成功之后进行TLS握手)
# TLS/SSL握手对象: TLS client(浏览器) 和 TLS server(Web Server)
# TLS/SSL握手过程("握手阶段"涉及四次通信，都是"明文"通信)

                              握手过程 第?次通信
      ClientHello                  ----1---->


                                                      ServerHello
                                                     Certificate*
                                               ServerKeyExchange*
                                              CertificateRequest*
                                   <----2----      ServerHelloDone


      Certificate*
      ClientKeyExchange
      CertificateVerify*
      [ChangeCipherSpec]
      Finished                     -----3--->


                                               [ChangeCipherSpec]
                                   <----4----             Finished


# TLS/SSL握手结果:seesion key
# 最后协商的结果是:二者有了同一个对称加密密钥(session key,会话密钥).用于二者互相通信时对messages进行对称加密.
# 即 TLS握手使TLS client(浏览器)能够与TLS server(Web Server)这二者间建立了交流数据必需的密钥(对称加密的密钥).
      Application Data             <------->     Application Data
```
