## 1. 基于HTTP的协议

1. **HTTP协议的原始设计**：HTTP协议最初设计用于传输HTML文档，但随着Web用途的多样化（如在线购物、社交网络服务等），其性能逐渐显得不足。

2. **Web应用和脚本程序的局限性**：尽管Web应用和脚本程序可以实现许多功能，但在性能上未必最优，主要受限于HTTP协议本身的限制。

3. **新协议的必要性**：为了弥补HTTP协议的不足，创建了一套全新的协议，但这些新协议通常是基于HTTP并在其基础上添加新功能的。

4. **现有环境的约束**：由于基于HTTP的Web浏览器在全球范围内广泛使用，完全抛弃HTTP并不现实，因此新协议需要在HTTP的基础上进行扩展和改进。

## 2. 消除 HTTP 瓶颈的 SPDY

### 2.1 **背景与目标**
   - **背景**: 随着Web应用的发展，HTTP协议在处理实时内容更新和性能方面显得力不从心。特别是在SNS网站，大量用户同时发布内容时，HTTP的性能瓶颈尤为明显。
   - **目标**: SPDY（发音同“speedy”）由Google在2010年发布，旨在解决HTTP的性能瓶颈，缩短Web页面的加载时间（目标为50%）。

### 2.2 **HTTP的瓶颈**

![以前的HTTP通信](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250107161552039.png)

   - **单连接限制**: HTTP协议规定一条连接上只能发送一个请求，导致频繁的客户端到服务器端的确认通信。
   - **请求/响应首部未压缩**: 首部信息多且未压缩，增加了延迟。
   - **冗长首部**: 每次通信都需发送相同的首部，造成资源浪费。
   - **非强制压缩**: 数据压缩格式可任意选择，但非强制压缩。

### 2.3 **Ajax的解决方法**

![Ajax](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250107161636452.png)

   - **异步通信**: Ajax通过JavaScript和DOM操作实现局部页面更新，减少数据传输量。
   - **XMLHttpRequest API**: 通过JavaScript脚本与服务器进行HTTP通信，发起局部页面请求。
   - **局限性**: 虽然减少了数据量，但大量请求仍可能导致性能问题，且未解决HTTP协议本身的问题。

### 2.4 **Comet的解决方法**

![Comet通信](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250107161725565.png)

   - **服务器推送**: Comet通过延迟应答实现服务器向客户端的推送功能。
   - **挂起响应**: 服务器端接收到请求后不立即返回响应，而是等待内容更新后再返回。
   - **局限性**: 虽然实现了实时更新，但长时间连接消耗更多资源，且未解决HTTP协议本身的问题。

### 2.5 **SPDY的设计与功能**

![SPDY](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250107162056417.png)

   - **会话层**: SPDY在TCP/IP的应用层与运输层之间加入会话层，控制数据流动，并使用SSL保证安全性。
   - **多路复用流**: 通过单一TCP连接处理多个HTTP请求，提高TCP处理效率。
   - **请求优先级**: 分配请求优先级顺序，解决带宽低导致的响应慢问题。
   - **压缩HTTP首部**: 压缩请求和响应的首部，减少数据包数量和字节数。
   - **推送功能**: 支持服务器主动向客户端推送数据。
   - **服务器提示功能**: 服务器主动提示客户端请求所需资源，避免不必要的请求。

### 2.6 **SPDY的实际应用**
   - **浏览器和服务器的改动**: Web浏览器和服务器需要为SPDY做出相应改动。部分浏览器已支持SPDY，但实际Web网站的导入进展不佳。
   - **多域名限制**: SPDY仅能多路复用单个域名的通信，对于使用多个域名的Web网站，改善效果受限。

### 2.7 **总结**
   - **有效性**: SPDY是一种有效消除HTTP瓶颈的技术。
   - **局限性**: Web网站的速度提升还需从其他方面入手，如改善Web内容的编写方式等。



## 3. 使用浏览器进行全双工通信的 WebSocket

