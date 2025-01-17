好的，以下是对第4章内容的详细总结：

### 第4章 返回结果的 HTTP状态码

HTTP状态码是用于表示客户端HTTP请求的返回结果、标记服务器端的处理是否正常、以及通知出现的错误等信息的重要工具。

![image-20250103091331807](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103091331807.png)

#### 4.1 状态码告知从服务器端返回的请求结果

状态码的主要职责是描述客户端向服务器发送请求后的处理结果。通过状态码，用户可以了解服务器是否正常处理了请求，或者是否出现了错误。

状态码由三位数字和原因短语组成。第一位数字表示响应类别，后两位则没有分类。响应类别有以下五种：

1. **1XX (Informational)**：表示接收的请求正在处理。
2. **2XX (Success)**：表示请求正常处理完毕。
3. **3XX (Redirection)**：表示需要进行附加操作以完成请求。
4. **4XX (Client Error)**：表示服务器无法处理请求。
5. **5XX (Server Error)**：表示服务器处理请求出错。

尽管RFC2616中定义了40种状态码，加上WebDAV（RFC4918、5842）和附加HTTP状态码（RFC6585）等扩展，总数超过60种，但常用的状态码大约只有14种。接下来，本章将详细介绍这些具有代表性的状态码。

#### 4.2 2XX 成功

2XX系列的状态码表示请求被正常处理。

##### 4.2.1 200 OK

![200](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103091358021.png)

200 OK是最常见的成功状态码，表示从客户端发来的请求在服务器端被正常处理。响应报文中的信息会根据请求方法的不同而有所变化。例如，使用GET方法时，请求资源的实体会作为响应返回；而使用HEAD方法时，只返回请求资源的首部，不返回实体主体部分。

##### 4.2.2 204 No Content

![204](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103091411587.png)

204 No Content状态码表示服务器接收的请求已成功处理，但在响应报文中不包含实体的主体部分。例如，当浏览器发出请求处理后，返回204响应，浏览器显示的页面不会发生更新。这个状态码通常用于只需要从客户端向服务器发送信息，而不需要返回新信息内容的情况。

##### 4.2.3 206 Partial Content

206 Partial Content状态码表示客户端进行了范围请求，服务器成功执行了这部分GET请求。响应报文中包含由Content-Range指定的范围的实体内容。

#### 4.3 3XX 重定向

3XX系列的状态码表示浏览器需要执行某些特殊的处理以正确处理请求。

##### 4.3.1 301 Moved Permanently

![301](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103091440235.png)

301 Moved Permanently状态码表示请求的资源已被分配了新的URI，以后应使用资源现在所指的URI。例如，如果请求URI的最后忘记添加斜杠“/”，就会产生301状态码。已保存为书签的URI应按Location首部字段提示的新URI重新保存。

##### 4.3.2 302 Found

![302](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103091454641.png)

302 Found状态码表示请求的资源已被分配了新的URI，希望用户（本次）能使用新的URI访问。与301状态码相似，但302状态码代表的资源不是永久移动，只是临时性质的。已移动的资源对应的URI将来还有可能发生改变。

##### 4.3.3 303 See Other

303 See Other状态码表示由于请求对应的资源存在另一个URI，应使用GET方法定向获取请求的资源。虽然303状态码和302 Found状态码功能相同，但303状态码明确表示客户端应当采用GET方法获取资源。

##### 4.3.4 304 Not Modified

304 Not Modified状态码表示客户端发送附带条件的请求时，服务器端允许请求访问资源，但未满足条件。304状态码返回时不包含任何响应的主体部分，虽然被划分在3XX类别中，但与重定向无关。

##### 4.3.5 307 Temporary Redirect

307 Temporary Redirect状态码与302 Found有着相同的含义，但不会从POST变成GET。尽管302标准禁止POST变换成GET，但实际使用时大家并不遵守。307会遵照浏览器标准，不会从POST变成GET，但每种浏览器处理响应时的行为可能有所不同。

#### 4.4 4XX 客户端错误

4XX系列的状态码表示客户端是发生错误的原因所在。

##### 4.4.1 400 Bad Request

![image-20250103091535896](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103091535896.png)

400 Bad Request状态码表示请求报文中存在语法错误。当错误发生时，需修改请求的内容后再次发送请求。浏览器会像200 OK一样对待该状态码。

##### 4.4.2 401 Unauthorized

![401](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103091637145.png)

401 Unauthorized状态码表示发送的请求需要有通过HTTP认证（BASIC认证、DIGEST认证）的认证信息。若之前已进行过一次请求，则表示用户认证失败。返回含有401的响应必须包含一个适用于被请求资源的WWW-Authenticate首部用以质询用户信息。当浏览器初次接收到401响应，会弹出认证用的对话窗口。

##### 4.4.3 403 Forbidden

403 Forbidden状态码表明对请求资源的访问被服务器拒绝了。服务器端没有必要给出拒绝的详细理由，但如果想作说明的话，可以在实体的主体部分对原因进行描述。

##### 4.4.4 404 Not Found

404 Not Found状态码表明服务器上无法找到请求的资源。除此之外，也可以在服务器端拒绝请求且不想说明理由时使用。

#### 4.5 5XX 服务器错误

5XX系列的状态码表示服务器本身发生错误。

##### 4.5.1 500 Internal Server Error

![500](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103091716610.png)

500 Internal Server Error状态码表示服务器端在执行请求时发生了错误。也有可能是Web应用存在的bug或某些临时的故障。

##### 4.5.2 503 Service Unavailable

![image-20250103091734448](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103091734448.png)

503 Service Unavailable状态码表示服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。如果事先得知解除以上状况需要的时间，最好写入RetryAfter首部字段再返回给客户端。

#### 状态码和状况的不一致

不少返回的状态码响应都是错误的，但用户可能察觉不到这点。例如，Web应用程序内部发生错误，状态码依然返回200 OK，这种情况也经常遇到。

