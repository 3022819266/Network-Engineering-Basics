# HTTP 核心知识学习文档

## 目录

1. [HTTP 请求方法](#一http-请求方法)
2. [HTTP 状态码](#二http-状态码)
3. [HTTP 请求头](#三http-请求头)
4. [会话管理：Cookie 与 Session](#四会话管理cookie-与-session)
5. [HTTP 缓存机制](#五http-缓存机制)
6. [附录：知识关联图](#附录知识关联图)

---

## 一、HTTP 请求方法

HTTP 请求方法（也称 HTTP 动词）定义了客户端希望对目标资源执行的操作类型。

### 1.1 五种核心方法

| 方法 | 语义 | 安全 | 幂等 | 可缓存 | 请求体 | 典型场景 |
|------|------|:----:|:----:|:------:|:------:|------|
| **GET** | 获取资源 |  |  |  |  | 查询数据、请求静态文件 |
| **POST** | 提交数据 |  |  |  |  | 表单提交、文件上传、新建资源 |
| **PUT** | 全量更新 |  |  |  |  | 整体替换某个资源 |
| **PATCH** | 部分更新 |  |  |  |  | 只修改资源的某些字段 |
| **DELETE** | 删除资源 |  |  |  |  | 删除指定资源 |

### 1.2 关键概念说明

- **安全（Safe）**：请求不会对服务器资源产生副作用，即不会修改数据。`GET`、`HEAD`、`OPTIONS`、`TRACE` 是安全的。
- **幂等（Idempotent）**：多次执行相同请求，结果与执行一次相同。`GET`、`PUT`、`DELETE` 是幂等的；`POST` 非幂等（多次提交可能创建多个资源）。
- **PUT vs PATCH**：`PUT` 要求发送完整的资源数据进行整体替换；`PATCH` 只需发送需要变更的字段，更加轻量。

### 1.3 RESTful API 设计示例
GET /users → 获取用户列表 <br>
POST /users → 新建用户 <br>
GET /users/1 → 获取 ID=1 的用户 <br>
PUT /users/1 → 全量更新 ID=1 的用户 <br>
PATCH /users/1 → 部分更新 ID=1 的用户 <br>
DELETE /users/1 → 删除 ID=1 的用户 <br>

---

## 二、HTTP 状态码

HTTP 状态码是服务器返回的三位数字代码，用于告知客户端请求的处理结果。

### 2.1 状态码分类

| 类别 | 含义 | 说明 |
|------|------|------|
| 1xx | 信息性 | 协议处理中间状态，需后续操作 |
| 2xx | 成功 | 请求已被成功接收和处理 |
| 3xx | 重定向 | 资源位置变动，需客户端重新请求 |
| 4xx | 客户端错误 | 请求报文有误，服务器无法处理 |
| 5xx | 服务器错误 | 服务器内部处理出错 |

### 2.2 常见状态码详解

#### 2xx 成功

- **200 OK**：请求成功，返回目标资源。最常见的状态码。
- **204 No Content**：请求成功但无响应体，常用于 `DELETE` 操作。

#### 3xx 重定向

- **301 Moved Permanently**：永久重定向。资源已永久移动到新 URL，浏览器会缓存该跳转。
- **302 Found**：临时重定向。资源临时从另一个 URL 响应，浏览器不缓存。

#### 4xx 客户端错误

- **400 Bad Request**：请求报文存在语法错误或参数无效。
- **401 Unauthorized**：请求需要身份认证，未提供或凭证无效。
- **403 Forbidden**：服务器理解请求但拒绝执行（已认证但无权限）。
- **404 Not Found**：请求的资源在服务器上不存在。

#### 5xx 服务器错误

- **500 Internal Server Error**：服务器内部发生未知错误。
- **502 Bad Gateway**：网关/代理服务器从上游服务器收到了无效响应。

---

## 三、HTTP 请求头

请求头是客户端向服务器发送的元数据，用于传递格式声明、认证信息、客户端标识等控制信息。

> **核心理解**：URL 路径决定"访问哪个资源"，请求头补充说明"我是谁、我用什么格式、我从哪来、我有什么条件"。

### 3.1 常见请求头

| 请求头 | 作用 | 示例 |
|--------|------|------|
| **Host** | 指定目标主机/域名 | `example.com` |
| **Content-Type** | 声明请求体的数据格式 | `application/json`、`multipart/form-data` |
| **Authorization** | 携带认证凭据 | `Bearer eyJhbGci...`、`Basic dXNlcjpwYXNz` |
| **User-Agent** | 标识客户端软件信息 | `Mozilla/5.0 (Windows NT 10.0; Win64; x64)` |
| **Accept** | 声明客户端能接受的响应格式 | `application/json`、`text/html` |
| **Cookie** | 携带客户端保存的会话/状态信息 | `sessionid=abc123` |
| **Referer** | 标识请求来源页面 | `https://example.com/login` |
| **X-Forwarded-For** | 代理场景下传递客户端真实 IP | `203.0.113.19, 70.41.3.18` |

### 3.2 重点说明

- **Content-Type**：告诉服务器请求体是什么格式，服务器据此解析数据。常见的有 `application/json`、`application/x-www-form-urlencoded`、`multipart/form-data`。
- **Authorization**：用于携带身份认证信息，常见方案包括 Bearer Token（JWT）、Basic Auth 等。
- **User-Agent**：服务器可根据该字段识别客户端类型（浏览器、爬虫、移动端等），做差异化响应。
- **X-Forwarded-For（XFF）**：当请求经过反向代理或负载均衡时，源站看到的 IP 是代理的 IP。XFF 用于追溯客户端真实 IP，格式为 `client, proxy1, proxy2, ...`，最左侧为原始客户端 IP。**注意：该字段可被伪造，需配合信任代理白名单使用。**

---

## 四、会话管理：Cookie 与 Session

HTTP 协议本身是无状态的，服务器无法自动关联同一用户的多次请求。Cookie 和 Session 是解决这一问题的核心机制。

### 4.1 对比总览

| 特性 | Cookie | Session |
|------|--------|---------|
| 存储位置 | 客户端（浏览器） | 服务器端 |
| 数据大小 | ≤ 4KB | 理论上无限制 |
| 安全性 | 较低（存储在客户端） | 较高（敏感数据在服务器） |
| 生命周期 | 可设置过期时间，持久化 | 通常随会话结束或超时失效 |
| 依赖关系 | 可独立使用 | 通常依赖 Cookie 传递 Session ID |

### 4.2 工作流程

1. 用户首次访问，服务器创建 Session 并生成唯一 **Session ID**。
2. 服务器通过响应头 `Set-Cookie` 将 Session ID 发送给浏览器。
3. 浏览器保存 Cookie，后续请求自动在请求头 `Cookie` 中携带 Session ID。
4. 服务器根据 Session ID 查找对应的 Session 数据，识别用户身份。

### 4.3 Cookie 安全属性

- **HttpOnly**：禁止 JavaScript 访问该 Cookie，防止 XSS 攻击窃取。
- **Secure**：仅通过 HTTPS 传输，防止明文嗅探。
- **SameSite**：限制跨站请求时携带 Cookie，防止 CSRF 攻击（取值：`Strict` / `Lax` / `None`）。

> **安全提示**：Session ID 若被窃取（如通过 XSS），攻击者可冒充用户身份。因此生产环境应始终为 Session Cookie 设置 `HttpOnly` 和 `Secure` 属性。

---

## 五、HTTP 缓存机制

HTTP 缓存通过在客户端本地存储资源副本，减少网络请求、降低服务器负载、提升页面加载速度。缓存分为**强缓存**和**协商缓存**两个阶段，强缓存优先级更高。

### 5.1 强缓存（强制缓存）

浏览器直接使用本地缓存，**不向服务器发送任何请求**。命中时状态码显示 `200 (from memory/disk cache)`。

控制字段：

- **Cache-Control**（HTTP/1.1，优先级高）
    - `max-age=3600`：缓存有效期 3600 秒
    - `no-cache`：禁用强缓存，强制走协商缓存
    - `no-store`：完全不缓存
    - `public/private`：控制是否允许代理服务器缓存
- **Expires**（HTTP/1.0，已逐渐被 Cache-Control 取代）：指定绝对过期时间，依赖客户端系统时钟。

### 5.2 协商缓存

强缓存过期后，浏览器向服务器发送请求，携带缓存标识进行验证：

- 资源**未变化** → 服务器返回 **304 Not Modified**，浏览器继续使用本地缓存。
- 资源**已变化** → 服务器返回 **200 OK** + 新资源。

协商缓存有两种实现方式：

| 方案 | 响应头（首次请求） | 请求头（后续请求） | 特点 |
|------|------|------|------|
| **Last-Modified** | `Last-Modified: Wed, 03 Jan 2026 10:00:00 GMT` | `If-Modified-Since: ...` | 基于修改时间，精度为秒级，可能存在误判 |
| **ETag** | `ETag: "abc123"` | `If-None-Match: "abc123"` | 基于内容哈希，字节级精确，优先级更高 |

**最佳实践**：静态资源（JS/CSS/图片）使用文件名哈希 + 长 `max-age` 实现强缓存；动态接口使用 `no-cache` 走协商缓存，兼顾性能与数据新鲜度。

### 5.3 缓存存储位置

| 位置 | 特点 | 适用场景 |
|------|------|------|
| Memory Cache | 读取极快，关闭标签页即释放 | 当前页面频繁使用的 JS/CSS |
| Disk Cache | 持久化存储，容量大 | 图片、视频等大资源 |
| Service Worker | 可自定义缓存策略，支持离线 | PWA 应用 |

缓存查找顺序：Service Worker → Memory Cache → Disk Cache → 网络请求。

### 5.4 完整请求流程示例

以资源 `/static/logo.png` 为例：

1. **第一次请求**：服务器返回 `200 OK` + 资源 + `Cache-Control: max-age=3600` + `ETag: "abc123"`，浏览器缓存资源。
2. **5 分钟后再次访问**：强缓存未过期，浏览器直接读取本地副本，状态码 `200 (from disk cache)`，**不向服务器发请求**。
3. **1 小时后再次访问**：强缓存已过期，浏览器带上 `If-None-Match: "abc123"` 询问服务器。
    - 资源未变 → 返回 `304 Not Modified`，浏览器继续使用本地缓存。
    - 资源已变 → 返回 `200 OK` + 新资源 + 新 ETag，浏览器更新缓存。

---

## 附录：知识关联图
HTTP 请求 <br>
├── 请求行（方法 + URL）→ 定义操作意图与目标资源 <br>
├── 请求头（Content-Type/Authorization/User-Agent/Cookie/Host）→ 传递元数据与身份 <br>
├── 请求体（POST/PUT/PATCH 时携带）→ 提交具体数据 <br>
└── 响应 <br>
├── 状态码（200/301/400/401/403/404/500/502）→ 告知处理结果 <br>
├── 响应头（Cache-Control/ETag/Set-Cookie/Content-Type）→ 控制缓存与会话 <br>
└── 响应体 → 返回实际数据 <br>