WebSocket 是一种新的网络协议，旨在解决传统 HTTP 协议在实时通信中的局限性。

### 3.1 WebSocket 的设计与功能

WebSocket 是一种在 Web 浏览器和 Web 服务器之间实现全双工通信的标准协议。全双工通信意味着服务器和客户端可以同时发送和接收数据，而不需要像传统的 HTTP 协议那样每次通信都需要客户端发起请求。

- **推送功能**: 支持服务器向客户端推送数据，服务器可以直接发送数据而不必等待客户端的请求。
- **减少通信量**: 一旦建立 WebSocket 连接，希望一直保持连接状态。相比 HTTP，WebSocket 的首部信息很小，通信量相应减少。
- **独立协议**: 建立在 HTTP 基础上的协议，但连接一旦确立，后续通信不再使用 HTTP 数据帧，而是采用 WebSocket 独立的数据帧。

### 3.2 WebSocket 协议

![WebSocket通信](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250107162430797.png)

WebSocket 协议在 HTTP 连接建立之后，通过一次“握手”步骤来完成连接的建立。

##### 握手·请求

为了实现 WebSocket 通信，需要用到 HTTP 的 Upgrade 首部字段，告知服务器通信协议发生改变。

```http
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Origin: http://example.com
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```

- `Sec-WebSocket-Key`: 握手过程中必不可少的键值。
- `Sec-WebSocket-Protocol`: 记录使用的子协议。
- `Sec-WebSocket-Version`: WebSocket 协议的版本号。

##### 握手·响应

