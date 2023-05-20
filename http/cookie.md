# cookie

## cookies的起源

&#x20;早期Web开发面临的最大问题之一是如何管理状态。简言之，服务器端没有办法知道两个请求是否来自于同一个浏览器。那时的办法是在请求的页面中插入一个token，并且在下一次请求中将这个token返回（至服务器）。这就需要在form中插入一个包含token的隐藏表单域，或着在URL的qurey字符串中传递该token。这两种办法都强调手工操作并且极易出错。

&#x20;[Lou](http://en.wikipedia.org/wiki/Lou\_Montulli)[ Montulli,](http://en.wikipedia.org/wiki/Lou\_Montulli)那时是网景通讯的一个雇员，被认为在1994年将“[magic cookies](http://en.wikipedia.org/wiki/Magic\_cookie)”的概念应用到了web通讯中。他意图解决的是web中的购物车，现在所有购物网站都依赖购物车。他的最早的[说明文档](http://curl.haxx.se/rfc/cookie\_spec.html)提供了一些cookies工作原理的基本信息该文档在[RFC2109](http://tools.ietf.org/html/rfc2109)中被规范化（这是所有浏览器实现cookies的参考依据），并且最终逐步形成了[REF2965](http://tools.ietf.org/html/rfc2965).Montulli最终也被授予了关于cookies的美国[专利](http://v3.espacenet.com/publicationDetails/biblio?CC=US\&NR=5774670\&KC=\&FT=E)。网景浏览器在它的第一个版本中就开始支持cookies，并且当前所有web浏览器都支持cookies。

## 简介

HTTP Cookie（也叫 Web Cookie 或浏览器 Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态。Cookie 使基于无状态的HTTP协议记录稳定的状态信息成为了可能。

Cookie 主要用于以下三个方面：

* **会话状态管理**（如用户登录状态、购物车、游戏分数或其它需要记录的信息）&#x20;
* **个性化设置**（如用户自定义设置、主题等）&#x20;
* **浏览器行为跟踪**（如跟踪分析用户行为等）

&#x20;Cookie 曾一度用于客户端数据的存储，因当时并没有其它合适的存储办法而作为唯一的存储手段，但现在随着现代浏览器开始支持各种各样的存储方式，Cookie 渐渐被淘汰。由于服务器指定 Cookie 后，浏览器的每次请求都会携带 Cookie 数据，会带来额外的性能开销（尤其是在移动环境下）。新的浏览器API已经允许开发者直接将数据存储到本地，如使用 Web storage API （本地存储和会话存储）或 IndexedDB 。



## 创建Cookie

当服务器收到 HTTP 请求时，服务器可以在响应头里面添加一个 [`Set-Cookie`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Set-Cookie) 选项。浏览器收到响应后通常会保存下 Cookie，之后对该服务器每一次请求中都通过  [`Cookie`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cookie) 请求头部将 Cookie 信息发送给服务器。另外，Cookie 的过期时间、域、路径、有效期、适用站点都可以根据需要来指定。

### [`Set-Cookie响应头部`和`Cookie请求头部`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies#set-cookie%E5%93%8D%E5%BA%94%E5%A4%B4%E9%83%A8%E5%92%8Ccookie%E8%AF%B7%E6%B1%82%E5%A4%B4%E9%83%A8) <a href="#setcookie-xiang-ying-tou-bu-he-cookie-qing-qiu-tou-bu" id="setcookie-xiang-ying-tou-bu-he-cookie-qing-qiu-tou-bu"></a>

服务器使用 [`Set-Cookie`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Set-Cookie) 响应头部向用户代理（一般是浏览器）发送 Cookie信息。一个简单的 Cookie 可能像这样：

```
Set-Cookie: <cookie名>=<cookie值>
Set-Cookie:value [ ;expires=date][ ;domain=domain][ ;path=path][ ;secure]
```

消息头的第一部分，value部分，通常是一个name=value格式的字符串。事实上，原始手册指示这是应该使用的格式，但是浏览器对cookie的所有值并不会按此格式校验。实际上，你可以指定一个不包含等号的字符串并且它同样会被存储。然而，通常性的使用方式是以name=value的格式（并且多数的接口只支持该格式）来指定cookie的值。

服务器通过该头部告知客户端保存 Cookie 信息。

```
HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry; Expires=Wed, 21 Oct 2015 07:28:00 GMT;

[页面内容]
```

