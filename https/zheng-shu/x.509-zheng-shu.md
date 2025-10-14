# X.509 证书

## X.509 证书 <a href="#x509-zheng-shu" id="x509-zheng-shu"></a>

X.509 是密码学里公钥证书的格式标准。是 ITU-T 标准化部门基于他们之前的 ASN.1 定义的一套证书标准。

X.509 证书里含有公钥、身份信息（比如网络主机名，组织的名称或个体名称等）和签名信息（可以是证书签发机构CA的签名，也可以是自签名）。X.509 还附带了证书吊销列表和用于从最终对证书进行签名的证书签发机构直到最终可信点为止的证书合法性验证算法。

X.509 最早与 X.500 一起发布于1988年7月3日。它假设有一套严格的层次化的证书颁发机构（CA）。X.500系统仅由主权国家实施，以实现国家身份信息共享条约的实施目的；而IETF的公钥基础设施（X.509）简称 PKIX 工作组将该标准制定成适用于更灵活的互联网组织。而且事实上 X.509 认证指的是 RFC5280 里定义的 X.509 v3，包括对IETF 的 PKIX 证书和证书吊销列表，通常也称为公钥基础设施（PKI）。

在 X.509 里，组织机构通过发起证书签名请求（CSR）来得到一份签名的证书。首先需要生成一对密钥对，然后用其中的私钥对CSR进行数字签署（签名），并安全地保存私钥。CSR进而包含有请求发起者的身份信息、用来对此请求进行验真的的公钥以及所请求证书专有名称。CSR里还可能带有CA要求的其它有关身份证明的信息。然后CA对这个CSR进行签名。 组织机构可以把受信的根证书分发给所有的成员，这样就可以使用公司的PKI系统了。浏览器（如Firefox）或操作系统预装有可信任的根证书列表，所以主流CA发布的TLS证书都直接可以正常使用。浏览器的开发者直接影响着它的用户对CA的信任。X.509也定义了CRL实现标准。另一种检查合法性的方式是OCSP。

X.509 证书已应用在包括 TLS/SSL 在内的众多网络协议里，同时它也用在很多非在线应用场景里，比如电子签名服务。

## 证书组成结构 <a href="#zheng-shu-zu-cheng-jie-gou" id="zheng-shu-zu-cheng-jie-gou"></a>

证书组成结构标准用 ASN.1（一种标准的语言）来进行描述. X.509 v3 数字证书结构如下：

> * 证书/signedCertificate
>   * 版本号/version
>   * 序列号/serialNumber
>   * 签名算法/signature
>   * 颁发者/issuer
>   * 证书有效期/validity
>     * 此日期前无效/notBefore
>     * 此日期后无效/notAfter
>   * 证书主体/subject
>   * 主体公钥信息/subjectPublicKeyInfo
>     * 公钥算法/algorithm
>     * 主题公钥/subjectPublicKey
>   * 颁发者唯一身份信息（可选项）/issuerUniqueID
>   * 主题唯一身份信息（可选项）/subjectUniqueID
>   * 扩展信息（可选项）/extensions
>   * 证书签名算法/algorithmIdentifier
>   * 数字签名/encrypted

字段解释：

