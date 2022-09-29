---
title: 数字签名，PKI 与 SSL/TLS
tag: [SSL, 加密, 签名, 抓包]
key: 2022-09-25-数字签名，PKI 与 SSL/TLS
---

[“先签名后加密”的思考-云社区-华为云](https%3A%2F%2Fbbs.huaweicloud.com%2Fblogs%2F205524)

[公开密钥基础建设-WIKI](https%3A%2F%2Fzh.wikipedia.org%2Fzh-sg%2F%25E5%2585%25AC%25E9%2596%258B%25E9%2587%2591%25E9%2591%25B0%25E5%259F%25BA%25E7%25A4%258E%25E5%25BB%25BA%25E8%25A8%25AD)

[网络安全系统之四 PKI 体系 - PBDragon - 博客园](https%3A%2F%2Fwww.cnblogs.com%2FPBDragon%2Fp%2F12694274.html)
[数字签名、数字证书与 HTTPS 是什么关系? - 知乎](https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F52493697%2Fanswer%2F130813213)

[密钥交换算法](https%3A%2F%2Fwww.liaoxuefeng.com%2Fwiki%2F1252599548343744%2F1304227905273889)

[一文详解 HTTPS 与 TLS 证书链校验 - xiaxueliang - 博客园](https%3A%2F%2Fwww.cnblogs.com%2Fxiaxveliang%2Fp%2F13183175.html)

首先非对称密钥加密，和摘要算法都了解，但是如何用其签名，可能一时分不开。

本文就从数字签名——PKI 体系——SSL/TLS 进行讲解，**注意并没有把过程细节详细讲清楚，只适合大致了解。**

# 1. 数字签名

加密能够保护消息不被第三方读取，但是攻击者有可能拦截加密的消息并进行篡改，所以需要**数字签名**来保证**消息的完整性**，并**验证发送者的身份**。

数字签名的公私钥是和非对称加密反过来的，具体如下图所示。私钥是签名者用来加密数据的散列值的，而公钥是分发给外界，用来解密散列值，进而验证签名的数据是由签名者发送过来的，且没有被篡改。