现在，对该服务器发起的每一次新请求，浏览器都会将之前保存的Cookie信息通过 [`Cookie`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cookie) 请求头部再发送给服务器。当客户端发起请求，`cookie`会被包含在 `HTTP` 报文的 `Cookie` 头中发送到服务器，设置项会被忽略。

```
GET /sample_page.html HTTP/1.1
Host: www.example.org
Cookie: yummy_cookie=choco; tasty_cookie=strawberry
```

### set-cookie

```
Set-Cookie: <cookie-name>=<cookie-value>
Set-Cookie: <cookie-name>=<cookie-value>; Expires=<date>
Set-Cookie: <cookie-name>=<cookie-value>; Max-Age=<non-zero-digit>

// Multiple directives are also possible, for example:
Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>; Secure; HttpOnly
```

一个 cookie 开始于一个名称/值对：

* `<cookie-name>` 可以是除了控制字符 (CTLs)、空格 (spaces) 或制表符 (tab)之外的任何 US-ASCII 字符。同时不能包含以下分隔字符： ( ) < > @ , ; : \ " /  \[ ] ? = { }.
* `<cookie-value>` 是可选的，如果存在的话，那么需要包含在双引号里面。支持除了控制字符（CTLs）、空格（whitespace）、双引号（double quotes）、逗号（comma）、分号（semicolon）以及反斜线（backslash）之外的任意 US-ASCII 字符。**关于编码**：许多应用会对 cookie 值按照URL编码（URL encoding）规则进行编码，但是按照 RFC 规范，这不是必须的。不过满足规范中对于 \<cookie-value> 所允许使用的字符的要求是有用的。
* **`__Secure-` 前缀**：以 \_\_Secure- 为前缀的 cookie（其中连接符是前缀的一部分），必须与 secure 属性一同设置，同时必须应用于安全页面（即使用 HTTPS 访问的页面）。
* **`__Host-` 前缀：** 以 \_\_Host- 为前缀的 cookie，必须与 secure 属性一同设置，必须应用于安全页面（即使用 HTTPS 访问的页面），必须不能设置 domain 属性 （也就不会发送给子域），同时 path 属性的值必须为“/”。

其余属性见  [set-cookie](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Set-Cookie)

### cookie编码（cookie encoding）

对于cookie的值进行编码一直都存在一些困惑。通常的观点是cookie的值必须被URL编码，但是这其实是一个谬误，尽管可以对cookie的值进行URL编码。原始的文档中指示仅有三种类型的字符必须进行编码：分号，逗号，和空格。规范中提到可以利用URL编码，但是并不是必须。RFC没有提及任何的编码。然而，几乎所有的实现方式都对cookie的值进行了一系列的URL编码。对于name=value的格式，name和value通常都单独进行编码并且不对等号“=”进行编码操作。

## cookie的生命周期

cookie是有生命周期的，一旦到了cookie的失效日期，客户端的cookie就会被删除。服务器在创建cookie时可以控制一个cookie可以在客户端“存活”多长时间。在以下几种情况下，cookie都会结束它自己的生命周期：

* **未指定过期时间的cookie(会话期cookie)**：没有指定对应的过期时间时，客户端会将这类cookie写入浏览器开辟的一块内存中，当关闭浏览器以后，这块内存也就被释放了，对应的cookie也就是结束了它的生命。需要注意的是，有些浏览器提供了会话恢复功能，这种情况下即使关闭了浏览器，会话期Cookie 也会被保留下来，就好像浏览器从来没有关闭一样，这会导致 Cookie 的生命周期无限期延长。
* **指定过期时间的cookie**：当服务器创建一个cookie的时候指定了对应的过期时间时，当到达了过期时间时，对应的cookie就会被删除；&#x20;
* **当浏览器中的cookie数量达到了限制时**，那么浏览器就会按照某种策略删除一些旧的cookie，腾出空间来创建新的cookie；&#x20;
* **手动的人为删除cookie**



**References**

* [Http cookie解析](https://www.jianshu.com/p/add4e97be281)
* [HTTP cookies](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies) , MDN
* [Set-Cookie](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Set-Cookie), MDN
* [HTTP cookies 详解](https://www.cnblogs.com/ajianbeyourself/p/4900140.html) ([英文原文 HTTP cookies explained](https://humanwhocodes.com/blog/2009/05/05/http-cookies-explained/))，推荐

&#x20;
