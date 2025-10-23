# 基本的认证方式

W3C在 [RFC 1945: HTTP/1.0 (1996)](https://tools.ietf.org/html/rfc1945) 中，定義了HTTP架構的認證方式。

#### Server - 認證提示 <a href="#toc_2" id="toc_2"></a>

如果Client傳送了未認證的request，Server會回應：

```http
HTTP 401 Unauthorized
WWW-Authenticate: <type> realm="xxx" ...
```

例如：

```http
WWW-Authenticate: Basic realm="User Visible Realm"
```

#### Client - 認證 <a href="#toc_3" id="toc_3"></a>

Client會在request header加入以下資料來認證自己：

```http
Authorization: <type> <credentials>
```

* type：認證方式，例如Basic, Digest...
* credentials：與type相符的密碼

例如：

```http
Authorization: Basic YWxpY2U6c3VwZXJtYW4=
```

四种基本的认证方式:

```
1. Basic Auth
2. Digest Auth
3. Bearer
4. OAuth
```

## basic authentication 基本认证



Basic Auth，也称为 HTTP 基本认证（HTTP Basic Authentication），是一种用于 HTTP 协议的简单认证机制，HTTP 基本认证由互联网工程任务组（IETF）在 [RFC 7617](https://datatracker.ietf.org/doc/html/rfc7617) 中定义。

在 Basic Auth 中，客户端在发送请求时，将用户名和密码以 <mark style="color:red;">`Base64`</mark> 编码的形式包含在请求头的 <mark style="color:red;">Authorization</mark> 字段中发送给服务器，服务器收到请求后，会解码 Authorization 字段并验证用户名和密码。

### Basic Auth 的工作原理 <a href="#basicauth-de-gong-zuo-yuan-li" id="basicauth-de-gong-zuo-yuan-li"></a>

1.**客户端请求**：当客户端访问一个保护的资源时，如果没有提供认证信息，服务器会返回一个 `401 Unauthorized` 状态和一个 `WWW-Authenticate` 响应头，指示客户端需要通过何种认证机制进行认证。例如:

```
WWW-Authenticate: Basic realm="Access to the staging site", charset="UTF-8"
```

2.**编码认证信息**：客户端收到 401 响应后，会提示用户输入用户名和密码。然后，客户端将用户名和密码以 <mark style="color:red;">`username:password`</mark> 的形式拼接成一个字符串，再将这个字符串通过 <mark style="color:red;">Base64</mark> 编码方式编码成一个编码字符串。

3.**发送认证请求**：编码后的字符串被置于 HTTP 的 `Authorization` 请求头中并重新发送请求，格式如下：

```
Authorization: Basic [encoded_string]
```

其中 `[encoded_string]` 是 Base64 编码后的字符串。

4.**服务器认证**：服务器接收到包含认证信息的请求后，解码 Base64 编码字符串，提取用户名和密码，然后验证这些凭据。如果认证通过，则服务器会处理请求并返回所请求的资源。如果认证失败，它可以再次返回 401 状态码。

<figure><img src="https://mdn.github.io/shared-assets/images/diagrams/http/authentication/basic-auth.svg" alt="" width="563"><figcaption></figcaption></figure>

### 客户端如何使用 Basic Auth？ <a href="#ke-hu-duan-ru-he-shi-yong-basicauth" id="ke-hu-duan-ru-he-shi-yong-basicauth"></a>

在客户端使用 `Basic Auth` 时，通常需要在 HTTP 请求的头部中添加一个字段来传递用户名和密码，这个字段的名称是`Authorization`，它的值是 `Basic`加上用户名和密码的 Base64 编码。下面是一个使用 JavaScript 的 Axios 库发送 `Basic Auth` 请求的例子：

```
const axios = require('axios')

// 设置用户名和密码
const username = 'your_username'
const password = 'your_password'

// 构造认证信息
const credentials = `${username}:${password}`

// 对认证信息进行 Base64 编码
const encodedCredentials = Buffer.from(credentials).toString('base64')

// 发送请求
axios
  .get('https://api.example.com/data', {
    headers: {
      // highlight-next-line
      Authorization: `Basic ${encodedCredentials}`,
    },
  })
  .then((response) => {
    // 处理响应数据
    console.log('Response:', response.data)
  })
  .catch((error) => {
    // 处理错误
    console.error('Error:', error)
  })
```

### 优缺点 <a href="#an-quan-xing-he-xian-zhi" id="an-quan-xing-he-xian-zhi"></a>

**优点:**

1、简单，容易实现，而且大多数客户端和浏览器都支持basic auth 兼容性较好

2、高效，由于服务端会对每个接受到的请求独立的进行认证，因此认证流程是无状态的，只需要一次请求就可以完整认证和请求。

**缺点:**

1、不安全， base64 是编码不是加密，攻击者很融合拦截和解密认证信息，并进行重放攻击

2、由于不支持复杂的密码策略，容易暴力破解

### 安全性和限制 <a href="#an-quan-xing-he-xian-zhi" id="an-quan-xing-he-xian-zhi"></a>

尽管 Basic Auth 是 HTTP 认证中最直接的一种方式，但它存在一些明显的安全风险：

**明文问题**：Base64 是一种编码方式，而不是加密方式。任何人在拦截 HTTP 请求的情况下可以轻松地解码并获取用户名和密码。

**不提供认证状态的持续性**：每次请求都需要发送用户名和密码，这可能会导致凭据被多次暴露。

因此，Basic Auth 通常不推荐用于传输敏感信息或在不安全的网络（如互联网）中使用，除非连接通过 SSL/TLS（如 HTTPS）来确保传输层的安全。



### 补充说明

basic auth 只提供只是提供了基础的认证功能，可以定义了客户端向服务端提供认证信息的方式。

其中使用的 base64 方式进行编码，是对称编码方式，拦截到请求后是可以直接反向解码出来的。这里进行<mark style="color:red;">base64 编码应该是由于http 协议header 中 value 只能是 ascii 的可打印字符</mark>。[https://www.rfc-editor.org/rfc/rfc7230.html#section-3.2.6](https://www.rfc-editor.org/rfc/rfc7230.html#section-3.2.6) 进行编码后，控制字符，非 `ascii` 字符等都会被编码符合http 协议要求。

由于 base64 不是加密，因此使用 basic auth 认证是不安全的。在使用了 https +  basic auth 后，https 可以提供加密功能，但是如果攻击者控制了客户端，依然可以重新发送之前的报文，造成重放攻击。

## Digest 认证-摘要认证 <a href="#id-77699fd8-763d-52d5-5fe9-5e5ee46230ae" id="id-77699fd8-763d-52d5-5fe9-5e5ee46230ae"></a>

Digest Auth（摘要认证），全称 Digest Access Authentication，是一种用于网络协议中的安全验证机制，它提供了比[基本认证](https://docs.apifox.com/what-is-basic-auth)（Basic Authentication）更安全的方式来管理网页或网络服务的用户验证。这种认证机制主要用于通过 HTTP 协议进行通信的场景中，是 HTTP 认证的一种方式，该规范被定义在 [RFC 7616](https://datatracker.ietf.org/doc/html/rfc7616) 中。

### Digest Auth 的工作原理 <a href="#digestauth-de-gong-zuo-yuan-li" id="digestauth-de-gong-zuo-yuan-li"></a>

1.**客户端请求访问资源：** 当用户尝试访问一个受保护的资源时，客户端（通常是浏览器）会发送一个请求到服务器。

2.**服务器响应：** 如果资源需要认证，服务器会返回一个 401 未授权响应，并在响应头中包含一个 `WWW-Authenticate` 字段，该字段提供了进行下一步加密操作所需的数据，如域（Realm）、一个随机数（Nonce），以及其他可选参数。

3.**客户端计算摘要并发送：** 客户端使用用户名、密码和从服务器接收到的数据（比如：Realm、Nonce），生成一个加密的摘要（Digest）。这个过程通常涉及到哈希函数，如 MD5。客户端再次发送请求，附带 Authorization 头，其中包含摘要以及相关的认证信息。

4.**服务器验证：** 服务器接收到客户端的响应后，会用同样的方法计算摘要值，如果计算结果与客户端发送的摘要值匹配，则认证成功；否则，认证失败。

注： md5 是非对称算法，客户端对用户名和密码等使用 md5 计算后得到response， 服务端在根据用户名和服务端的密码等重新使用md5 计算一次，如果得到的 response 相等就认证成功。攻击者由于不知道客户端密码，所以无法伪造报文。也无法重放请求。

```


客户端                                                 服务器
  |                                                     |
  | ----------- 1. 请求访问受保护资源 ----------------->  |
  |                                                     |
  |                                                     |
  | <---------- 2. 401 未授权，发送 challenge ---------  |
  |            包含随机数（nonce）和域（realm）           |
  |                                                     |
  |                                                     |
  | ----------- 3. 发送认证摘要 ---------------------->  |
  |             包含用户名、摘要值等                      |
  |                                                     |
  |                                                     |
  |<----------  4. 认证结果 -------------------------    |
  |              认证成功/失败                           |
  |                                                     |
```



### WWW-Authenticate 的响应头字段 <a href="#wwwauthenticate-de-xiang-ying-tou-zi-duan" id="wwwauthenticate-de-xiang-ying-tou-zi-duan"></a>

在 HTTP Digest Auth（摘要认证）中，服务器通过 `WWW-Authenticate` 首部字段向客户端发出认证请求。根据 [RFC 7616](https://datatracker.ietf.org/doc/html/rfc7616) 规范，该字段包含若干参数，用于定义认证所需的具体细节。常见的字段及其描述如下：

**realm：** 指定了受保护资源所在的域（realm），客户端通常会显示该信息，以提示用户输入正确的凭据。

**nonce：** 一个唯一的随机字符串，用于防止[重放攻击](https://zh.wikipedia.org/wiki/%E9%87%8D%E6%94%BE%E6%94%BB%E5%87%BB)。服务器生成 nonce 值，并在每次需要进行身份验证时发送给客户端，客户端使用它来计算摘要。

**algorithm：** 指定用于计算消息摘要的算法。常见的包括 MD5、MD5-sess、SHA-256 等。

**qop：** 指定质询保护的类型（Quality of Protection），常见值为`auth`，表示身份验证质询仅认证头，`auth-int` 认证头和请求体。

**nc (nonce count)：** 这是一个十六进制的计数器，表示客户端使用同一 `nonce` 值发送请求的次数。每次发送请求时，计数器的值都会增加。这确保了每个请求都是唯一的，从而阻止重放攻击。

**cnonce (client nonce)：** 这是由客户端生成并发送的一个随机字符串，与服务器的 nonce 一起工作，增强了认证过程的安全性。客户端 nonce 确保了客户端生成的请求具有独特性，并可能参与生成响应中的摘要。

**opaque：** 一个字符串，由服务器生成，并作为一个参数提供给客户端，客户端必须将其原样返回。通常用于增加安全性，防止攻击者获取敏感信息。

**domain：** 指定了受保护资源的范围，即可以使用此认证机制的 URI 范围。

**stale：** 指示服务器之前发送的 nonce 是否过期，当设置为 true 时，表示之前提供的认证信息已过期，需要客户端重新认证。



服务器向客户端提供认证所需的各项参数，下面是一个示例`WWW-Authenticate`响应头字段：

```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Digest realm="example.com",
                   qop="auth,auth-int",
                   nonce="dcd98b7102dd2f0e8b11d0f600bfb0c093",
                   opaque="5ccc069c403ebaf9f0171e9517f40e41",
                   stale="false",
                   algorithm=MD5
```

在这个示例中，服务器向客户端提供了`realm`、`qop`、`nonce`、`opaque` 和 `stale` 这些字段，用于协助客户端进行 Digest Auth（摘要认证）。在客户端响应服务器的`WWW-Authenticate`请求时，客户端将这些值包含在 Authorization 头字段中，形式大致如下：

```
Authorization: Digest username="user",
               realm="example.com",
               nonce="dcd98b7102dd2f0e8b11d0f600bfb0c093",
               uri="/dir/index.html",
               qop=auth,
               nc=00000001,
               cnonce="0a4f113b",
               response="6629fae49393a05397450978507c4ef1",
               opaque="5ccc069c403ebaf9f0171e9517f40e41"
```

### 客户端如何使用 Digest Auth？

在客户端使用 HTTP Digest Auth（摘要认证）通常涉及几个步骤，一般情况下你需要手动处理认证过程，即需要自行处理服务器的挑战（challenge），计算响应值（response），再发起请求。以下一个基本的示例说明如何在 JavaScript 中处理摘要认证，使用 Axios 作为 HTTP 客户端库。注意，以下代码不是即插即用的完整实现，但展示了核心逻辑。

#### 1.发送初步请求，获取 challenge 参数 <a href="#id-1-fa-song-chu-bu-qing-qiu-huo-qu-challenge-can-shu" id="id-1-fa-song-chu-bu-qing-qiu-huo-qu-challenge-can-shu"></a>

```
import axios from 'axios'

// 初始请求资源URL
const url = 'http://example.com/protected'

// 第一次请求，预期会失败并获得 401 和 WWW-Authenticate 头
axios.get(url).catch((error) => {
  if (error.response && error.response.status === 401) {
    const wwwAuthenticate = error.response.headers['www-authenticate']
    const authDetails = parseWWWAuthenticate(wwwAuthenticate)
    performDigestAuth(authDetails, url)
  }
})

// 解析认证头信息的函数
function parseWWWAuthenticate(header) {
  const parts = header.split(',')
  const details = {}
  parts.forEach((part) => {
    const [key, value] = part.split('=')
    details[key.trim()] = value.replace(/"/g, '')
  })

  return details
}
```

#### 2.使用认证参数和用户信息计算响应，发起认证请求 <a href="#id-2-shi-yong-ren-zheng-can-shu-he-yong-hu-xin-xi-ji-suan-xiang-ying-fa-qi-ren-zheng-qing-qiu" id="id-2-shi-yong-ren-zheng-can-shu-he-yong-hu-xin-xi-ji-suan-xiang-ying-fa-qi-ren-zheng-qing-qiu"></a>

```
function performDigestAuth(authDetails, url) {
  // 认证参数 authDetails 中包含realm, nonce等
  const username = 'your_username'
  const password = 'your_password'

  const cnonce = generateCNonce()
  const nc = '00000001' // Nonce count, 多次请求时递增
  const response = calculateDigestResponse(
    authDetails,
    username,
    password,
    cnonce,
    nc,
  )

  // 构造 Authorization 头
  const authHeader = `Digest username="${username}", realm="${authDetails.realm}", nonce="${authDetails.nonce}", uri="${url}", response="${response}", opaque="${authDetails.opaque}", qop=${authDetails.qop}, nc=${nc}, cnonce="${cnonce}"`

  // 使用摘要认证信息再次发起请求
  axios
    .get(url, {headers: {Authorization: authHeader}})
    .then((response) => {
      console.log('Authenticated Request Successful', response.data)
    })
    .catch((error) => {
      console.log('Authenticated Request Failed', error)
    })
}

function generateCNonce() {
  return Math.random().toString(36).substring(7)
}

function calculateDigestResponse(authDetails, username, password, cnonce, nc) {
  // 需要根据RFC 7616实现摘要计算
  // 示例仅为演示目的
  // 通常会涉及到MD5或其他散列函数生成response
  return 'Calculated response hash here'
}
```

在实际实现中，你需要根据 [RFC 7616](https://datatracker.ietf.org/doc/html/rfc7616) 设定的公式和方法，正确生成 response 值。这包括使用合适的散列函数（如 MD5）计算 HA1、HA2，最后计算 response 值。还应该处理 `auth-int` 中的实体主体散列处理（如果使用了 qop=auth-int）。



### 优缺点

**优点：**

1、由于传输的数据中同时包含了密码和服务端的随机数，进行加密后攻击者不知道密码就无法解密也无法重新生成报文； 由于报文中有服务端随机数与nc计数器，攻击者也没有办法进行报文的重放。

2、支持通过 qop 设置计算摘要的范围，如果同时对head 和 body 都计算摘要的话，还可以保证请求的完整性

3、支持双向认证

**缺点：**

1、实现复杂

2、由于需要沟通摘要算法的相关信息，需要一次而外的通信





## Bearer-令牌认证

* 在 Bearer Token 认证中，客户端在请求中携带一个令牌（Token），该令牌通常是通过 OAuth 或类似的身份验证流程获得的。服务端验证这个令牌来确认用户的身份。
* Bearer 认证的核心是 bearer token。bearer token 是一个加密字符串，通常由服务端根据密钥生成。客户端在请求服务端时，必须在请求头中包含 Authorization: Bearer \<token>。服务端收到请求后，解析出 \<token> ，并校验 \<token> 的合法性，如果校验通过，则认证通过。跟基本认证一样，Bearer 认证需要配合 HTTPS 一起使用，来保证认证安全性。
* 为了区分用户和保证安全，必须对 API 请求进行鉴权，但是不能要求每一个请求都进行登录操作。合理做法是，在第一次登录之后产生一个有一定有效期的 token，并将它存储在浏览器的 Cookie 或 LocalStorage 之中。之后的请求都携带这个 token ，请求到达服务器端后，服务器端用这个 token 对请求进行认证。在第一次登录之后，服务器会将这个 token 用文件、数据库或缓存服务器等方法存下来，用于之后请求中的比对。
* 当前最流行的 token 编码方式是 JSON Web Token(JWT)



## OAuth-开放授权 <a href="#id-57e6cf54-f9cc-497f-aab1-77c974cba074" id="id-57e6cf54-f9cc-497f-aab1-77c974cba074"></a>



## References

{% embed url="http://www.webdav.org/specs/rfc2617.html#rfc.section.2" %}

{% embed url="https://www.baeldung.com/cs/digest-vs-basic-authentication" %}

{% embed url="https://knowledge.zhaoweiguo.com/build/html/secure/authns/basic" %}

{% embed url="https://docs.apifox.com/what-is-digest-auth" %}

{% embed url="https://docs.apifox.com/what-is-basic-auth" %}
