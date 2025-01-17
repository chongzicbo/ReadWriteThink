## 1. HTTP报文

![image-20250102165228332](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102165228332.png)

HTTP报文是用于HTTP协议交互的信息，分为请求报文和响应报文两种类型。请求报文由客户端发送，响应报文由服务器端返回。HTTP报文由多行数据构成，每行以CR+LF（回车符和换行符）作为换行符。

#### 结构
HTTP报文大致分为两个主要部分：
1. **报文首部**：包含服务器端或客户端需要处理的请求或响应的内容及属性。
2. **报文主体**：包含应被发送的数据。

这两部分由最初出现的空行（CR+LF）来划分。需要注意的是，报文主体并不是必须的。

#### 示例
- **请求报文**：
  ```
  GET / HTTP/1.1
  Host: hackr.jp
  User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64; rv:13.0) Gecko/20100101 Firefox/13.0.1
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
  Accept-Language: ja,en-us;q=0.7,en;q=0.3
  Accept-Encoding: gzip, deflate
  DNT:1
  Connection: keep-alive
  Pragma: no-cache
  Cache-Control: no-cache
  
  ```

- **响应报文**：
  ```
  HTTP/1.1 200 OK
  Date: Fri, 13 Jul 2012 02:45:26 GMT
  Server: Apache
  Last-Modified: Fri, 31 Aug 2007 02:02:20 GMT
  ETag: "45bae1-16a-46d776ac"
  Accept-Ranges: bytes
  Content-Length: 362
  Connection: close
  Content-Type: text/html
  
  hackr.jp
  ```

## 2. 请求报文及响应报文的结构

#### 请求报文和响应报文的基本结构

![image-20250102165438438](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102165438438.png)

请求报文和响应报文是HTTP通信中的基本单位，分别由客户端和服务器端生成。它们的结构相似，主要由以下几个部分组成：

1. **请求行（Request Line）**（仅存在于请求报文中）
   - 包含用于请求的方法（如GET、POST等）、请求的URI（统一资源标识符）和HTTP版本。

2. **状态行（Status Line）**（仅存在于响应报文中）
   - 包含表明响应结果的状态码、原因短语和HTTP版本。

3. **首部字段（Header Fields）**
   - 包含表示请求和响应的各种条件和属性的各类首部字段。
   - 首部字段分为四种类型：
     - 通用首部（General Headers）：适用于请求和响应报文。
     - 请求首部（Request Headers）：仅适用于请求报文。
     - 响应首部（Response Headers）：仅适用于响应报文。
     - 实体首部（Entity Headers）：适用于请求和响应报文的实体部分。

4. **空行（CR+LF）**
   - 用于分隔首部字段和报文主体。

5. **报文主体（Message Body）**
   - 包含实际传输的数据。

#### 请求报文的实例

![image-20250102165602500](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102165602500.png)

1. **请求行**：
   - `GET /HTTP/1.1`：这是请求方法、请求URI和HTTP版本。这里使用了GET方法，表示请求获取资源，URI是`/`，HTTP版本是1.1。
2. **首部字段**：
   - `Host: hackr.jp`：指定请求的目标服务器主机名。
   - `User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64; rv:13.0) Gecko/20100101 Firefox/13.0.1`：说明发起请求的用户代理（浏览器）信息。
   - `Accept: text/html, application/xhtml+xml, application/xml;q=0.9, */*;q=0.8`：说明客户端可以接受的响应内容类型及其优先级。
   - `Accept-Language: ja,en-us;q=0.7,en;q=0.3`：说明客户端可以接受的语言及其优先级。
   - `Accept-Encoding: gzip, deflate`：说明客户端可以接受的编码方式。
   - `DNT: 1`：表示客户端请求不要追踪（Do Not Track）。
   - `Connection: keep-alive`：表示客户端希望保持连接，以便后续请求可以复用该连接。
   - `Pragma: no-cache`：指示客户端不使用缓存。
   - `Cache-Control: no-cache`：指示客户端不使用缓存。
3. **空行（CR+LF）**：
   - 空行表示首部字段的结束，接下来可能会有消息体（如果有的话），但在这个实例中没有消息体

#### 响应报文的实例

![image-20250102165622595](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102165622595.png)

