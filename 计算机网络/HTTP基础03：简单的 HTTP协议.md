---
title: 'HTTP基础03：简单的HTTP协议'
categories:
  - [计算机基础,计算机网络]
tags:
  - 计算机网络
  - 计算机基础
  - HTTP
date: 2025-01-02 12:00:00
---



## 简单的 HTTP协议

## 1. HTTP协议用于客户端和服务器端之间的通信

![image-20250102104311290](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102104311290.png)

- **定义**：HTTP协议用于客户端和服务器之间的通信。
- **角色**：请求访问资源的一端称为客户端，提供资源响应的一端称为服务器端。
- **通信线路**：在一条通信线路上，必定有一端是客户端，另一端是服务器端。
- **角色互换**：在某些情况下，两台计算机可能会互换客户端和服务器端的角色，但在一条通信路线上，角色是确定的。

## 2. 通过请求和响应的交换达成通信

![请求和响应](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102104434929.png)

**请求和响应**：HTTP协议规定请求从客户端发出，服务器端响应该请求并返回。

**请求报文**：由请求方法、请求URI、协议版本、可选的请求首部字段和内容实体构成。

- **示例**：`GET /index.htm HTTP/1.1 Host: hackr.jp`

![请求报文](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102104508513.png)

**响应报文**：由协议版本、状态码、原因短语、可选的响应首部字段以及实体主体构成。

![image-20250102104617041](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102104617041.png)

- **示例**：
  
  ```
  HTTP/1.1 200 OK
  Date: Tue, 10 Jul 2012 06:50:15 GMT
  Content-Length: 362
  Content-Type: text/html
  ```

## 3. HTTP是不保存状态的协议

![无状态协议](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102104640175.png)

- **无状态协议**：HTTP协议自身不对请求和响应之间的通信状态进行保存。
- **优点**：为了更快地处理大量事务，确保协议的可伸缩性。
- **缺点**：随着Web的发展，无状态导致业务处理变得棘手。
- **解决方案**：引入Cookie技术来管理状态。

## 4. 请求URI定位资源

![URI定位资源](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102104835594.png)

- **URI功能**：HTTP协议使用URI定位互联网上的资源。
- **请求URI**：客户端请求访问资源时，URI需要包含在请求报文中。
  - **完整URI**：`GET http://hackr.jp/index.htm HTTP/1.1`
  - **Host字段**：`GET /index.htm HTTP/1.1 Host: hackr.jp`
  - **通配符**：`OPTIONS * HTTP/1.1`

## 5. 告知服务器意图的 HTTP方法
### 5.1 GET: 获取资源
- **用途**：用于请求访问已被URI识别的资源。
- **特点**：请求的资源会被服务器解析后返回响应内容。如果是文本资源，则原样返回；如果是程序（如CGI），则返回执行后的输出结果。
- **示例请求**：
  ```
  GET /index.html HTTP/1.1
  Host: www.example.com
  ```
- **示例响应**：
  ```
  HTTP/1.1 200 OK
  Content-Type: text/html
  <html>...</html>
  ```

### 5.2 POST: 传输实体主体
- **用途**：用于传输实体的主体，通常用于提交表单数据或其他数据。
- **特点**：虽然GET方法也可以传输数据，但POST方法更适合传输大量数据或敏感信息，因为它不会将数据暴露在URL中。
- **示例请求**：
  ```
  POST /submit.cgi HTTP/1.1
  Host: www.example.com
  Content-Length: 1560
  <form data>
  ```
- **示例响应**：
  ```
  HTTP/1.1 200 OK
  Content-Type: text/html
  <result of form submission>
  ```

### 5.3 PUT: 传输文件
- **用途**：用于传输文件，类似于FTP协议的文件上传。
- **特点**：请求报文的主体中包含文件内容，服务器将其保存到请求URI指定的位置。由于PUT方法不带验证机制，存在安全性问题，一般Web网站不使用该方法。
- **示例请求**：
  ```
  PUT /example.html HTTP/1.1
  Host: www.example.com
  Content-Type: text/html
  Content-Length: 1560
  <file content>
  ```
- **示例响应**：
  ```
  HTTP/1.1 204 No Content
  ```

### 5.4 HEAD: 获得报文首部
- **用途**：类似于GET方法，但不返回报文主体部分，仅用于确认URI的有效性及资源更新的日期时间等。
- **特点**：常用于检查资源是否存在或获取资源的元数据。
- **示例请求**：
  ```
  HEAD /index.html HTTP/1.1
  Host: www.example.com
  ```
- **示例响应**：
  ```
  HTTP/1.1 200 OK
  Date: Mon, 27 Jul 2009 12:28:53 GMT
  Server: Apache/2.2.14 (Win32)
  Last-Modified: Wed, 22 Jul 2009 19:15:56 GMT
  Content-Length: 88
  Content-Type: text/html
  ```

### 5.5 DELETE: 删除文件
- **用途**：用于删除文件，是与PUT相反的操作。
- **特点**：请求URI指定的资源将被删除。由于DELETE方法不带验证机制，存在安全性问题，一般Web网站不使用该方法。
- **示例请求**：
  ```
  DELETE /example.html HTTP/1.1
  Host: www.example.com
  ```
- **示例响应**：
  ```
  HTTP/1.1 204 No Content
  ```

### 5.6 OPTIONS: 询问支持的方法
- **用途**：用于查询针对请求URI指定的资源支持的方法。
- **特点**：返回服务器支持的各种HTTP方法。
- **示例请求**：
  ```
  OPTIONS * HTTP/1.1
  Host: www.example.com
  ```
- **示例响应**：
  ```
  HTTP/1.1 200 OK
  Allow: GET, POST, HEAD, OPTIONS
  ```