* **版本号**：RFC 1422 给出了v1的证书结构, ITU-T 在v2里增加了颁发者和主题唯一标识符，从而可以在一段时间后可以重用。重用的一个例子是当一个CA破产了，它的名称也在公共列表里清除掉了，一段时间之后另一个CA可以用相同的名称来注册，即使它与之前的并没有任何瓜葛。不过 IETF 并不建议重用同名注册。另外v2在 Internet 也没有多大范围的使用。 v3 引入了扩展。CA 使用扩展来发布一份特定使用目的的证书（比如说仅用于代码签名） 所有的版本中。
* **序列号**: 标识证书的唯一整数，由证书颁发者分配的本证书的唯一标识符，同一个CA颁发的证书序列号都必须是唯一的。在一开始，序列号只要是正整数即可，是每个CA用来唯一标识其所签发的证书。但是在出现了针对证书签名的预选前缀攻击之后，序列号增加了更多的要求来防止此类攻击；现在序列号需要是无序的（无法被预测）而且至少包括20位的熵。
* **签名算法**: 用于签证书的算法标识，由对象标识符加上相关的参数组成，用于说明本证书所用的数字签名算法。例如，SHA-1和RSA的对象标识符就用来说明该数字签名是利用RSA对SHA-1杂凑加密。(sha256WithRSAEncryption)
* **颁发者**: 证书颁发者的可识别名（DN）。
* **证书有效期**: 证书有效期的时间段。本字段由 ”Not Before” 和 ”Not After” 两项组成，它们分别由 UTC 时间或一般的时间表示（在RFC2459中有详细的时间表示规则）。
* **主题**: 证书拥有者的可识别名，这个字段必须是非空的，除非你在证书扩展中有别名。
* **主题公钥信息**: 主题的公钥（以及算法标识符）。
* **颁发者唯一标识符**: 标识符—证书颁发者的唯一标识符，仅在版本2和版本3中有要求，属于可选项。
* **主题唯一标识符**: 证书拥有者的唯一标识符，仅在版本2和版本3中有要求，属于可选项。
* **扩展信息**: X.509证书扩展部分，可选的标准和专用的扩展（仅在版本2和版本3中使用），扩展部分的元素都有这样的结构：
  * extnID：表示一个扩展元素的OID（必选）
  * critical：表示这个扩展元素是否极重要 （可选）
  * extnValue：表示这个扩展元素的值，字符串类型。（可选）
  * 发行者密钥标识符: 证书所含密钥的唯一标识符，用来区分同一证书拥有者的多对密钥。
  * 密钥使用: 一个比特串，指明（限定）证书的公钥可以完成的功能或服务，如：证书签名、数据加密等。如果某一证书将 KeyUsage 扩展标记为“极重要”，而且设置为“keyCertSign”，则在 SSL 通信期间该证书出现时将被拒绝，因为该证书扩展表示相关私钥应只用于签写证书，而不应该用于 SSL。
  * CRL分布点: 指明CRL的分布地点。
  * 私钥的使用期: 指明证书中与公钥相联系的私钥的使用期限，它也有Not Before和Not After组成。若此项不存在时，公私钥的使用期是一样的。
  * 证书策略: 由对象标识符和限定符组成，这些对象标识符说明证书的颁发和使用策略有关。
  * 策略映射: 表明两个CA域之间的一个或多个策略对象标识符的等价关系，仅在CA证书里存在。
  * 主体别名: 指出证书拥有者的别名，如电子邮件地址、IP地址等，别名是和DN绑定在一起的。
  * 颁发者别名: 指出证书颁发者的别名，如电子邮件地址、IP地址等，但颁发者的DN必须出现在证书的颁发者字段。
  * 主体目录属性: 指出证书拥有者的一系列属性。可以使用这一项来传递访问控制信息。

## 证书类型与示例 <a href="#zheng-shu-lei-xing-yu-shi-li" id="zheng-shu-lei-xing-yu-shi-li"></a>

证书信任链中主要包含一下三种类型的证书：

* 根证书
* 中间件证书
* 终端证书

下面是 GlobalSign 颁发的用于 wikipedia.org 以及一些其它 Wikipedia 网站 X.509 证书。证书颁发者填在颁发者(Issuer)字段，主题内容里是组织机构 Wikipedia 的描述，主题备用名称是那些采用该证书的服务器的主机名。主题公钥里的信息表明采用的是椭圆曲线公共密钥，位于最后的签名算法表示它是由 GlobalSign 用其私钥并采用带RSA加密的SHA-256算法进行签名的。

### 最终实体证书（或者叫叶子证书）