1. **状态行**：`HTTP/1.1 200 OK`
   - `HTTP/1.1` 表示使用的是HTTP/1.1协议。
   - `200 OK` 是状态码，表示请求成功。
2. **首部字段**：
   - `Date`: 响应生成的日期和时间。
   - `Server`: 服务器软件的信息。
   - `Last-Modified`: 资源的最后修改时间。
   - `ETag`: 资源的实体标签，用于缓存验证。
   - `Accept-Ranges`: 表示服务器是否支持范围请求。
   - `Content-Length`: 响应主体的长度。
   - `Connection`: 连接的状态，这里是`close`，表示关闭连接。
   - `Content-Type`: 响应主体的媒体类型和字符集。
3. **空行**：状态行和首部字段与报文主体之间的空行（CR+LF），表示首部字段的结束。
4. **报文主体**：包含实际的HTML内容，这里是简单的HTML页面，包含一个图片标签。

#### 首部字段的详细说明

- **请求行**：包含请求的方法、请求URI和HTTP版本。
  - 例如：`GET / HTTP/1.1`

- **状态行**：包含状态码、原因短语和HTTP版本。
  - 例如：`HTTP/1.1 200 OK`

- **首部字段**：包含各种条件和属性的信息。
  - 通用首部：如`Date`、`Server`、`Connection`等。
  - 请求首部：如`Host`、`User-Agent`、`Accept`等。
  - 响应首部：如`Last-Modified`、`ETag`、`Accept-Ranges`等。
  - 实体首部：如`Content-Length`、`Content-Type`等。

- **空行**：用于分隔首部字段和报文主体。

- **报文主体**：包含实际传输的数据。

通过以上结构，HTTP请求报文和响应报文能够有效地传递请求和响应的信息，确保HTTP通信的正常进行。

## 3.　编码提升传输速率

HTTP协议在传输数据时可以采用编码方式来提升传输速率，尽管这会增加服务器的CPU等资源消耗。

#### 3.1 报文主体和实体主体的差异

- **报文（Message）**：HTTP通信中的基本单位，由8位组字节流（octet sequence）组成，通过HTTP通信传输。
- **实体（Entity）**：作为请求或响应的有效载荷数据，由实体首部和实体主体组成。
- **报文主体**：用于传输请求或响应的实体主体。
- **差异**：通常报文主体等于实体主体，但在传输中进行编码操作时，实体主体的内容发生变化，导致两者产生差异。例如，使用gzip压缩时，实体主体的内容会被压缩，而报文主体则包含压缩后的数据。

#### 3.2 压缩传输的内容编码

![image-20250102173124080](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102173124080.png)

- **内容编码**：类似于邮件附件的压缩，HTTP协议中的内容编码功能可以对实体内容进行压缩，保持实体信息原样压缩。
- **常用编码方式**：
  - `gzip`（GNU zip）
  - `compress`（UNIX系统的标准压缩）
  - `deflate`（zlib）
  - `identity`（不进行编码）

#### 3.3 分割发送的分块传输编码

![image-20250102173144111](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250102173144111.png)

- **分块传输编码（Chunked Transfer Coding）**：将实体主体分成多个部分（块），每一块用十六进制标记块的大小，最后一块使用“0(CR+LF)”标记。
- **优点**：允许在HTTP通信过程中逐步传输大容量数据，浏览器可以逐步显示页面，而不必等待整个资源传输完成。
- **过程**：客户端接收分块传输编码的实体主体并负责解码，恢复到编码前的实体主体。

这些编码和传输技术提高了HTTP协议的效率和灵活性，能够有效处理大量访问请求。

## 4. 多部分对象集合（Multipart）

![多部分对象集合](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103084922896.png)

1. **多部分对象集合（Multipart）**：允许在一个HTTP报文主体内包含多种不同类型的实体数据，类似于电子邮件中的附件功能。

2. **常用多部分对象集合类型**：
   - **multipart/form-data**：用于Web表单文件上传。
   - **multipart/byteranges**：用于状态码206（Partial Content）响应报文，包含多个范围的内容。

