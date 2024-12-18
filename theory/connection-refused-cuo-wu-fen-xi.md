# Connection refused 错误分析



## 常见原因

1\. **服务器的端口没有打开**       这种直接就是一直会Connection refused，不会间歇出现，可以直接排除；

2\. **服务器的防火墙没有开白名单**    很多跟外部对接的时候，是需要将公司出口ip加到对方防火墙白名单，这种也会直接Connection refused，不会间歇出现，可以直接排除；

3\. **服务器上的backlog设置的太小**，导致连接队列满了，服务器可能会报Connection refused，或者Connecttion reset by peer，这个看服务器上的连接队列满时的设置；



## TCF建链流程

<figure><img src="../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

TCP正常三次握手

第一步：client 发送syn到server发起握手；

第二步:  server收到syn后回复syn + ack 给client；

第三步：client收到syn + ack后，回复server一个ack表示收到server的syn + ack；



详细流程如下：

<figure><img src="../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