对于之前的请求，服务器返回状态码 101 Switching Protocols 的响应。

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```

- `Sec-WebSocket-Accept`: 由握手请求中的 `Sec-WebSocket-Key` 生成的值。

成功握手后，通信时不再使用 HTTP 的数据帧，而是采用 WebSocket 独立的数据帧。

### 3.3 WebSocket API

JavaScript 可以调用 W3C 标准制定的 “The WebSocket API” 来实现 WebSocket 协议下的全双工通信。

以下是一个每 50ms 发送一次数据的实例：

```javascript
var socket = new WebSocket('ws://game.example.com:12010/');
socket.onopen = function() {
    setInterval(function() {
        if (socket.bufferedAmount == 0)
            socket.send(getUpdateData());
    }, 50);
};
```

- `WebSocket`: 创建一个新的 WebSocket 对象。
- `onopen`: 当 WebSocket 连接成功建立时触发的事件处理函数。
- `setInterval`: 定时发送数据。
- `socket.send`: 向服务器发送数据。

### 3.4 总结

WebSocket 通过全双工通信解决了传统 HTTP 协议在实时通信中的局限性，支持服务器向客户端推送数据，减少了通信量，并且通过握手过程实现了从 HTTP 到 WebSocket 的协议转换。WebSocket API 提供了简单易用的接口，使得开发者可以方便地在 Web 应用中实现实时通信功能。

## 4. HTTP2.0

HTTP/2.0 是互联网工程任务组（IETF）在2014年11月实现标准化的下一代HTTP协议。其主要目标是改善用户在Web浏览时的速度体验。HTTP/2.0基于SPDY和WebSocket等技术，旨在解决HTTP/1.1的性能瓶颈。

### 4.1 HTTP/2.0的特点

1. **多路复用**：允许在单一TCP连接上并行处理多个请求和响应，减少延迟。
2. **头部压缩**：使用HPACK算法压缩HTTP头部，减少传输的数据量。
3. **服务器推送**：服务器可以主动向客户端推送资源，减少客户端请求的次数。
4. **二进制分帧层**：将所有传输的信息分割成更小的帧，提高传输效率。
5. **优先级和依赖性**：允许为请求设置优先级，确保重要资源优先加载。

### 4.2 HTTP/2.0的7项技术及讨论

HTTP/2.0围绕以下7项技术进行讨论：

1. **压缩**：SPDY和Friendly的HTTP Speed+ Mobility。
2. **多路复用**：SPDY。
3. **TLS义务化**：Speed+ Mobility。
4. **协商**：Speed+ Mobility和Friendly。
5. **客户端拉曳/服务器推送**：Speed+ Mobility。
6. **流量控制**：SPDY。
7. **WebSocket**：Speed+ Mobility。

### 4.3 扩展

HTTP/2.0不仅提高了传输效率，还引入了许多新特性，使其在现代Web应用中更加高效。以下是一些扩展：

1. **增强的安全性**：HTTP/2.0强制使用TLS加密，提高了数据传输的安全性。
2. **更好的资源管理**：通过服务器推送和优先级设置，优化了资源的加载顺序和速度。
3. **减少延迟**：多路复用和二进制分帧层减少了网络延迟，提高了用户体验。
4. **兼容性**：尽管HTTP/2.0引入了许多新特性，但它仍然向后兼容HTTP/1.1，确保现有Web应用的无缝过渡。

HTTP/2.0的推出标志着Web协议的一个重要进步，进一步提升了Web的性能和用户体验。

## 5. Web 服务器管理文件的 WebDAV

WebDAV（Web-based Distributed Authoring and Versioning）是一个扩展HTTP/1.1的协议，允许用户直接在Web服务器上进行文件的复制、编辑等操作。它定义在RFC4918中，提供了比传统HTTP更多的功能，特别是在文件管理和版本控制方面。

### 5.1 WebDAV的基本概念

WebDAV扩展了HTTP/1.1的功能，增加了一些新的概念和操作：

- **集合（Collection）**：类似于文件夹，可以统一管理多个资源。
- **资源（Resource）**：可以是文件或集合。
- **属性（Property）**：定义资源的属性，通常以“名称=值”的格式表示。
- **锁（Lock）**：用于防止多人同时编辑同一资源，确保数据一致性。

### 5.2 WebDAV新增的方法及状态码

为了实现远程文件管理，WebDAV向HTTP/1.1中添加了以下方法和状态码：

- **PROPFIND**: 获取资源的属性。
- **PROPPATCH**: 修改资源的属性。
- **MKCOL**: 创建集合。
- **COPY**: 复制资源及其属性。
- **MOVE**: 移动资源。
- **LOCK**: 对资源加锁。
- **UNLOCK**: 解锁资源。

相应地，状态码也有所扩展：

- **102 Processing**: 请求已被接收，但处理尚未完成。
- **207 Multi-Status**: 包含多种状态信息。
- **422 Unprocessable Entity**: 请求格式正确但内容有误。
- **423 Locked**: 资源已被加锁。
- **424 Failed Dependency**: 处理与某请求关联的请求失败。
- **507 Insufficient Storage**: 保存空间不足。

### 5.3 WebDAV的请求实例

以下是一个使用PROPFIND方法获取文件属性的请求示例：

```http
PROPFIND /file HTTP/1.1
Host: www.example.com
Content-Type: application/xml; charset="utf-8"
Content-Length: 219
```

### 5.4 WebDAV的响应实例

以下是针对上述PROPFIND方法的响应示例：

```http
HTTP/1.1 207 Multi-Status
Content-Type: application/xml; charset="utf-8"
Content-Length: 831
```

### 5.5 WebDAV的优势

WebDAV的主要优势在于其提供的分布式文件系统功能，使得用户可以通过标准的HTTP协议进行文件的远程管理和版本控制。这对于协作编辑、文件共享和版本管理等场景非常有用。

### 5.6 WebDAV的应用场景

WebDAV广泛应用于以下场景：

- **协同编辑**: 多个用户可以同时对同一文件进行编辑，系统会自动处理冲突。
- **文件共享**: 用户可以通过WebDAV协议直接访问和修改服务器上的文件。
- **版本控制**: WebDAV支持文件的版本控制，用户可以查看和管理文件的历史版本。

通过这些扩展功能，WebDAV大大增强了Web服务器的文件管理能力，使其更加灵活和高效。