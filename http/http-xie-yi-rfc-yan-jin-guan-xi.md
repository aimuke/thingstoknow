# http 协议 rfc 演进关系

## 一、时间线 + RFC 演进图（1991–2025）

<figure><img src="../.gitbook/assets/image (155).png" alt=""><figcaption></figcaption></figure>



```svg
graph TD
    A[1991: HTTP/0.9<br>Tim Berners-Lee 草稿] --> B[1996: RFC 1945<br>HTTP/1.0]
    B --> C[1997: RFC 2068<br>HTTP/1.1 初版]
    C --> D[1999: RFC 2616<br>HTTP/1.1 正式版<br>取代 2068]
    D --> E[2014: RFC 7230–7235<br>HTTP/1.1 重构<br>6个文档]
    E --> F[2015: RFC 7540<br>HTTP/2]
    F --> G[2018: RFC 7541<br>HPACK]
    G --> H[2022: RFC 9110–9114<br>现代 HTTP 标准<br>语义+传输分离]
    H --> I[2022: RFC 9113<br>HTTP/2 新版]
    H --> J[2022: RFC 9114<br>HTTP/3]
    J --> K[未来: HTTP/4?]
```



## 二、详细演进表（每阶段核心 RFC）



<table><thead><tr><th width="128">阶段</th><th>年份</th><th>主要 RFC</th><th>标题</th><th>关键变化</th><th>状态</th></tr></thead><tbody><tr><td><strong>HTTP/0.9</strong></td><td>1991</td><td>—</td><td>—</td><td>仅 <code>GET</code>，无头字段，无状态码</td><td>实验性</td></tr><tr><td><strong>HTTP/1.0</strong></td><td>1996</td><td><strong>RFC 1945</strong></td><td><em>HTTP/1.0</em></td><td>引入 <code>HEAD</code>、<code>POST</code>、状态码、头字段</td><td><strong>Informational</strong></td></tr><tr><td><strong>HTTP/1.1 初版</strong></td><td>1997</td><td><strong>RFC 2068</strong></td><td><em>HTTP/1.1</em></td><td>持久连接、<code>Host</code> 头、管道化</td><td><strong>Draft Standard</strong></td></tr><tr><td><strong>HTTP/1.1 正式</strong></td><td>1999</td><td><strong>RFC 2616</strong></td><td><em>HTTP/1.1</em></td><td>取代 2068，规范持久连接、缓存</td><td><strong>Draft Standard</strong> → <strong>Standard</strong></td></tr><tr><td><strong>HTTP/1.1 重构</strong></td><td>2014</td><td><strong>RFC 7230–7235</strong></td><td><em>HTTP/1.1 系列</em></td><td>分拆为 6 文档： • 7230: 语法 • <strong>7231: 语义</strong> • 7232: 条件请求 • 7233: 范围请求 • 7234: 缓存 • 7235: 认证</td><td><strong>Standard</strong></td></tr><tr><td><strong>HTTP/2</strong></td><td>2015</td><td><strong>RFC 7540</strong></td><td><em>HTTP/2</em></td><td>二进制帧、流、多路复用、HPACK</td><td><strong>Proposed Standard</strong></td></tr><tr><td><strong>HPACK</strong></td><td>2015</td><td><strong>RFC 7541</strong></td><td><em>HPACK</em></td><td>头压缩</td><td><strong>Proposed Standard</strong></td></tr><tr><td><strong>HTTP 语义统一</strong></td><td>2022</td><td><strong>RFC 9110</strong></td><td><em>HTTP Semantics</em></td><td><strong>取代 RFC 7231</strong>，适用于 1.1/2/3</td><td><strong>Internet Standard</strong></td></tr><tr><td><strong>HTTP/1.1 新语法</strong></td><td>2022</td><td><strong>RFC 9112</strong></td><td><em>HTTP/1.1</em></td><td>取代 RFC 7230</td><td><strong>Internet Standard</strong></td></tr><tr><td><strong>HTTP/2 新版</strong></td><td>2022</td><td><strong>RFC 9113</strong></td><td><em>HTTP/2</em></td><td><strong>取代 RFC 7540</strong>，引用 RFC 9110</td><td><strong>Internet Standard</strong></td></tr><tr><td><strong>HTTP/3</strong></td><td>2022</td><td><strong>RFC 9114</strong></td><td><em>HTTP/3</em></td><td>基于 QUIC</td><td><strong>Proposed Standard</strong></td></tr><tr><td><strong>状态码注册</strong></td><td>2004</td><td><strong>RFC 3864</strong></td><td><em>Status Code Registry</em></td><td>规范化 <code>418</code>、<code>425</code> 等注册</td><td><strong>Updates RFC 2817</strong></td></tr></tbody></table>
