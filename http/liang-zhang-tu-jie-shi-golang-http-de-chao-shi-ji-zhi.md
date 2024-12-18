# 两张图解释Golang http的超时机制

转自[The complete guide to Go net/http timeouts](https://blog.cloudflare.com/the-complete-guide-to-golang-net-http-timeouts/)。

## 服务端超时

对于`http.Server`服务端有两个超时可以设置：`ReadTimeout`和`WriteTimeout`

```go
srv := &http.Server{
    ReadTimeout: 5 * time.Second,
    WriteTimeout: 10 * time.Second,
}
log.Println(srv.ListenAndServe())
```

各自的作用时间见图：

<figure><img src="https://www.ichenfu.com/images/go_http_timeout_server.png" alt=""><figcaption></figcaption></figure>

需要注意的是`WriteTimeout`被设置了两次，一次是在读取Http头过程中，另一次是在读取Http头结束后。

## 客户端超时

对于`http.Client`客户端，相对要复杂一点，一般的初始化代码如下：

```go
c := &http.Client{
    Transport: &http.Transport{
        Dial: (&net.Dialer{
                Timeout:   30 * time.Second,
                KeepAlive: 30 * time.Second,
        }).Dial,
        TLSHandshakeTimeout:   10 * time.Second,
        ResponseHeaderTimeout: 10 * time.Second,
        ExpectContinueTimeout: 1 * time.Second,
    }
}
```

这些Timeout各自的作用时间见：

<figure><img src="https://www.ichenfu.com/images/go_http_timeout_client.png" alt=""><figcaption></figcaption></figure>

\
