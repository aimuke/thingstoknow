# http协议制定者 http working group



**HTTPWG（HTTP Working Group）是 HTTP RFC 规范的** **作者、维护者和发布者**，是 **IETF 内负责 HTTP 协议标准化的核心工作组**。

***

一句话总结

| 组织           | 角色       | HTTP RFC 关系             |
| ------------ | -------- | ----------------------- |
| **HTTPWG**   | **制定者**  | **编写、维护、发布所有 HTTP RFC** |
| **IETF**     | **发布平台** | **HTTPWG 的母组织，发布 RFC**  |
| **HTTP RFC** | **最终产品** | **HTTPWG 工作成果**         |

## 1. **基本信息**

| 项目         | 内容                                                     |
| ---------- | ------------------------------------------------------ |
| **全称**     | HTTP Working Group                                     |
| **网址**     | [https://httpwg.org/](https://httpwg.org/)             |
| **成立时间**   | 2012 年（接管 HTTP/1.1 维护）                                 |
| **当前主席**   | Mark Nottingham, Tommy Pauly                           |
| **邮件列表**   | [httpwg@ietf.org](mailto:httpwg@ietf.org)              |
| **GitHub** | [https://github.com/httpwg](https://github.com/httpwg) |

## 2. **HTTPWG 的使命**

> **"标准化 HTTP 协议的所有方面，包括语义、语法、传输、安全和扩展"**

具体职责：

* **维护 HTTP/1.1** → RFC 9112
* **定义 HTTP 语义** → RFC 9110
* **标准化 HTTP/2** → RFC 9113
* **开发 HTTP/3** → RFC 9114
* **缓存、认证、安全** → RFC 9111, 7235 等

***

## HTTPWG 与 HTTP RFC 的关系演变

| 时期            | 组织                     | 成果                                       |
| ------------- | ---------------------- | ---------------------------------------- |
| **1990s**     | **个体 + IETF**          | RFC 1945 (HTTP/1.0), RFC 2616 (HTTP/1.1) |
| **2008-2014** | **HTTP Bis WG**        | RFC 7230-7235（HTTP/1.1 重构）               |
| **2012-至今**   | **HTTPWG**             | **RFC 9110-9114**（现代 HTTP 标准）            |
| **2016-至今**   | **HTTPWG + HTTP/3 WG** | HTTP/3（QUIC）                             |

***

## 当前 HTTPWG 活跃项目（2025 年）

| 项目                          | 状态   | 相关 RFC                      |
| --------------------------- | ---- | --------------------------- |
| **HTTP/3**                  | ✅ 完成 | RFC 9114                    |
| **HTTP Semantics**          | ✅ 完成 | RFC 9110                    |
| **HTTP/2**                  | ✅ 完成 | RFC 9113                    |
| **HTTP Structured Headers** | ✅ 完成 | RFC 8941                    |
| **HTTP Priority**           | ✅ 完成 | RFC 9218                    |
| **HTTP CONNECT**            | 进行中  | draft-ietf-httpbis-tunnel   |
| **HTTP/2.0 后续**             | 讨论中  | draft-ietf-httpbis-http2bis |

**查看最新**：[https://datatracker.ietf.org/wg/httpbis/about/](https://datatracker.ietf.org/wg/httpbis/about/)

***

### HTTPWG 的关键人物

| 姓名                  | 角色     | 贡献             |
| ------------------- | ------ | -------------- |
| **Roy Fielding**    | 作者     | HTTP/1.1 原始设计者 |
| **Mark Nottingham** | 主席     | RFC 9110 主编    |
| **Julian Reschke**  | 编辑     | RFC 7230-7235  |
| **Martin Thomson**  | 编辑     | HTTP/2, HTTP/3 |
| **Mike Bishop**     | HTTP/3 | QUIC/HTTP/3 专家 |

***

## 实际使用 HTTPWG 资源

### 1. **HTML 版 RFC（推荐）**

```
https://httpwg.org/specs/
├── rfc9110.html     # HTTP Semantics
├── rfc9112.html     # HTTP/1.1
├── rfc9113.html     # HTTP/2
└── rfc9114.html     # HTTP/3
```

### 2. **最新草案**

```
https://httpwg.org/http-core/draft-ietf-httpbis-semantics-latest.html
```

### 3. **编辑器版本（实时更新）**

```
https://httpwg.org/http-core/  # 比 RFC 更新的开发版
```

***

## 与其他组织的对比

| 组织                   | 职责              | HTTP RFC 关系 |
| -------------------- | --------------- | ----------- |
| **HTTPWG**           | **制定 HTTP RFC** | **核心**      |
| **IETF**             | 发布 RFC          | **平台**      |
| **W3C**              | Web 应用标准        | 无           |
| **WHATWG**           | HTML/JS 规范      | 部分重叠        |
| **Cloudflare/Cloud** | 实现反馈            | **测试者**     |

***

### 推荐关注方式

1. **订阅 HTTPWG 邮件**：[https://lists.ietf.org/listinfo/httpwg](https://lists.ietf.org/listinfo/httpwg)
2. **关注 GitHub**：[https://github.com/httpwg/http-core](https://github.com/httpwg/http-core)
3. **定期查最新 RFC**：[https://httpwg.org/specs/](https://httpwg.org/specs/)

