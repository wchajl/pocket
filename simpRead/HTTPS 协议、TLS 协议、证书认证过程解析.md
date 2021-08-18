> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/snowater/p/7804889.html)

### 一、HTTPS 协议

HTTPS 协议其实就是 HTTP over TSL，TSL(Transport Layer Security) 传输层安全协议是 https 协议的核心。

TSL 可以理解为 SSL (Secure Socket Layer) 安全套接字层的后续版本。

TSL 握手协议如下图所示

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171108204533294-1339745312.png)

（注：图片来源于 google 图片）

在建立 TCP 连接后，开始建立 TLS 连接。下面抓包分析 TLS 握手过程，抓包图片来源于[传输层安全协议抓包分析之 SSL/TLS](http://www.freebuf.com/articles/network/116497.html) (自己没抓到这么完整的包，只能搬运过来了，摔)

(1) client 端发起握手请求，会向服务器发送一个 ClientHello 消息，该消息包括其所支持的 SSL/TLS 版本、Cipher Suite 加密算法列表（告知服务器自己支持哪些加密算法）、sessionID、随机数等内容。

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171108210913731-381290333.png)

(2) 服务器收到请求后会向 client 端发送 ServerHello 消息，其中包括：

SSL/TLS 版本；

session ID，因为是首次连接会新生成一个 session id 发给 client；

Cipher Suite，sever 端从 Client Hello 消息中的 Cipher Suite 加密算法列表中选择使用的加密算法；

Radmon 随机数。

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171108211404888-25605470.png)

(3) 经过 ServerHello 消息确定 TLS 协议版本和选择加密算法之后，就可以开始发送证书给 client 端了。证书中包含公钥、签名、证书机构等信息。

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171108212037372-893214258.png)

(4) 服务器向 client 发送 ServerKeyExchange 消息，消息中包含了服务器这边的 EC Diffie-Hellman 算法相关参数。此消息一般只在选择使用 DHE 和 DH_anon 等加密算法组合时才会由服务器发出。

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171108212528247-743481578.png) 

(5) server 端发送 ServerHelloDone 消息，表明服务器端握手消息已经发送完成了。

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171108212652544-470476594.png)

(6) client 端收到 server 发来的证书，会去验证证书，当认为证书可信之后，会向 server 发送 ClientKeyExchange 消息，消息中包含客户端这边的 EC Diffie-Hellman 算法相关参数，然后服务器和客户端都可根据接收到的对方参数和自身参数运算出 Premaster secret，为生成会话密钥做准备。

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171108212742997-638983083.png)

(7) 此时 client 端和 server 端都可以根据之前通信内容计算出 Master Secret（加密传输所使用的对称加密秘钥），client 端通过发送此消息告知 server 端开始使用加密方式发送消息。

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171108213240841-1040450792.png)

(8) 客户端使用之前握手过程中获得的服务器随机数、客户端随机数、Premaster secret 计算生成会话密钥 master secret，然后使用该会话密钥加密之前所有收发握手消息的 Hash 和 MAC 值，发送给服务器，以验证加密通信是否可用。服务器将使用相同的方法生成相同的会话密钥以解密此消息，校验其中的 Hash 和 MAC 值。

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171108213303466-1978167763.png)

(9) 服务器发送 ChangeCipherSpec 消息，通知客户端此消息以后服务器会以加密方式发送数据。

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171108213315263-1721933592.png)

(10) sever 端使用会话密钥加密（生成方式与客户端相同，使用握手过程中获得的服务器随机数、客户端随机数、Premaster secret 计算生成）之前所有收发握手消息的 Hash 和 MAC 值，发送给客户端去校验。若客户端服务器都校验成功，握手阶段完成，双方将按照 SSL 记录协议的规范使用协商生成的会话密钥加密发送数据。

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171108213325309-1347694901.png)

### 二、session ID 的复用