![](https://xdo0.github.io/_posts/imgsrc/boxcnuijxGoQF7rWsL1Othhe9Oh.png)

所以就有了下图，即消息发送方 Alice 和消息接收方 Bob 的处理过程，先签名后加密发送。

![](https://xdo0.github.io/_posts/imgsrc/boxcnl6CM00PyWQBvDTRWBSgrue.png)

图 2 发送消息的处理过程

至于是否可以将签名加密顺序换一下，先加密后签名呢？拿以下两种风险场景举例：

1. 可以看下面的图片，中间人拦截消息并替换签名导致 Bob 以为消息是 Mallory 发送的（前提是 Bob 保存了 Mallory 的验签公钥）。

![](https://xdo0.github.io/_posts/imgsrc/boxcn8apkDG09LUFvPevk37mhre.png)

1. 即使 Bob 只会用 Alice 的公钥验签，但中间人一旦在不可信的信道拦截到消息，也可以通过 Alice 的验签公钥验证出消息是 Alice 发送的。

# 2. PKI 体系

图二保证了消息的机密（明文不被窃取）、完整（消息不被篡改）。但是无法保证可靠（双方身份合法，使用密钥准确）。

- 这就有了 PKI 体系，来验证身份合法性；
- 使用 [Diffie-Hellman 密钥](https%3A%2F%2Fwww.liaoxuefeng.com%2Fwiki%2F1252599548343744%2F1304227905273889)交换算法来传递密钥。

> PKI 体系，即公开密钥基础建设（Public Key Infrastructure，PKI），通过使用**公钥技术和数据证书**来提供信息系统安全服务，并负责**验证数字证书持有者身份**的一种体系。PKI 基础设施采用证书管理公钥，通过第三方可信任认证中心，把用户的**公钥和用户的身份信息捆绑**在一起，它是具有通用性的安全基础设施，是一套服务体系。

## **2.1 PKI 体系架构**

PKI 体系架构由证书申请者、注册机构 RA、认证中心 CA、证书撤销列表 CRL 组成。

1. **CA（Certification Authority）**：负责证书的颁发和吊销（Revoke），接收来自 RA 的请求，是最核心的部分。
2. **RA（Registration Authority）**：对用户身份进行验证，校验数据合法性，负责登记，审核过了就发给 CA。
3. **证书存储库：**存放证书，多采用 X.500 系列标准格式。

常见的操作流程为，用户通过 RA 登记申请证书，提供身份和认证信息等；CA 审核后完成证书的制造，颁发给用户。用户如果需要撤销证书则需要再次向 CA 发出申请。

## 2.2 证书校验

客户端验证服务端下发的证书，主要包括以下几个方面：

- 1、校验证书是否是 `受信任` 的 `CA根证书` 颁发机构颁发；
- 2、校验证书是否在上级证书的 `吊销列表`；
- 3、校验证书 `是否过期`；
- 4、校验证书 `域名` 是否 `一致`。

![](https://xdo0.github.io/_posts/imgsrc/boxcnJ7OXVpH5ILsrUBDaOxonTz.png)

### **CA 机构 颁发证书的基本原理：**

- `服务端` 生成一对 `公钥`、`私钥`。
- `服务端` 将自己的公钥提供给 `CA机构`。
- `CA机构` 核实 `服务端公钥` 拥有者信息： 核实申请者提供信息的真实性：如组织是否存在、企业是否合法、是否拥有域名的所有权等。
- `CA机构` 签发证书： CA 机构 计算 服务器 `公钥摘要信息`，然后利用 `CA机构私钥`（CA 机构有一对公钥、私钥）加密 `摘要信息`。 加密后的包含 `加密摘要` 信息和 `服务端公钥` 即 `CA机构` 颁发的 `证书`。

### **客户端 验证服务端公钥的基本原理为：**

- `客户端` 获取到服务端的证书中的 `公钥`： Https 请求 TLS 握手过程中，服务器公钥会下发到请求的客户端。
- `客户端` 用存储在本地的 `CA机构的公钥`，对 `服务端公钥` 中对应的 `摘要信息` 进行解密，获取到 `服务端公钥` 的 `摘要信息A`；
- `客户端` 根据对 `服务端公钥` 进行摘要计算，得到 `摘要信息B`；
- `对比` 摘要信息 `A与B`，相同则证书验证通过；

## 2.3 证书链校验

实际证书申请中，由于权威的 `CA机构` 数量不多，若所有的 `服务器证书` 都向权威 CA 机构申请，那么 CA 机构的工作量就会非常大。因此 CA 机构采取 `授权` `二级机构` 的方式来管理证书申请，经 `授权` 的 `二级机构` 也可以签发 `服务器证书`。

![](https://xdo0.github.io/_posts/imgsrc/boxcndGWoHP9LwIU6FqVxKXREkh.png)

接下来讲下现实的场景，上面 Bob 和 Alice 的场景只是其中的一环。以 https 的 ssl/tls 为例：

# 3. SSL/TLS

`HTTPS` （Secure Hypertext Transfer Protocol）安全超文本传输协议，是一种通过计算机网络进行安全通信的传输协议。 `HTTPS` 利用 `SSL/TLS` 来加密数据包，经由 `HTTP` 进行通信。

其设计的主要目的是，提供对网站服务器的身份认证、保护交换数据的隐私与完整性。

`TLS` 前身为 SSL，TLS（Transport LayerSecurity） 与 SSL（Secure Socket Layer） 某种程度上指的是同一个概念。

## 3.1 SSL/TLS 握手

总结：用 `非对称加密` 的手段 `传递密钥`，然后 `用密钥` 进行 `对称加密传递` 数据。

简要步骤，这里参考[大佬的文章](https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F52493697%2Fanswer%2F130813213)，以浏览器输入：https://某宝.com 的过程为例，这里浏览器是 `client`，某宝是 `server`

1. 浏览器与某宝建立 TCP 连接
2. 某宝向浏览器下发自己的数字证书
3. Server Key Exchange：某宝向浏览器发送 DH 密钥交换的额外数据
4. 浏览器对证书进行合法性校验
5. 双方会运行 Diffie Hellman 算法。

证书合法性验证通过之后，浏览器再利用证书中的公钥加密传输 DH 算法需要的其他数据，就会协商出 AES 密钥 `enc_key`，注意 `enc_key` 不会在网络上传递。

1. 后续的通信都采用协商的 `enc_key` 和 `加密算法` 进行加密通信

# 4. Fiddler 抓包原理

![](https://xdo0.github.io/_posts/imgsrc/boxcndhZKd7dpwUlLCyFVtiYHOh.png)

### 4.1 SSL Pinning

可以看到 app 很容易遭到中间人攻击，因此可以采用 SSL Pinning：

证书锁定（SSL/TLS Pinning）顾名思义，将服务器提供的 SSL/TLS 证书内置到移动端开发的 APP 客户端中，当客户端发起请求时，通过比对内置的证书和服务器端证书的内容，以确定这个连接的合法性。
