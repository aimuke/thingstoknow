# 从wireshake分析http和https的通信过程

原文地址：[https://www.cnblogs.com/zhangrunhao/p/10428759.html](https://www.cnblogs.com/zhangrunhao/p/10428759.html)

## HTTPS中的TCP交互过程 <a href="#https-zhong-de-tcp-jiao-hu-guo-cheng" id="https-zhong-de-tcp-jiao-hu-guo-cheng"></a>

> 一句话概况: 通过非对称加密的方式来传递对称加密所需要的秘钥.

我懒, 还是贴图好了(原谅我这名盗图狗), 等会我们在抓包的过程中逐步分析.

![https图解](https://upload-images.jianshu.io/upload_images/2111324-19f47f0a6829c6f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/409/format/webp)

注意一点: `ssl / tls`的交互过程, 是在完成TCP三次握手后开始的.

## 具体的抓包分析 <a href="#ju-ti-de-zhua-bao-fen-xi" id="ju-ti-de-zhua-bao-fen-xi"></a>

### 分析`SSL/TLS`Client Hello数据包 <a href="#bu-zhou-4-fen-xi-ssltlsclienthello-shu-ju-bao" id="bu-zhou-4-fen-xi-ssltlsclienthello-shu-ju-bao"></a>

双击打开, 带有`Client Hello`标识的tsl数据包. 展开安全传输层, TLS和握手协议, 去查看SSL/TLS中的详细数据.

可以观察到客户端支持的各种加密方式.

<figure><img src="http://zhangrunhao.oss-cn-beijing.aliyuncs.com/blog/wireshark-tcp/16.jpg" alt=""><figcaption></figcaption></figure>

下一个带有`TCP ACK`标识的tcp数据包, 那是服务器端对于收到客户端请求的回应.

### `SSL/TLS`Server Hello数据包 <a href="#bu-zhou-5-fen-xi-ssltlsserverhello-shu-ju-bao" id="bu-zhou-5-fen-xi-ssltlsserverhello-shu-ju-bao"></a>

双击打开, 带有`Server Hello`标识的数据包.观察`Secure Sockets Layer`, 也就数安全数据层.

服务器端返回了他所支持的加密方式, 是客户端所传递的一个子集.&#x20;

<figure><img src="http://zhangrunhao.oss-cn-beijing.aliyuncs.com/blog/wireshark-tcp/17.jpg" alt=""><figcaption></figcaption></figure>

### SSL/TLS 交换证书的阶段 <a href="#bu-zhou-6-fen-xi-ssltls-jiao-huan-zheng-shu-de-jie-duan" id="bu-zhou-6-fen-xi-ssltls-jiao-huan-zheng-shu-de-jie-duan"></a>

打开带有`Certificate, Server Key Exchange, Server Hello Done.`标识的数据包. 展开`Secure Sockets Layer`, 让我们来仔细观察安全层所携带的数据.

这一次tcp上面, tls的数据包含了三块: 分别是: 证书, 非对称加密的公钥(Server Key), 还有一个服务器信息结束标识.

<figure><img src="http://zhangrunhao.oss-cn-beijing.aliyuncs.com/blog/wireshark-tcp/18.jpg" alt=""><figcaption></figcaption></figure>

还可以看到我们的公钥信息!.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

客户端使用证书来验证公钥和签名. 这些工作浏览器会帮助我们进行处理.

### 客户端的秘钥交换 <a href="#bu-zhou-7-ke-hu-duan-de-mi-yao-jiao-huan" id="bu-zhou-7-ke-hu-duan-de-mi-yao-jiao-huan"></a>

双击打开, 带有`Client Key Exchange, Change Cipher Spec, Encrypted Handshake Message`标识的tcp数据包.客户端使用公钥对将对称加密的秘钥进行加密, 并发送给了服务器.

从抓包上来看: 具体过程应该是, 客户端的秘钥交换, 然后更改加密规范, 然后对与握手的信息进行了加密

<figure><img src="http://zhangrunhao.oss-cn-beijing.aliyuncs.com/blog/wireshark-tcp/20.jpg" alt=""><figcaption></figcaption></figure>

### 开始数据交互 <a href="#bu-zhou-8-kai-shi-shu-ju-jiao-hu" id="bu-zhou-8-kai-shi-shu-ju-jiao-hu"></a>

* 后面就是带有`Application Data`的数据之间的传递了, 此时的数据都是经过加密的.
* 留个疑问, 有些交互过程带有`New Session Ticket.`标识的数据包, 服务器用来确定加密信息, 有些不带有, 还不是很清楚的具体原因. 有待学习, 有待学习.\