### 5.7 TRACE: 追踪路径
- **用途**：用于让Web服务器端将之前的请求通信环回给客户端。
- **特点**：通过Max-Forwards首部字段的值递减，最终返回状态码200 OK的响应，包含请求内容。常用于调试和诊断请求路径。
- **示例请求**：
  ```
  TRACE / HTTP/1.1
  Host: www.example.com
  Max-Forwards: 2
  ```
- **示例响应**：
  ```
  HTTP/1.1 200 OK
  Content-Type: message/http
  Content-Length: 1024
  TRACE / HTTP/1.1 Host: www.example.com Max-Forwards: 2
  ```

### 5.8 CONNECT: 要求用隧道协议连接代理
- **用途**：用于在与代理服务器通信时建立隧道，实现用隧道协议进行TCP通信，常用于SSL/TLS加密通信。
- **特点**：主要用于HTTPS请求。
- **示例请求**：
  ```
  CONNECT www.example.com:443 HTTP/1.1
  Host: www.example.com
  ```
- **示例响应**：
  ```
  HTTP/1.1 200 Connection Established
  ```



## 6. 使用方法下达命令
- **方法**：向请求URI指定的资源发送请求报文时，采用称为方法的命令。
- **支持的方法**：GET、POST、HEAD等。

![image-20250102113459101](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102113459101.png)

## 7. 持久连接节省通信量

![初始版本](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102113558809.png)

HTTP协议的初始版本中，每次HTTP通信都需要断开TCP连接。这在传输小容量文本时问题不大，但随着HTTP的普及，文档中包含大量图片的情况增多，导致每次请求都会造成无谓的TCP连接建立和断开，增加了通信量的开销。

![image-20250102113810021](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102113810021.png)

### 7.1 持久连接

![image-20250102113835652](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102113835652.png)

为了解决上述TCP连接的问题，HTTP/1.1和一部分HTTP/1.0引入了持久连接（HTTP Persistent Connections，也称为HTTP keep-alive或HTTP connection reuse）的方法。持久连接的特点是，只要任意一端没有明确提出断开连接，则保持TCP连接状态。

**优点**：
- 减少了TCP连接的重复建立和断开所造成的额外开销。
- 减轻了服务器端的负载。
- 减少开销的那部分时间，使HTTP请求和响应能够更早地结束，从而提高了Web页面的显示速度。

在HTTP/1.1中，所有的连接默认都是持久连接，但在HTTP/1.0内并未标准化。虽然有一部分服务器通过非标准的手段实现了持久连接，但服务器端不一定能够支持持久连接。毫无疑问，除了服务器端，客户端也需要支持持久连接。

### 7.2 管线化

![image-20250102113951113](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102113951113.png)

持久连接使得多数请求以管线化（pipelining）方式发送成为可能。以前发送请求后需等待并收到响应，才能发送下一个请求。管线化技术出现后，不用等待响应亦可直接发送下一个请求。

**优点**：
- 能够同时并行发送多个请求，而不需要一个接一个地等待响应。
- 当请求一个包含多张图片的HTML Web页面时，与挨个连接相比，用持久连接可以让请求更快结束。而管线化技术则比持久连接还要快。请求数越多，时间差就越明显。

通过持久连接和管线化技术，HTTP协议大大提高了通信效率，减少了不必要的开销，提升了用户体验。

## 8. 使用 Cookie的状态管理
HTTP协议是一种无状态协议，这意味着它不会保存请求和响应之间的通信状态。这种设计简化了协议，使其能够高效地处理大量事务，但也带来了一些挑战，特别是在需要保持用户状态的情况下。例如，用户在登录后访问网站的不同页面时，仍然需要保持登录状态。

![没有Cookie](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102114136299.png)

#### 8.1 无状态协议的缺点
- **状态管理困难**：由于HTTP不保存状态，服务器无法识别同一用户的多次请求，导致每次请求都需要重新认证。
- **用户体验不佳**：用户每次访问新页面时可能需要重新登录，影响用户体验。

#### 8.2 Cookie技术的引入

![没有Cookie](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102114244688.png)

![有了Cookie](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102114256778.png)

为了解决无状态协议带来的问题，引入了Cookie技术。Cookie通过在请求和响应报文中写入Cookie信息来控制客户端的状态。

#### 8.3 Cookie的工作原理
1. **服务器生成Cookie**：服务器在响应报文中添加一个`Set-Cookie`首部字段，通知客户端保存Cookie。
   ```http
   HTTP/1.1 200 OK
   Set-Cookie: session_id=1234567890
   ```

2. **客户端保存Cookie**：客户端在收到响应后，会自动保存Cookie信息。

3. **客户端发送Cookie**：下次客户端向同一服务器发送请求时，会自动在请求报文中加入Cookie值。
   ```http
   GET /page HTTP/1.1
   Host: example.com
   Cookie: session_id=1234567890
   ```

4. **服务器读取Cookie**：服务器端发现客户端发送的Cookie后，会检查并对比服务器上的记录，从而识别出请求的用户。

#### 8.4 Cookie的应用场景
- **用户认证**：通过Cookie保存用户的登录状态，避免每次请求都需要重新登录。
- **个性化设置**：保存用户的偏好设置，如主题、语言等。
- **购物车**：保存用户的购物车内容，方便用户继续购物。

#### 8.5 Cookie的限制
- **安全性**：Cookie可以被篡改，因此敏感信息不应存储在Cookie中。
- **隐私**：Cookie可以追踪用户的浏览行为，可能引发隐私问题。
- **大小限制**：每个域名下的Cookie数量和总大小有限制，通常每个Cookie不超过4KB。

通过Cookie技术，HTTP协议能够在保持其简单高效的同时，实现状态管理功能，提升用户体验。





参考：《图解HTTP》第一章

文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)