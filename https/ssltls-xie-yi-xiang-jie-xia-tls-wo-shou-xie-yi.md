# SSL/TLS协议详解(下)——TLS握手协议

原文 [https://xz.aliyun.com/news/2213](https://xz.aliyun.com/news/2213)

***

本文翻译自：[https://www.wst.space/ssl-part-4-tls-handshake-protocol/](https://www.wst.space/ssl-part-4-tls-handshake-protocol/)

***

  在博客系列的[第2部分](https://xz.aliyun.com/t/2530)中,对证书颁发机构进行了深入的讨论.在这篇文章中，将会探索整个SSL/TLS握手过程，在此之前，先简述下最后这块内容的关键要点：

> * TLS适用于对称密钥
> * 对称密钥可以通过安全密钥交换算法共享
> * 如果请求被截获，密钥交换可能会被欺骗
> * 使用数字签名进行身份验证
> * 证书颁发机构和信任链。

  在这篇文章中，使用WireShark的工具来查看网络流量，我个人使用的是Linux（Ubuntu 16.04）系统，可以使用以下命令轻松安装WireShark：

```
$sudo apt install wireshark
```

  以sudo的权限打开WireShark并选择提供互联网连接的接口，我这里是eth0，然后点击WireShark右上角的“开始捕获数据包”按钮，Wireshark将立即开始抓取通过机器的所有流量。现在我们从浏览器中加载[github.com](https://github.com/)。Github使用TLS进行所有通信，将重新定向到https并加载。现在关闭浏览器，看看WireShark抓到了什么。

## DNS解析 <a href="#id-755bf308c5c179aefb7c6d213af950a6" id="id-755bf308c5c179aefb7c6d213af950a6"></a>

  这并不是TLS的一部分，但我们在WireShark中看到了它。

<figure><img src="../.gitbook/assets/image (132).png" alt=""><figcaption></figcaption></figure>

我已将Google DNS设置为我的DNS服务器，它的地址是8.8.8.8，在图像中可以看到请求已发送到8.8.8.8查询github.com的A记录，也就是我们想要连接的Github的IP地址。

<figure><img src="../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>

\
  DNS服务器使用github.com的IP响应为192.30.253.113，蓝色表示选择显示相应的部分，现在，浏览器已获取了将要用来连接服务器的目标IP。

## 发起TLS握手 <a href="#id-862c16f432830d6635e41bdd39b8deb6" id="id-862c16f432830d6635e41bdd39b8deb6"></a>

  解析IP后，浏览器将通过http请求页面，如果服务器支持TLS，那么它将发送协议升级请求来响应浏览器，这个新的地址[https://github.com](https://github.com/) ,将使用端口号443来指定，随后浏览器将启动TLS握手请求。大多数现代浏览器都存有与Web服务器的最后一次连接的记录，如果最后一次连接是通过https进行的，那么下次浏览器将自动启动https请求而无需等待服务器。\
TLS握手分为以下几个步骤：

> * 客户端发送Hello报文
> * 服务器接收Hello报文
> * 共享证书和服务器密钥交换
> * 更改密码规范
> * 加密握手

### 客户端发送Hello报文 <a href="#id-458c45458f62555887247aac78bbcb3a" id="id-458c45458f62555887247aac78bbcb3a"></a>

  从这里开始，我将会重点讨论图片中标记为蓝色的主题，Client发送的Hello报文如下图所示。\


<figure><img src="../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

\
  我们知道TLS是在TCP之上实现的协议，TLS本身是一层协议并且它的底层叫做**记录协议(Record protocol)**，这意味着所有数据都被记录。典型的记录格式如下：

```
HH V1:V2 L1:L2 data
```

> * HH是单个字节，表示记录中的数据类型。共定义了四种类型：change\_cipher\_spec（20），alert（21），handshake（22）和application\_data（23）。
> * V1：V2是协议版本，用两个以上的字节表示。对于当前定义的所有版本，V1的值为0x03，而对于SSLv3，V2的值为0x00，对于TLS 1.0为0x01，对于TLS 1.1为0x02，对于TLS 1.2为0x03。
> * L1：L2是数据的长度，以字节为单位（使用big-endian约定：长度为256 \* L1 + L2），数据的总长度不能超过18432字节，但实际上它无法达到这个值。

  在图中，可以看出内容类型是Handshake，TLS版本1.0，数据长度为512.真实数据位于名为 **Handshake Protocol：Client Hello**的下拉列表中。我们继续观察下Client Hello中共享的数据。

#### 客户端发送Hello报文的内容 <a href="#aa90c15d4746feb9e41d8264e2d61038" id="aa90c15d4746feb9e41d8264e2d61038"></a>

  浏览器与服务器共享以下详细信息\


<figure><img src="../.gitbook/assets/image (135).png" alt=""><figcaption></figcaption></figure>

#### 客户端版本 <a href="#id-535ef41011752967597280b4943598a7" id="id-535ef41011752967597280b4943598a7"></a>

  按优先顺序列出的客户端支持的协议版本，首选客户希望支持的最新协议版本。

#### 客户端的随机数 <a href="#id-9c3abd9daeb8688ba61e662772529bc6" id="id-9c3abd9daeb8688ba61e662772529bc6"></a>

  一个32字节的数据，其中前4个字节表示epoch格式的当前日期时间。[纪元时间](https://en.wikipedia.org/wiki/Unix_time)是自1970年1月1日以来的秒数。其余28个字节由加密强随机数生成器生成（例如，Linux中的`/dev/urandom`），客户端随机会在后面用到，请先记住这点。

#### 会话id(Session id) <a href="#id-8ad2c8fde6a04b15beee4a6b9e51acef" id="id-8ad2c8fde6a04b15beee4a6b9e51acef"></a>

  如果客户端第一次连接到服务器，那么这个字段就会保持为空。在上图中，您可以看到Session id正在给服务器发送东西，之所以会发生这种情况是由于我之前是通过https连接到github.com的，在此期间，服务器将使用Session id映射对称密钥，并将Session id存储在客户端浏览器中，为映射设置一个时间限。如果浏览器将来连接到同一台服务器（当然要在时间限到期之前），它将发送Session id，服务器将对映射的Session进行验证，并使用以前用过的对称密钥来恢复Session，这种情况下，就不需要完全握手。

#### 密码套件 <a href="#id-0c250c2fa6b9b420a5d5010b32d091db" id="id-0c250c2fa6b9b420a5d5010b32d091db"></a>

  客户端还将发送自己已经知道的密码套件列表，这个是由客户按优先顺序排列的，但完全由服务器来决定发送与否。TLS中使用的密码套件有一种标准格式。\


<figure><img src="../.gitbook/assets/image (137).png" alt=""><figcaption></figcaption></figure>

\
我们从列表中用一个例子来进行分析.

```
Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 (0xc02b)
```

> * TLS：指使用的协议是TLS
> * ECDHE：密钥交换算法
> * ECDSA：签名或验证算法
> * AES\_128\_GCM：称为批量加密算法。对称密钥加密算法是AES，密钥长度为128位，AES是**块密码**，也就是对输入的纯文本用固定长度的块来进行加密，加密后的每个块按再顺序发送，最后按类似的方式来进行解密。按标准规定，AES块固定长度为128位，但是输入的明文不要求必须是128的倍数，所以我们可能需要对最后一个块中进行填充，使其为固定的长度128位。除此之外，为了提高平均信息量，通常在加密之前会添加一些随机的比特到明文中，我们称为初始化矢量（IV）。有很多算法都可以在块上添加IV实现填充。在我们的例子[Galois/Counter Mode(GCM)](https://en.wikipedia.org/wiki/Galois/Counter_Mode)中用到过。或许详细解释GCM模式会使事情变得复杂，可见这并不是一个好主意。\
>   SHA256：消息验证代码（MAC）算法。我们将详细讨论MAC。

#### 压缩数据 <a href="#id-1dbf785cd85a44bdcdc1469243506518" id="id-1dbf785cd85a44bdcdc1469243506518"></a>

  为了减少带宽，可以进行压缩。但从成功攻击TLS的事例中来看，其中使用压缩时的攻击可以捕获到用HTTP头发送的参数，这个攻击可以劫持Cookie，这个漏洞我们称为[CRIME](https://en.wikipedia.org/wiki/CRIME)。从TLS 1.3开始，协议就禁用了TLS压缩。

#### 扩展名 <a href="#id-2d7e123da9fa3f15aa18ae5f5dc05207" id="id-2d7e123da9fa3f15aa18ae5f5dc05207"></a>

  其他参数（如服务器名称，填充，支持的签名算法等）可以作为扩展名使用，我们可以任意对用作扩展名的内容研究一番。

  这些是客户端问候的一部分，如果已收到客户端问候，接下来就是服务器的确认，服务器将发送服务器问候。\


<figure><img src="../.gitbook/assets/image (138).png" alt=""><figcaption></figcaption></figure>

### 服务器接收Hello报文 <a href="#c2f75c071557b0520bafadfbd9f31e67" id="c2f75c071557b0520bafadfbd9f31e67"></a>

  收到**客户端问候**之后服务器必须发送**服务器问候**信息，服务器会检查指定诸如TLS版本和算法的客户端问候的条件，如果服务器接受并支持所有条件，它将发送其证书以及其他详细信息，否则，服务器将发送握手失败消息。\


<figure><img src="../.gitbook/assets/image (139).png" alt=""><figcaption></figcaption></figure>

\
图中，我们可以看到服务器响应0x0303表示服务器同意使用TLS 1.2，我们来检查一下**服务器问候**中的记录。

#### 服务器接收Hello报文的内容 <a href="#id-702b9f7c424739894a318e53c885044c" id="id-702b9f7c424739894a318e53c885044c"></a>

  服务器问候消息包含以下信息。\


<figure><img src="../.gitbook/assets/image (140).png" alt=""><figcaption></figcaption></figure>

\
加下来我们会在这里讨论其中一些重要的参数。

#### 服务器版本 <a href="#id-0abbb04b60c3e43d1073397c25df4d1d" id="id-0abbb04b60c3e43d1073397c25df4d1d"></a>

  如果客户端可以支持，则服务器将选择客户端指定的TLS版本，这里选择了TLS 1.2

#### 服务器的随机数 <a href="#id-962464d7175d727a3803ba5bd02f3029" id="id-962464d7175d727a3803ba5bd02f3029"></a>

  类似于客户端随机，服务器随机也占32字节，前4个字节表示服务器的Unix纪元时间，后面加上28字节的随机数。客户端和服务器随机将用来创建加密密钥，我待会儿会解释。

#### 密码套件 <a href="#id-92764b65db62abbc271bf449d42904c6" id="id-92764b65db62abbc271bf449d42904c6"></a>

  还记得我们已经将发送支持的密码套件发送到客户端问候中的github.com吗？Github从名单中选出了第一个，也就是：

```
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
```

#### 会话id(Session id) <a href="#e24df9e95f4ce5af56f85e8e3fc93a7e" id="e24df9e95f4ce5af56f85e8e3fc93a7e"></a>

  服务器将约定的Session参数存储在TLS缓存中，并生成与其对应的Session id。它与Server Hello一起发送到客户端。客户端可以写入约定的参数到此Session id，并给定到期时间。客户端将在Client Hello中包含此id。如果客户端在此到期时间之前再次连接到服务器，则服务器可以检查与Session id对应的缓存参数，并重用它们而无需完全握手。这非常有用，因为服务器和客户端都可以节省大量的计算成本。

  在涉及亚马逊和谷歌等流量巨大的应用程序时，这种方法存在缺点。每天都有数百万人连接到服务器，服务器必须使用Session密钥保留所有Session参数的TLS缓存。这是一个巨大的开销。为了解决之前介绍过的Session Tickets的问题, 在这里，客户端可以在client hello中指定它是否支持Session Ticket。然后，服务器将创建一个**新的会话票证(Session Ticket)**，并使用只有服务器知道的经过私钥加密的Session参数。它将存储在客户端上，因此所有Session数据仅存储在客户端计算机上，但Ticket仍然是安全的，因为该密钥只有服务器知道。

  此数据可以作为名为Session Ticket的扩展包含在Client Hello中。在我们的例子中，此参数为空，因为这是第一次连接到github.com或前一个Session的浏览器已过期。

<figure><img src="../.gitbook/assets/image (141).png" alt=""><figcaption></figcaption></figure>

#### 压缩数据 <a href="#id-3b80acce94c89225aa153953b460c078" id="id-3b80acce94c89225aa153953b460c078"></a>

  如果支持，服务器将同意客户端的首选压缩方法。在这里，您可以看到服务器响应为空响应，则意味着不需要压缩。

  服务器不在ServerHello消息中发送任何证书; 它会在正确命名的[证书](https://tools.ietf.org/html/rfc5246#section-7.4.2)消息中发送证书。

#### 服务器证书的信息 <a href="#id-2cd9c36ad40edf9cb8614cdee9032156" id="id-2cd9c36ad40edf9cb8614cdee9032156"></a>

<figure><img src="../.gitbook/assets/image (142).png" alt=""><figcaption></figcaption></figure>

\
  在我们的例子中，证书消息长度为3080字节。毫无疑问，这是包含所有信息的服务器证书。服务器按[信任链](https://www.wst.space/ssl-part-3-certificate-authority/)的顺序发送完整的证书列表。该链中的第一个是服务器证书，接着是颁发服务器证书的intermediate CA 的证书,然后是下一个intermediate CA 的证书......直到Root CA的证书。服务器不可以发送Root CA证书，因为在大多数情况下，浏览器可以从任何intermediate CA 识别Root CA。

  在我们的例子中，您可以看到第一个证书是github.com，第二个证书是中间件Digicert SHA2扩展验证Server CA。 检查下图中的`id-at-commonName`参数。\


<figure><img src="../.gitbook/assets/image (143).png" alt=""><figcaption></figcaption></figure>

\
让我们分析证书的内容，看看浏览器如何验证它。

#### 证书的内容 <a href="#id-075f2470de1d0320ff6a22715de80e1c" id="id-075f2470de1d0320ff6a22715de80e1c"></a>

  证书被发送到浏览器，因此我们可以在访问github.com时查看Github的证书。来自Firefox的CA证书内容:\


<figure><img src="../.gitbook/assets/image (144).png" alt=""><figcaption></figcaption></figure>

\
可以通过单击"**详细信息**"选项卡查看github的intermediate CA 和Root CA.\


<figure><img src="../.gitbook/assets/image (145).png" alt=""><figcaption></figcaption></figure>

\
让我们了解这些领域是什么以及它们的用途。

#### 版本和序列号 <a href="#id-5fa82f12720b853355243835106055e2" id="id-5fa82f12720b853355243835106055e2"></a>

  版本表示使用的是哪个版本的[X.509](https://en.wikipedia.org/wiki/X.509)标准。X.509是用于定义公钥证书格式的标准。[X.509](https://en.wikipedia.org/wiki/X.509)有3个版本，github使用最新版本version 3。

  从RFC 5280开始，CA为每个证书分配的序列号必须是正整数。因此对于每个发布CA证书，它必须是唯一的（即颁发者名称和序列号标识唯一的证书）。所以，CA必须强制serialNumber为非负整数。

#### 证书的签名算法与值 <a href="#cc6af4d175b438b5e4c00b1880c027b5" id="cc6af4d175b438b5e4c00b1880c027b5"></a>

  浏览器需要知道签名算法以验证签名。如果使用的是RSA签名，则需要相同的算法来验证签名。对于Github，使用的是PKCS＃1 SHA-256和RSA加密，即SHA-256用于生成散列，RSA用于签名。

  从我们[上一篇文章](https://xz.aliyun.com/t/2530)中，证书数据使用SHA-256算法进行哈希处理，并使用RSA加密过Github的私钥对此哈希进行签名。

#### 颁布机构 <a href="#id-3d04eb9d080d8120b235c829e53e4939" id="id-3d04eb9d080d8120b235c829e53e4939"></a>

  此字段包含颁发证书的颁发机构的详细信息。Github的证书由Digicert的intermediate CA 颁发。

<figure><img src="../.gitbook/assets/image (146).png" alt=""><figcaption></figcaption></figure>

#### 合法性 <a href="#id-586b66ade6ac3ab8f0848e34a5e679f3" id="id-586b66ade6ac3ab8f0848e34a5e679f3"></a>

  该字段有两个值`Not Before` 和`Not After` 。如果当前日期时间不在这些值之间，则证书无效。浏览器就不会信任该证书。

#### 子公钥信息(Subject Public Key Info) <a href="#id-282e103105632e8d50ef468fb8cce119" id="id-282e103105632e8d50ef468fb8cce119"></a>

  该字段携带公钥和用于生成公钥的算法。此密钥用于交换密钥，我们将在稍后讨论。

#### 指纹 <a href="#id-5129f8b343cd48b0530a1924791898f6" id="id-5129f8b343cd48b0530a1924791898f6"></a>

  浏览器生成了两个指纹SHA 1和SHA-256，而且不会发送到服务器。这些指纹分别是通过SHA 1和SHA-256函数散列DER格式的证书产生的。我们可以通过将证书下载到我们的机器并应用哈希函数来验证这一点。

  单击**详细信息**选项卡左下角的“ **导出**” 按钮以下载证书，保存为.crt 扩展名，并在终端上运行以下命令以生成证书的指纹。

```
$ openssl x509 -noout -fingerprint -sha256 -inform pem -in [certificate-file.crt]

$ openssl x509 -noout -fingerprint -sha1 -inform pem -in [certificate-file.crt]
```

  

<figure><img src="../.gitbook/assets/image (147).png" alt=""><figcaption></figcaption></figure>

这应该产生与您在浏览器中看到的结果相同的结果。这些值不是证书的一部分，而是根据证书计算出来的。Root CA证书的指纹将在浏览器中进行硬编码，因此可以轻松地进行交叉验证。除此之外，这些指纹主要用于识别和组织证书。不要将[Signature与指纹混淆](https://security.stackexchange.com/questions/46230/digital-certificate-signature-and-fingerprint)。

  我们在这里讨论的证书信息是关于github.com的服务器证书。Github的intermediate CA 证书也将在同一请求中发送给客户，所有上述字段也适用于该证书。您可以通过转到详细信息选项卡并单击intermediate CA 来检查，如下所示。

<figure><img src="../.gitbook/assets/image (148).png" alt=""><figcaption></figcaption></figure>

### 服务器端密钥交换 <a href="#e3e28ccbb2ccf5a9609014a1710c80fc" id="e3e28ccbb2ccf5a9609014a1710c80fc"></a>

  随后是**Server Hello**和证书消息(Certificate message)，**服务器密钥交换(Server Key Exchange)**&#x662F;可选的。仅当服务器提供的证书不足以允许客户端交换预主密钥时，才会发送此消息。让我们看看为什么github.com必须发送服务器密钥交换消息。

<figure><img src="../.gitbook/assets/image (149).png" alt=""><figcaption></figcaption></figure>

  我们可以看到github.com首选Session的密码套件是`TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256`。这意味着双方使用`Elliptic Curve Diffie Hellman`算法来交换密钥。在[Diffie-Hellman](https://www.wst.space/ssl-part-2-diffie-hellman-key-exchange/)中，客户端无法自行计算预主密钥; 双方都有助于计算它，因此客户端需要从服务器获取Diffie-Hellman公钥。（不要对"Pre-Master Secret"一词感到困惑，我们将在下面深入讨论它。）当使用`Elliptic Curve Diffie-Hellman`时，该公钥不在证书中。因此，服务器必须在单独的消息中向客户端发送其DH公钥，以便客户端可以计算预主密钥。这可以在上面的图像中看到。请注意，此密钥交换也由签名保护。

  服务器密钥交换完成后，服务器将发送Server Hello Done 消息。客户端将开始计算Pre-Master Secret。我们来看看如何。

#### 如何计算Pre-Master Secret <a href="#id-29542149083a9da4b3b2c20149782f52" id="id-29542149083a9da4b3b2c20149782f52"></a>

  Pre-Master Secret计算取决于商定的密钥交换算法的类型。当使用RSA进行密钥交换时，从客户端（即浏览器）计算[预主密钥](https://security.stackexchange.com/questions/63971/how-is-the-premaster-secret-used-in-tls-generated)，客户端通过连接协议版本（2个字节）和客户端随机生成的一些字节（46个字节）来生成48字节的预主密钥。客户端从加密安全的**伪随机数发生器（PRNG）**&#x83B7;得这46个字节。实际上，这意味着使用操作系统提供的PRNG，例如`/dev/urandom`。然后，使用服务器的公共和共享对此Pre-Master密钥进行加密，以便服务器稍后可以使用它来创建**主密钥**。

  但是，在Github的情况下，如上所述，Diffie-Hellman算法用于密钥交换。这里的情况略有不同。服务器立即生成一对DH私钥 - 公钥。然后，与客户共享公钥。这是如上所述的"服务器密钥交换消息( Server Key Exchange)"。作为响应，客户端还将创建DH密钥对，并通过客户端密钥交换消息与服务器共享公钥，如下所示。

<figure><img src="../.gitbook/assets/image (150).png" alt=""><figcaption></figcaption></figure>

  您可以看到共享的客户端公钥。现在，如果您了解Diffie-Hellman算法的工作原理，您就知道客户端和服务器可以从这些共享公钥到达公共密钥。新生成的密钥称为Pre-Master密钥。

  使用Diffie Hellman算法进行TLS密钥交换具有优势。客户端和服务器都为每个新会话生成一个新密钥对。一旦计算出预主密钥，将立即删除客户端和服务器的私钥。这意味着私钥永远不会被窃取，确保[完美的前向保密](https://en.wikipedia.org/wiki/Perfect_forward_secrecy)。

### 客户端密钥交换 <a href="#b911f29d72a19ee117323127dc31e94a" id="b911f29d72a19ee117323127dc31e94a"></a>

  我们已经在上面讨论过，客户端的DH公钥通过客户端密钥交换消息共享给服务器。但是如果使用RSA，则客户端将如上所述通过其自己计算预主密钥，使用服务器的公钥（RSA公钥）对其进行加密，并通过客户端密钥交换消息将其发送回服务器。然后，服务器可以使用其私钥解密它。无论算法是什么，此时[客户端和服务器都达到了共同的Pre-Master Secert](https://security.stackexchange.com/questions/63971/how-is-the-premaster-secret-used-in-tls-generated) 。完成此操作后，客户端将发送Change Cipher Spec 消息，如下所示。

<figure><img src="../.gitbook/assets/image (151).png" alt=""><figcaption></figcaption></figure>

让我们往下走，看看如何在主密钥从预备主密钥来计算。

#### 如何计算主秘钥 <a href="#id-0e84750bc84b5146a6d0b430732b0421" id="id-0e84750bc84b5146a6d0b430732b0421"></a>

  现在客户端和服务器都有哪些随机数据呢？根据[RFC 5346](https://tools.ietf.org/html/rfc5246#section-8.1)标准，在问候消息期间客户端和服务器共享的预主密钥和随机值（还记得吗？）都会使用PRF（伪随机函数）产生的值来计算主密钥。

```
master_secret = PRF（pre_master_secret，“master secret”，ClientHello.random + ServerHello.random）[0..47];
```

这里，

```
pre_master_secret - 双方计算的48字节Pre-Master密码。

“master secret” - 它只是一个使用ASCII字节的字符串。

ClientHello.random - 客户端hello中共享的随机值

ServerHello.random - 服务器hello中共享的随机值。
```

  主密钥的大小共48个字节，好吧，到目前为止还不是太乱。双方都可以使用主密钥加密数据并来回发送，确实如此，但程序还没结束。你认为[双方使用相同的秘钥](https://crypto.stackexchange.com/questions/2878/separate-read-and-write-keys-in-tls-key-material)是个好办法吗？当然不是！TLS为客户端和服务器分配了单独的密钥，它们都来自主密钥本身，换句话说，主密钥不直接用于加密数据，而是将单独的加密密钥用于客户端和服务器。由于双方都有两个密钥，服务器用其密钥加密的数据可以由客户端轻松解密，反之亦然。

  还没完，TLS还具有用于对称密钥加密的附加安全机制。

#### 消息验证代码（MAC）和TLS数据完整性 <a href="#id-709d71d40ed11b20b1f0d0f4463432ff" id="id-709d71d40ed11b20b1f0d0f4463432ff"></a>

  窃听者可以对传输中的加密数据进行两种可能的攻击：尝试解密数据或尝试修改数据。只要密钥安全，我们就可以认为解密基本上是不可能的，但如果是修改数据呢？客户端和服务器是怎么知道攻击者没有修改过数据呢？如上所述，TLS不仅仅是加密数据，还可以保护数据，使其免受未检测到的修改，换句话说，TLS可以检查数据的完整性。让我们看看它是怎么做到的。

  当服务器或客户端使用主密钥加密数据时，它还会计算明文数据的校验和（哈希值），这个校验和称为**消息验证代码（MAC）**。然后在发送之前将MAC包含在加密数据中。密钥用于从数据中生成MAC，以确保传输过程中攻击者无法从数据中生成相同的MAC，故而MAC被称为HMAC（哈希消息认证码）。另一方面，在接收到消息时，解密器将MAC与明文分开，然后用它的密钥计算明文的校验和，并将其与接收到的MAC进行比较，如果匹配，那我们就可以得出结论：数据在传输过程中没有被篡改。

  客户端和服务器必须使用相同的散列算法来创建以及验证MAC，还记得Github同意的密码套件的最后一部分吗？\
`TLS_ECDHE_ECDSA_WITH_AES_128_GCM_ SHA256`。即SHA256 是用于处理HMAC的哈希函数，为了提高安全性，客户端和服务器使用MAC密钥。让我们看看这些是什么。

#### MAC密钥和IV密钥 <a href="#id-71510de1f18bee646142352ed6c049f9" id="id-71510de1f18bee646142352ed6c049f9"></a>

  根据要求，有4个密钥用于加密和验证每个消息的完整性，他们是：

> * 客户端写入加密密钥：客户端用赖加密数据，服务器用来解密数据。
> * 服务器写入加密密钥：服务器用来加密数据，客户端用来解密数据。
> * 客户端写入MAC密钥：客户端用来创建MAC，服务器用来验证MAC。
> * 服务器写入MAC密钥：服务器用来创建MAC，客户端用来验证MAC。

  这些密钥块由主密钥上的相同的PRF反复地生成，直至密钥有了足够的字节。

```
key_block = PRF（SecurityParameters.master_secret，“密钥扩展”，SecurityParameters.server_random + SecurityParameters.client_random）;
```

  如您所见，除了客户端 - 服务器随机值和字符串“密钥扩展”之外，主密钥还用来增加密钥的平均信息量。PRF可以生成任意长度的密钥，这点是很有用的，因为默认情况下不同的散列函数具有不同的长度。在我们的例子中用的是SHA256，它是256位，但MD5的默认长度为128位。

  除此之外，我们知道我们使用的AES和GCM算法是一种分组密码，它需要一组比特来作为初始化向量（IV）。在讨论密码套件时，我们已经提到IV用于改善AES加密的平均信息量，换句话说，当多次加密同一文件时，IV能够生成不同的密文，这些随机的字节也由相同的PRF生成，并且被称为客户端写入IV 和服务器写入IV ，术语是自解释的。我不会对IV的细节再进行更多讲解，因为它是一个很大的主题，超出了本文的范围。

#### 生成测试数据 <a href="#id-32844cd21657bc4ad4e8d7de2552853a" id="id-32844cd21657bc4ad4e8d7de2552853a"></a>

  现在双方都有了加密密钥，我们准备加密，但是在将TLS放到应用层之前，我们需要像每个进程一样来测试并验证客户端加密数据是否可以由服务器解密，反之亦然。为此，客户端将使用伪随机函数（PRF）计算12字节的[verify\_data](https://crypto.stackexchange.com/questions/34754/what-does-the-tls-1-2-client-finished-message-contain/34792)，如下所示。

```
verify_data = PRF(master_secret, "client finished", MD5(handshake_messages) + SHA-1(handshake_messages) ) [12]
```

  其中handshake\_messages 是所有握手消息的缓冲区，以上版本适用于版本1.2的TLS。版本1.2略有变化，即verify\_data的长度取决于密码套件而不总是12字节，任何未明确指定verify\_data\_length的密码套件都等于12。此外，伪随机函数（PRF）中的MD5 / SHA-1组合具有已被密码套件指定的PRF替换。所以根据最新规范，

```
Verify_data = PRF(master_secret, finished_label, Hash(handshake_messages)) [0..verify_data_length-1];
```

  因此我们有测试数据，用密钥和算法来加密测试数据。客户端所需要做的就是用客户端加密密钥（或简称客户端写入密钥）使用AES算法加密测试数据，如上所述还得计算HMAC，客户端获取结果并添加记录头字节“0x14”表明“已完成”，再通过客户端生成消息并且发送到服务器。这是由实体和客户端发送的最后一次握手消息之间协商的算法和密钥保护的第一条消息。由于消息是完全加密的，因此WireShark只会看到加密的内容，并通过名称为加密握手的消息来调用完成的握手信息，如下所示。

<figure><img src="../.gitbook/assets/image (152).png" alt=""><figcaption></figcaption></figure>

#### 验证磋商 <a href="#b6d5bcdcaac49e038840d3f64174af23" id="b6d5bcdcaac49e038840d3f64174af23"></a>

  服务器处理过程也几乎相同。它发出一个Change Cipher Spec ，然后发送一条包含所有握手消息的**已完成信息**。更改标记在该服务器切换到新协商的加密套件和键点的密码SPEC消息，然后再加密后续客户端的记录。除此之外，服务器的**完成消息**将包含对客户端的**完成消息**进行解密的版本，一旦客户端收到此数据，它将使用服务器写入密钥对其进行解密。故而这就向客户证明了服务器能够成功解密我们的消息。KABOOM！我们完成了TLS握手。

  所有的加密都是基于协商的算法。在我们的例子中，算法是AES\_128\_GCM，这里没有必要进行进一步的解释，因为当涉及到其他网站时，服务器指定的算法可能会有所不同。如果您有兴趣了解这些算法的工作原理，[维基百科有一个列表](https://en.wikipedia.org/wiki/Cipher_suite#Supported_algorithms)。我也是通过TLS基础知识来学习密码学。

## 加密应用程序数据 <a href="#id-027eb5989a4872addcf7c947b561732b" id="id-027eb5989a4872addcf7c947b561732b"></a>

  我们现在在[应用层](https://en.wikipedia.org/wiki/OSI_model#Layer_7:_Application_Layer)。如果你有一个中等速度的互联网连接，我们只是连接了几百毫秒。想象一下，在如此短的时间内会发生多少事情？

我要求的页面是homepade aka www.github.com

。所以在Mozilla Firefox的开发者工具中显示的纯文本请求是，

```
GET https://github.com/

Host: github.com

User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:61.0) Gecko/20100101 Firefox/61.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate, br

Connection: keep-alive

Upgrade-Insecure-Requests: 1

Cache-Control: max-age=0
```

请参阅下图：

<figure><img src="../.gitbook/assets/image (153).png" alt=""><figcaption></figcaption></figure>

前3个字节17 03 03 表示内容的数据类型（应用程序数据）和TLS版本（TLS 1.2）。

## 尾声 <a href="#id-80d9a6111f8c7d22577b9942305e84f4" id="id-80d9a6111f8c7d22577b9942305e84f4"></a>

对，就是这样。我们结束了。

<figure><img src="../.gitbook/assets/image (154).png" alt=""><figcaption></figcaption></figure>

  在本系列的下一部分中，我会添加一些在本文中无法包含的别的内容。我还发布了结构化的参考链接，这对于学习TLS中的密码学还是很有用的。\
  我想在这里再写点什么。整篇文章写的都是我对TLS的理解上的兴趣，就是说我所学到/理解的一切都在这里了，或许并不完整，或许会有错误，或许各位的看法和我有出入。总之，不管是什么，欢迎您在评论区中分享，我很高兴能和诸位一起学习更多的东西！