```
  认证:
      版本: 3 (0x2)
      序号: 10:e6:fc:62:b7:41:8a:d5:00:5e:45:b6
      签名算法: sha256WithRSAEncryption
      发行者: C=BE, O=GlobalSign nv-sa, CN=GlobalSign Organization Validation CA - SHA256 - G2
      有效期开始时间: Nov 21 08:00:00 2016 GMT
      有效期结束时间: Nov 22 07:59:59 2017 GMT
      主体：C=US, ST=California, L=San Francisco, O=Wikimedia Foundation, Inc., CN=*.wikipedia.org
      主体公钥信息(subject public key info):
            公钥算法: id-ecPublicKey
         256位的公钥:
                    04:c9:22:69:31:8a:d6:6c:ea:da:c3:7f:2c:ac:a5:
                    af：c0：02：ea：81：cb：65：b9：fd：0c：6d：46：5b：c9：1e：
                    ed:b2:ac:2a:1b:4a:ec:80:7b:e7:1a:51:e0:df:f7:
                    c7:4a:20:7b:91:4b:20:07:21:ce:cf:68:65:8c:c6:
                    9d:3b:ef:d5:c1
          ASN1 OID：prime256v1
          NIST 曲线：P-256
       额外资讯(extension):
          密钥使用: 
               敏感讯息(critical):是
               公钥用途:数位签章，密钥协商Key Agreement
       授权相关讯息: 
               敏感讯息(critical):否
               颁发者URI:http://secure.globalsign.com/cacert/gsorganizationvalsha2g2r1.crt
               线上凭证状态协定(OCSP)URI:http://ocsp2.globalsign.com/gsorganizationvalsha2g2
         证书原则(Certificate Policies): 
               敏感讯息(critical):否
          策略 ID#1：1.3.6.1.4.1.4146.1.20
           CPS URI：https://www.globalsign.com/repository/
          策略 ID#2：2.23.140.1.2.2
          基本限制: 
                CA：错误
       凭证撤销中心(X509v3 CRL Distribution Points): 
               敏感讯息(critical):否
               URI：http://crl.globalsign.com/gs/gsorganizationvalsha2g2.crl
       主体备用名称: 
               敏感讯息(critical):否
               DNS:*.wikipedia.org, DNS:*.m.mediawiki.org, DNS:*.m.wikibooks.org, DNS:*.m.wikidata.org, DNS:*.m.wikimedia.org, DNS: *.m.wikimediafoundation.org, DNS:*.m.wikinews.org, DNS:*.m.wikipedia.org, DNS:*.m.wikiquote.org, DNS:*.m.wikisource.org, DNS: *.m.wikiversity.org，DNS：*.m.wikivoyage.org，DNS：*.m.wiktionary.org，DNS：*.mediawiki.org，DNS：*.planet.wikimedia.org，DNS：*。 wikibooks.org, DNS:*.wikidata.org, DNS:*.wikimedia.org, DNS:*.wikimediafoundation.org, DNS:*.wikinews.org, DNS:*.wikiquote.org, DNS:*.wikisource。 org, DNS:*.wikiversity.org, DNS:*.wikivoyage.org, DNS:*.wiktionary.org, DNS:*.wmfusercontent.org, DNS:*.zero.wikipedia.org, DNS:mediawiki.org, DNS:w.wiki, DNS:wikibooks.org, DNS:wikidata.org, DNS:wikimedia.org, DNS:wikimediafoundation.org, DNS:wikinews.org, DNS:wikiquote.org, DNS:wikisource.org, DNS: wikiversity.org，DNS：wikivoyage。org, DNS:wiktionary.org, DNS:wmfusercontent.org, DNS:wikipedia.org
    (在额外讯息中的)密钥使用目的:
               敏感讯息(critical):否
              目的1:TLS Web伺服器鉴定
              目的2:TLS Web客户端鉴定
    主体密钥识别代码(Subject Key Identifier): 
               敏感讯息(critical):否
               密钥id: 28:2A:26:2A:57:8B:3B:CE:B4:D6:AB:54:EF:D7:38:21:2C:49:5C:36
    授权密钥识别代码(X509v3 Authority Key Identifier): 
               敏感讯息(critical):否
               密钥id:96:DE:61:F1:BD:1C:16:29:53:1C:C0:CC:7D:3B:83:00:40:E6:1A:7C

        签章算法: sha256WithRSAEncryption
             数位签章: 8b:c3:ed:d1:9d:39:6f:af:40:72:bd:1e:18:5e:30:54:23:35:
         ...
```

要验证这个证书，我们需要一个跟该证书颁发者及授权密钥标识符

颁发者 C=BE，O=GlobalSign nv-sa，CN=GlobalSign 组织验证 CA - SHA256 - G2 授权密钥标识符 96:DE:61:F1:BD:1C:16:29:53:1C:C0:CC:7D:3B:83:00:40:E6:1A:7C 都匹配的中间证书

配置正确的服务器可以在TLS连接创建的握手阶段同时提供其中间证书。但是也有可能需要根据证书里颁发者的URL去获取中间证书。

### 中间证书 <a href="#zhong-jian-zheng-shu" id="zhong-jian-zheng-shu"></a>