根据 [rfc5246](https://tools.ietf.org/html/rfc5246)，client 和 server 建立 TLS 握手过程如下所示：

```
      Client                                               Server

      ClientHello                  -------->
                                                      ServerHello
                                                     Certificate*
                                               ServerKeyExchange*
                                              CertificateRequest*
                                   <--------      ServerHelloDone
      Certificate*
      ClientKeyExchange
      CertificateVerify*
      [ChangeCipherSpec]
      Finished                     -------->
                                               [ChangeCipherSpec]
                                   <--------             Finished
      Application Data             <------->     Application Data

             Figure 1.  Message flow for a full handshake

```

```
* Indicates optional or situation-dependent messages that are not
   always sent.

```

client 在向 server 发送 ClientHello 消息的时候，会传送 Session ID 给 server 端，server 端收到 session Id 后会去 session 缓存中查找是否有相同值。如果找到相同值，则 server 直接发送一个具有相同 session ID 的 ServerHello 消息给 client 端（此时不必新建 Session ID），然后双方各发一次 ChangeCipherSpec 消息后直接进入 Finished 消息互发阶段，具体如下图所示：

```
      Client                                                Server

      ClientHello                   -------->
                                                       ServerHello
                                                [ChangeCipherSpec]
                                    <--------             Finished
      [ChangeCipherSpec]
      Finished                      -------->
      Application Data              <------->     Application Data

          Figure 2.  Message flow for an abbreviated handshake

```

client 和 server 通过缓存 Session ID 可以快速建立 TLS 握手，但是这么做也有一些弊端，例如：1）负载均衡中，多机之间往往没有同步 Session 信息，如果客户端两次请求没有落在同一台机器上就无法找到匹配的信息；2）服务端存储 Session ID 对应的信息不好控制失效时间，太短起不到作用，太长又占用服务端大量资源。而 Session Ticket（会话记录单）可以解决这些问题，Session Ticket 是用只有服务端知道的安全密钥加密过的会话信息，最终保存在浏览器端。浏览器如果在 ClientHello 时带上了 Session Ticket，只要服务器能成功解密就可以完成快速握手。

### 三、数字证书的验证

client 端收到 server 端发过来的证书，首先必须要做的事验证证书是否可信。如何验证证书是否可信呢？为了解决这个问题，我们先来了解下证书的组成

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171108221111716-1100586299.png)

从这里面我们能看到证书包含以下内容：

(1) Validity 也即有效期，有效期包含生效时间和失效时间，是一个时间区间；

(2) 公钥信息 Subject Public Key Info，包括公钥的加密算法和公钥内容；

(3) Fingerprints 信息，fingerprints 用于验证证书的完整性，也就是说确保证书没有被修改过。 其原理就是在发布证书时，发布者根据指纹算法 (此处证书使用了 SHA-1 和 SHA-256 算法) 计算整个证书的 hash 值 (指纹) 并和证书放在一起，client 在打开证书时，自己也根据指纹算法计算一下证书的 hash 值(指纹)，如果和刚开始的值相同，则说明证书未被修改过；如果 hash 值不一致，则表明证书内容被篡改过；

(4) 证书的签名 Certificate Signature Value 和 Certificate Signature Algorithm，对证书签名所使用的 Hash 算法和 Hash 值；

(5) 签发该证书的 CA 机构 Issuer；

(6) 该证书是签发给哪个组织 / 公司信息 Subject；

(7) 证书版本 Version、证书序列号 Serial Number 以及 Extensions 扩展信息等。

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171109201108247-971868063.png)

如上图所示，为签名过程和 client 端验证过程。中间方框为一个数字证书，在制作数字证书时，会将证书中的部分内容进行一次 Hash 得到一个 Hash 值，然后证书认证机构简称 CA 会使用私钥将该 Hash 值加密为 Certificate Signature。

当证书发送给 Client 端之后，Client 端首先会使用同样的 Hash 算法获得一个证书的 hash 值 H1。通常浏览器和操作系统中集成了 CA 机构的公钥信息，浏览器收到证书后可以使用这些公钥解密 Certificate Signature 内容，得到一个 hash 值 H2。