3. **结构示例**：
   - **multipart/form-data**：在Web表单文件上传时使用，示例如下：
     ```
     Content-Type: multipart/form-data; boundary=AaB03x
     --AaB03x
     Content-Disposition: form-data; name="field1"
     Joe Blow
     --AaB03x
     Content-Disposition: form-data; name="pics"; filename="file1.txt" Content-Type: text/plain
     ...(file1.txt的数据)...
     --AaB03x--
     ```
   - **multipart/byteranges**：在状态码206响应报文中使用，示例如下：
     ```
     HTTP/1.1 206 Partial Content
     Date: Fri, 13 Jul 2012 02:45:26 GMT
     Last-Modified: Fri, 31 Aug 2007 02:02:20 GMT
     Content-Type: multipart/byteranges; boundary=THIS_STRING_SEPARATES
     --THIS_STRING_SEPARATES
     Content-Type: application/pdf
     Content-Range: bytes 500-999/8000
     ...
     --THIS_STRING_SEPARATES Content-Type: application/pdf
     --- 
     Content-Range: bytes 7000-7999/8000
     ...(范围指定的数据) --THIS_STRING_SEPARATES--
     ```

4. **边界字符串（Boundary String）**：用于划分多部分对象集合中的各类实体，在每个实体的起始行之前插入“--”标记，并在最后一个实体的结束处插入“--”标记。

5. **嵌套使用**：多部分对象集合的每个部分类型中都可以含有首部字段，并且可以在某个部分中嵌套使用多部分对象集合。

## 5. 获取部分内容范围请求（Range Request）

![范围请求](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103085128597.png)

1. **背景**：
   - 在过去，由于互联网带宽较低，下载大文件时容易中断，重新下载会浪费时间和带宽。
   - 范围请求解决了这个问题，允许从上次中断的地方继续下载。

2. **范围请求的定义**：
   - 范围请求是一种指定下载资源范围的请求。
   - 例如，对于一个10000字节的资源，可以只请求5001到10000字节的部分。

3. **Range首部字段**：
   - 用于指定资源的字节范围。
   - 格式如下：
     - `Range: bytes=5001-10000` 表示请求5001到10000字节。
     - `Range: bytes=5001-` 表示从5001字节之后的所有内容。
     - `Range: bytes=-3000` 表示从开始到3000字节的内容。
     - `Range: bytes=5000-7000` 表示请求5000到7000字节的内容。

4. **响应**：
   - 如果服务器支持范围请求，会返回状态码206 Partial Content。
   - 响应报文的首部字段Content-Type会标明`multipart/byteranges`，并返回请求的部分内容。
   - 如果服务器不支持范围请求，会返回状态码200 OK和完整的实体内容。

5. **多部分范围请求**：
   - 支持多重范围的范围请求，例如同时请求5001到10000字节和5000到7000字节的内容。

通过这些机制，HTTP协议能够有效地处理大文件的传输，并在网络中断时实现断点续传。

## 6. 内容协商返回最合适的内容

![image-20250103085443032](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103085443032.png)

1. **内容协商的定义**：
   - 内容协商是指客户端和服务器端就响应的资源内容进行交涉，然后提供给客户端最为适合的资源。内容协商会以响应资源的语言、字符集、编码方式等作为判断的基准。

2. **请求报文中的首部字段**：
   - 包含在请求报文中的某些首部字段是判断的基准，这些字段包括：
     - `Accept`
     - `Accept-Charset`
     - `Accept-Encoding`
     - `Accept-Language`
     - `Content-Language`

3. **内容协商的类型**：
   - **服务器驱动协商（Server-driven Negotiation）**：由服务器端进行内容协商，以请求的首部字段为参考，在服务器端自动处理。但对用户来说，以浏览器发送的信息作为判定的依据，并不一定能筛选出最优内容。
   - **客户端驱动协商（Agent-driven Negotiation）**：由客户端进行内容协商的方式。用户从浏览器显示的可选项列表中手动选择。还可以利用JavaScript脚本在Web页面上自动进行上述选择。比如按OS的类型或浏览器类型，自行切换成PC版页面或手机版页面。
   - **透明协商（Transparent Negotiation）**：是服务器驱动和客户端驱动的结合体，是由服务器端和客户端各自进行内容协商的一种方法。

通过这些机制，HTTP协议能够根据客户端的偏好和能力，提供最合适的资源内容。