下面是证书颁发机构的证书示例。该证书是由下例根证书签名的用于颁发上例最终实体证书的证书。当然它的主题标识符跟上例证书的授权密钥标识符是相匹配的。

```
 证书:
          版本: 3 (0x2)
        序列号: 04:00:00:00:00:01:44:4e:f0:42:47
      签名算法: sha256WithRSAEncryption
        颁发者: C=BE, O=GlobalSign nv-sa, OU=Root CA, CN=GlobalSign Root CA
      此前无效: Feb 20 10:00:00 2014 GMT
      此后无效: Feb 20 10:00:00 2024 GMT
         主题: C=BE, O=GlobalSign nv-sa, CN=GlobalSign Organization Validation CA - SHA256 - G2
  主题公钥信息:
            公钥算法: rsaEncryption
        2048位的公钥:
                    00:c7:0e:6c:3f:23:93:7f:cc:70:a5:9d:20:c3:0e:
                    ...
               指数: 65537 (0x10001)
  X509 v3扩展:
   X509v3 密钥使用: 
               关键:是
               用于:证书签名, CRL签名
          基本约束:
               关键:是
        证书颁发机构:是
        路径长度限制:0
    主题密钥标识符: 
               关键:否
                密钥: 96:DE:61:F1:BD:1C:16:29:53:1C:C0:CC:7D:3B:83:00:40:E6:1A:7C
                96:DE:61:F1:BD:1C:16:29:53:1C:C0:CC:7D:3B:83:00:40:E6:1A:7C
        证书策略:
               关键:否
               策略1: 任何策略标识符
            CPS URI：https://www.globalsign.com/repository/

     CRL 分发点:
               关键:否
                URI：http://crl.globalsign.net/root.crl

       授权相关信息: 
               关键:否
  在线证书状态协议(OCSP)URI:http://ocsp.globalsign.com/rootr1

     授权密钥标识符:
               关键:否
                    密钥:60:7B:66:1A:45:0D:97:CA:89:50:2F:7D:04:CD:34:A8:FF:FC:FD:4B

        签名算法: sha256WithRSAEncryption
             数字签名:46:2a:ee:5e:bd:ae:01:60:37:31:11:86:71:74:b6:46:49:c8:
         ...
```

### 根证书 <a href="#gen-zheng-shu" id="gen-zheng-shu"></a>

下面是证书颁发机构的自签名根证书。它的颁发者和主题是相同的，可以用自身的公钥进行合法认证。证书认证过程也将在此终止。如果应用已经在它的可信公钥存贮里已经含有该公钥证书，那么TLS连接时的那个最终实体证书是可信的，否则就是不可信的。

```
 证书:
    版本: 3 (0x2)
    序列号: 04:00:00:00:00:01:15:4b:5a:c3:94
    签名算法：sha1WithRSAEncryption
        发行人：C = BE，O = GlobalSign nv-sa，OU = 根 CA，CN = GlobalSign 根 CA
        有效性
            之前无效：9 月 1 日 12:00:00 1998
            之后无效：1 月 28 日 12:00:00 2028 GMT
        主题：C = BE，O = GlobalSign nv-sa，OU = 根 CA，CN = GlobalSign 根 CA
        主题公钥信息：
            公钥算法：rsaEncryption
                公钥：（2048 位）
                模数：
                    00:da:0e:e6:99:8d:ce:a3:e3:4f:8a:7e:fb:f1:8b:
                    ...
                指数：65537 (0x10001)
        X509v3 扩展：
            X509v3 密钥用法：关键
                证书签名、CRL 签名
            X509v3 基本约束：关键
                CA:TRUE
            X509v3 主题密钥标识符： 
                60:7B:66:1A:45:0D:97:CA:89:50:2F:7D:04:CD:34:A8:FF:FC:FD:4B
    签名算法：sha1WithRSAEncryption
         d6:73:e7:7c:4f:76:d0:8d:bf:ec:ba:a2:be:34:c5:28:32:b5:
         ...
```





## References

{% embed url="https://www.zhaowenyu.com/https-doc/pki/x509.html" %}

{% embed url="https://zh.wikipedia.org/wiki/X.509" %}

{% embed url="https://zh.wikipedia.org/wiki/%E4%BF%A1%E4%BB%BB%E9%8F%88" %}