比较 H1 和 H2，如果值相同，则为可信赖的证书，否则则认为证书不可信。

自己通过 openssl 生成的自签发证书只是使用自己的私钥去加密上图左侧计算出的 Hash Value，这个时候 client 端得到 server 端发过来的证书之后，仍然会尝试使用浏览器或系统内置的 CA 机构的公钥去解密，解密出来的 hash 值 H2 当然不可能与 H1 相同，因此浏览器认为该证书不受信任。但是如果我们选择相信该证书并且继续访问该 web，访问并不会出现任何问题，这是因为证书中的公钥并未加密，使用该公钥也确实能和 server 端的私钥进行 TLS 握手。

### 四、证书信任链

在证书认证过程中还存在一个证书信任链的问题，因为我们从 CA 机构申请到的证书基本不可能是根证书签发。还是以百度的证书为例，如下图所示，百度证书层级为三层，认证过程如下：

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171109202015216-514052730.png)

(1) 当 client 端访问 baidu.com 的时候，baidu 的 server 会将 baidu.com 证书发送给 client 端。

(2) client 端的操作系统或者浏览器中内置了根证书，但是 client 端收到 baidu.com 这个证书后，发现这个证书不是根证书签发，无法根据本地已有的根证书中的公钥去验证 baidu.com 证书是否可信。

于是 client 端根据 baidu.com 证书中的 Issuer 找到该证书的颁发机构 GlobalSign Organization Validation CA - SHA256 - G2，去 CA 请求 baidu.com 证书的颁发机构 GlobalSign Organization Validation CA - SHA256 - G2 的证书。

(3) 请求到证书后发现 GlobalSign Organization Validation CA - SHA256 - G2 证书是由根证书签发，而本地刚好有根证书，于是可以利用根证书中的公钥去验证（验证方法见上一节）GlobalSign Organization Validation CA - SHA256 - G2 证书，发现验证通过，于是信任 GlobalSign Organization Validation CA - SHA256 - G2 证书。

(4) GlobalSign Organization Validation CA - SHA256 - G2 证书被信任后，可以使用 GlobalSign Organization Validation CA - SHA256 - G2 证书中的公钥去验证 baidu.com 证书的可信性。验证通过，于是信任 baidu.com 证书。

在这四个步骤中，最开始 client 端只信任根证书 GlobalSign Root CA 证书的，然后 GlobalSign Root CA 证书信任 GlobalSign Organization Validation CA - SHA256 - G2 证书，而 GlobalSign Organization Validation CA - SHA256 - G2 证书又信任 baidu.com 证书，于是 client 端也信任 baidu.com 证书。这样的一个过程就构成了一条信任链路，整个证书信任链验证流程如下图所示。

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171109204708278-1042248741.png)

### 五、非对称加解密

非对称加密包含一个密钥对：公钥和私钥。公钥可以公开，私钥必须安全保存。

![](https://images2017.cnblogs.com/blog/1244495/201711/1244495-20171109093358122-1780177389.png)

如上图所示，数据可以被公钥加密，加密后的数据只有持有私钥才能进行解密。同理私钥加密的数据，也只有对应的公钥才能解密。

建立 HTTPS 连接以后，client（浏览器）已经获得 server 段的公钥，并且经过 TLS 协议在握手过程中协商出一个只有双方知道的**对称密钥**，在后续的数据传输过程中都将使用该密钥进行数据加密传输。因为公钥是公开的，可以下发给所有 client 端的，这个时候即使其他拥有公钥的 client 端截获到 server 端发给 client 端的消息，也无法仅仅凭借公钥去解密这个消息，因为这个消息加密密钥是由 client 和 server 通过相同算法计算出来，并且添加了随机数，理论上这个对称密钥只有 client 和 server 才能知道。

参考文档：

https://tools.ietf.org/html/rfc5246#page-6

http://www.freebuf.com/articles/network/116497.html

https://imququ.com/post/optimize-tls-handshake.html

http://blog.csdn.net/wzzvictory/article/details/9015155