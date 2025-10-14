# 证书格式

## 证书文件扩展名 <a href="#zheng-shu-wen-jian-kuo-zhan-ming" id="zheng-shu-wen-jian-kuo-zhan-ming"></a>

X.509 有多种常用的扩展名。不过其中的一些还用于其它用途，就是说具有这个扩展名的文件可能并不是证书，比如说可能只是保存了私钥。

* `.pem` – 隐私增强型电子邮件格式，通常是Base64格式的。
* `.cer`, `.crt`, `.der` – 通常是 DER 二进制格式的。
* `.p7b`, `.p7c` – PKCS#7 SignedData structure without data, just certificate(s) or CRL(s)
* `.p12` – PKCS#12格式，包含证书的同时可能还包含私钥
* `.pfx` – PFX，PKCS#12之前的格式（通常用PKCS#12格式，比如由互联网信息服务产生的PFX文件）

PKCS#7 是签名或加密数据的格式标准，官方称之为容器。由于证书是可验真的签名数据，所以可以用SignedData结构表述。 .P7C文件是退化的SignedData结构，没有包括签名的数据。

PKCS#12 由PFX进化而来的用于交换公共的和私有的对象的标准格式。

.key格式：私有的密钥

.csr格式：证书签名请求（证书请求文件），含有公钥信息，certificate signing request的缩写

.crt格式：证书文件，certificate的缩写

.crl格式：证书吊销列表，Certificate Revocation List的缩写

.pem格式：用于导出，导入证书时候的证书的格式，有证书开头，结尾的格式

## 证书格式转换 <a href="#zheng-shu-ge-shi-zhuan-huan" id="zheng-shu-ge-shi-zhuan-huan"></a>

### PEM转为DER <a href="#pem-zhuan-wei-der" id="pem-zhuan-wei-der"></a>

```
openssl x509 -in cert.crt -outform der -out cert.der
```

### DER转为PEM <a href="#der-zhuan-wei-pem" id="der-zhuan-wei-pem"></a>

```
openssl x509 -in cert.crt -inform der -outform pem -out cert.pem
```

查看 DER 格式证书的信息

```
openssl x509 -in certificate.der -inform der -text -noout
```

### 证书文件扩展名 <a href="#zheng-shu-wen-jian-kuo-zhan-ming-1" id="zheng-shu-wen-jian-kuo-zhan-ming-1"></a>

#### CRT <a href="#crt" id="crt"></a>

表示证书，常见于 linux 系统，可能是 PEM 编码和 DER 编码，大多数是 PEM 编码。

#### CER <a href="#cer" id="cer"></a>

表示证书，常见于 Windows 系统，可能是 PEM 编码和 DER 编码，大多数是 DER 编码。

#### KEY <a href="#key" id="key"></a>

存放一个公钥或者私钥，编码可能是 PEM 或者 DER。查看 KEY 的办法:

```
openssl rsa -in test.key -text -noout
```

## CSR <a href="#csr" id="csr"></a>

Certificate Signing Request，证书签名请求，核心内容是一个公钥，在生成这个申请的时候，同时也会生成一个私钥。

CSR 是 Certificate Signing Request 的简称，它是向 CA 机构申请数字证书时使用的请求文件。在生成请求文件前，我们需要准备一对对称密钥。私钥信息自己保存，请求中会附上公钥信息以及国家，城市，域名，Email等信息，CSR中还会附上签名信息。当我们准备好 CSR 文件后就可以提交给CA机构，等待他们给我们签名，签好名后我们会收到crt文件，即证书。

CSR并不是证书，而是向权威证书颁发机构获得签名证书的申请。把CSR交给权威证书颁发机构，权威证书颁发机构对此进行签名。保留好CSR，当权威证书颁发机构颁发的证书过期的时候，还可以用同样的CSR来申请新的证书，key保持不变。

#### 1.3.5. PFX/P12 <a href="#pfxp12" id="pfxp12"></a>

predecessor of PKCS#12，常用语windows IIS。

#### 1.3.6. JKS <a href="#jks" id="jks"></a>

即Java Key Storage，这是Java的专利。

## 证书编码格式 <a href="#zheng-shu-wen-jian-ge-shi" id="zheng-shu-wen-jian-ge-shi"></a>

### PEM <a href="#pem" id="pem"></a>

是 Privacy Enhanced Mail 的简称，通常用于数字证书认证机构（Certificate Authorities，CA），扩展名为 `.pem`， `.crt`, `.cer`， `.key`。内容为 Base64 编码的 ASCII 码文件，有类似 `“-----BEGIN CERTIFICATE-----“` 和 `“-----END CERTIFICATE-----“` 的头尾标记。服务器认证证书，中级认证证书和私钥都可以储存为 PEM 格式（认证证书其实就是公钥）。Apache 和 nginx 等类似的服务器使用 PEM 格式证书。

### DER <a href="#der" id="der"></a>

是 Distinguished Encoding Rules 的简称，与 PEM 不同之处在于其使用二进制而不是 Base64 编码的 ASCII。扩展名为 `.der`，但也经常使用 `.cer` 用作扩展名，所有类型的认证证书和私钥都可以存储为 DER 格式。Java 和 Windows 服务器使用 DER 格式证书。\


## References

{% embed url="https://www.zhaowenyu.com/https-doc/pki/x509-format.html" %}
