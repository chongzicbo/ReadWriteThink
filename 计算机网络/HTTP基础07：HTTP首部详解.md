## 1. HTTP报文首部

![HTTP报文结构](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103120810958.png)



- HTTP 协议的请求和响应报文均包含首部，为客户端和服务器处理请求与响应提供重要信息。请求报文由方法、URI、HTTP 版本、首部字段等构成；响应报文由 HTTP 版本、状态码、首部字段构成。

- 首部字段由首部字段名和字段值组成，中间用冒号分隔，部分字段可有多个值。当出现重复首部字段名时，浏览器处理逻辑不同。

![请求报文](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103121055993.png)

![响应报文](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103121122872.png)

## 2. HTTP首部字段

1. HTTP 首部字段的重要性与结构

   - HTTP 首部字段是 HTTP 报文的关键要素，在客户端和服务器通信中传递重要信息，如报文主体大小、语言、认证信息等。
   - 其由首部字段名和字段值构成，用冒号 “:” 分隔，如 Content-Type:text/html，部分字段可有多个值，如 Keep-Alive:timeout=15,max=100。若出现重复首部字段名，浏览器处理逻辑不同。

2. HTTP 首部字段的类型

   - **通用首部字段**：在请求和响应报文两方都会使用，如 Cache-Control、Connection 等，用于控制缓存、连接管理等多种功能。
   - **请求首部字段**：客户端向服务器发送请求报文时使用，补充请求附加内容、客户端信息及响应优先级等，像 Accept 用于告知可处理的媒体类型。
   - **响应首部字段**：服务器返回响应报文时使用，补充响应附加内容并要求客户端附加信息，例如 Accept-Ranges 表明是否接受字节范围请求。
   - **实体首部字段**：针对请求和响应报文的实体部分，补充资源更新时间等与实体有关的信息，如 Content-Encoding 说明实体主体适用的编码方式。

3. HTTP/1.1 首部字段一览

   ![通用首部字段](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103121459064.png)

   - **通用首部字段**：定义了 Cache-Control、Connection、Date 等 9 种，如 Cache-Control 可控制缓存行为，Connection 用于连接管理和控制首部字段转发。

   | 请求首部字段名      | 说明                                        |
   | ------------------- | ------------------------------------------- |
   | Accept              | 用户代理可处理的媒体类型                    |
   | Accept-Charset      | 优先的字符集                                |
   | Accept-Encoding     | 优先的内容编码                              |
   | Accept-Language     | 优先的语言(自然语言)                        |
   | Authorization       | Web认证信息                                 |
   | Expect              | 期待服务器的特定行为                        |
   | From                | 用户的电子邮箱地址                          |
   | Host                | 请求资源所在服务器                          |
   | If-Match            | 比较实体标记(ETag)                          |
   | If-Modified-Since   | 比较资源的更新时间                          |
   | If-None-Match       | 比较实体标记(与If-Match相反)                |
   | If-Range            | 资源未更新时发送实体Byte的范围请求          |
   | If-Unmodified-Since | 比较资源的更新时间(与If-Modified-Since相反) |
   | Max-Forwards        | 最大传输逐跳数                              |
   | Proxy-Authorization | 代理服务器要求客户端的认证信息              |
   | Range               | 实体的字节范围请求                          |
   | Referer             | 对请求中URI的原始获取方                     |
   | TE                  | 传输编码的优先级                            |
   | User-A gent         | HTTP客户端程序的信息                        |

   - **请求首部字段**：有 Accept、Accept-Charset 等 20 种，如 Accept-Charset 可通知服务器用户代理支持的字符集及优先级。

   | 响应首部字段名     | 说明                         |
   | ------------------ | ---------------------------- |
   | Accept-Ranges      | 是否接受字节范围请求         |
   | Age                | 推算资源创建经过时间         |
   | ETag               | 资源的匹配信息               |
   | Location           | 令客户端重定向至指定URI      |
   | Proxy-Authenticate | 代理服务器对客户端的认证信息 |
   | Retry-After        | 对再次发起请求的时机要求     |
   | Server             | HTTP服务器的安装信息         |
   | Vary               | 代理服务器缓存的管理信息     |
   | WWW-Authenticate   | 服务器对客户端的认证信息     |

   - **响应首部字段**：包含 Accept-Ranges、Age 等 10 种，如 Age 能告知客户端源服务器创建响应的时间。

     ![实体首部字段](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103121651401.png)

   - **实体首部字段**：包括 Allow、Content-Encoding 等 10 种，如 Content-Length 表明实体主体大小。

4. **非 HTTP/1.1 首部字段**：HTTP 通信中还有 Cookie、Set-Cookie 和 Content-Disposition 等在其他 RFC 中定义的首部字段，使用频率高，统一归纳在 RFC4229 中。

5. End-to-end 首部和 Hop-by-hop 首部

   - **端到端首部**：会转发给最终接收目标，且必须保存在缓存生成的响应中并被转发，除特定 8 个首部字段外的其他字段都属于此类。
   - **逐跳首部**：只对单次转发有效，通过缓存或代理后不再转发，使用时需提供 Connection 首部字段，如 Connection、Keep-Alive 等 8 个字段属于此类。

![逐条首部](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103121846585.png)

## 3. HTTP通用首部字段

### 3.1 Cache-Control

通过指定首部字段 Cache-Control 的指令，就能操作缓存的工作机制。

![Cache-Control](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103134130111.png)

指令的参数是可选的，多个指令之间通过“,”分隔。首部字段 CacheControl 的指令可用于请求及响应时。

#### 缓存请求指令
| 指令             | 参数   | 说明                                                         |
| ---------------- | ------ | ------------------------------------------------------------ |
| no-cache         | 无     | 强制向源服务器再次验证，客户端发送此指令时不接收缓存响应，缓存服务器需转发请求给源服务器 |
| no-store         | 无     | 不缓存请求或响应内容，适用于含机密信息场景                   |
| max-age =[秒]    | 必需   | 指定响应最大 Age 值，客户端依此判断是否接收缓存资源，为 0 时缓存服务器常转发请求 |
| max-stale(=[秒]) | 可省略 | 可接收过期响应，未指定参数值时无条件接收，指定后在过期但不超该时长内也可接收 |
| min-fresh=[秒]   | 必需   | 期望在指定时间内响应有效，超期资源不能作为返回               |
| no-transform     | 无     | 禁止代理更改媒体类型，如阻止压缩图片操作                     |
| only-if-cached   | 无     | 仅当目标资源在缓存服务器本地有缓存时才要求返回，否则返回 504 |
| cache-extension  | 无     | 新指令标记，用于扩展指令集，不被理解时会被缓存服务器忽略     |

![max-age指令](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103134837909.png)

#### 缓存响应指令

| 指令             | 参数   | 说明                                                         |
| ---------------- | ------ | ------------------------------------------------------------ |
| public           | 无     | 允许任意方使用缓存，其他用户可利用缓存资源                   |
| private          | 可省略 | 响应仅针对特定用户缓存，与 public 相反，缓存服务器对其他用户不返回此缓存 |
| no-cache         | 可省略 | 缓存前需确认有效性，服务器返回此指令时缓存服务器不能缓存资源且源服务器不再确认 |
| no-store         | 无     | 同缓存请求指令，不缓存请求或响应内容                         |
| no-transform     | 无     | 同缓存请求指令，禁止代理更改媒体类型                         |
| must-revalidate  | 无     | 可缓存但必须向源服务器再次确认，否则返回 504 给客户端        |
| proxy-revalidate | 无     | 要求中间缓存服务器再次确认缓存有效性                         |
| max-age =[秒]    | 必需   | 定义响应最大 Age 值，缓存服务器依此管理缓存，HTTP/1.1 优先处理此指令而非 Expires 首部字段 |
| s-maxage =[秒]   | 必需   | 专为公共缓存服务器设置的最大 Age 值，公共缓存服务器使用后忽略 Expires 和 max-age 指令 |
| cache-extension  | 无     | 新指令标记，功能同缓存请求指令中的相应标记                   |

### 3.2 Connection

- **控制不再转发给代理的首部字段**：在客户端发送请求和服务器返回响应过程中，通过 Connection 首部字段可指定不再转发给代理的首部字段（即 Hop-by-hop 首部）。例如，若客户端发送请求包含 “Connection: Upgrade”，则代理服务器会在转发请求给源服务器前删除 Upgrade 首部字段，确保某些首部信息仅在特定的相邻节点间传递，防止不必要的信息扩散到后续的代理环节，以此精确控制首部字段在代理链中的传播范围，保障通信的安全性和有效性。

![Connection](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103135247177.png)

- **管理持久连接**：在 HTTP/1.1 版本中，默认连接为持久连接，这使得客户端能够在同一连接上连续发送多个请求，减少连接建立和关闭的开销，提高通信效率。然而，当服务器端希望结束连接时，会指定 Connection 首部字段的值为 Close，如 “Connection: close”，向客户端表明此次通信后将关闭连接。而在 HTTP/1.1 之前的版本，默认连接是非持久连接，若要在这些旧版本协议上维持持续连接，则需要在请求中明确指定 Connection 首部字段的值为 Keep-Alive，如 “Connection: Keep-Alive”，服务器在响应时也会添加相应的 Keep-Alive 首部字段及 Connection 首部字段，以实现连接的持续维持，确保在不同 HTTP 版本环境下都能根据需求灵活管理连接的持续性。

![管理持久连接](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103135336245.png)

### 3.3 Date

- 表明创建 HTTP 报文的日期和时间，HTTP/1.1 协议使用 RFC1123 规定的格式（如 Tue,03Jul 201204:40:59GMT），之前版本使用不同格式，还有与 C 标准库 asctime () 函数输出格式一致的形式，用于准确记录报文创建时间。

> Date: Tue, 03 Jul 2012 04:40:59 GMT

> Date: Tue, 03-Jul-12 04:40:59 GMT

> Date: Tue Jul 03 04:40:59 2012

### 3.4 Pragma

- 是 HTTP/1.1 之前版本的遗留字段，用于与 HTTP/1.0 向后兼容，仅在客户端请求中使用，规范形式为 Pragma:no-cache，常与 Cache-Control:no-cache 同时出现，以确保中间服务器不返回缓存资源，弥补无法确定中间服务器 HTTP 协议版本的问题。

### 3.5 Trailer

- 用于说明在报文主体后记录的首部字段，应用于 HTTP/1.1 分块传输编码时，如指定 Trailer:Expires 后，在报文主体之后会出现 Expires 首部字段，方便在分块传输中传递重要的首部信息。

> HTTP/1.1 200 OK
> Date: Tue, 03 Jul 2012 04:40:56 GMT
> Content-Type: text/html
> ...
> Transfer-Encoding: chunked
> Trailer: Expires
> ...(报文主体)...
> 0
> Expires: Tue, 28 Sep 2004 23:59:59 GMT

以上用例中，指定首部字段 Trailer 的值为 Expires，在报文主体之后 （分块长度 0 之后）出现了首部字段 Expires。

### 3.6 Transfer-Encoding

首部字段 Transfer-Encoding 规定了传输报文主体时采用的编码方式。 HTTP/1.1 的传输编码方式仅对分块传输编码有效。

> HTTP/1.1 200 OK
> Date: Tue, 03 Jul 2012 04:40:56 GMT
> Cache-Control: public, max-age=604800
> Content-Type: text/javascript; charset=utf-8
> Expires: Tue, 10 Jul 2012 04:40:56 GMT
> X-Frame-Options: DENY
> X-XSS-Protection: 1; mode=block
> Content-Encoding: gzip
> **Transfer-Encoding: chunked**
> Connection: keep-alive
> cf0 ←16进制(10进制为3312)
> ...3312字节分块数据...
> 392 ←16进制(10进制为914)
> ...914字节分块数据...
> 0

### 3.7 Upgrade

- 用于检测 HTTP 协议及其他协议是否可升级通信，指定参数值可切换协议，如指定 Upgrade:TLS/1.0 并配合 Connection:Upgrade，服务器可用 101 Switching Protocols 状态码响应，实现协议升级功能，且作用仅限于客户端和邻接服务器之间。

![Upgrade字段](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103135937501.png)

### 3.8 Via

![Via字段](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103140101073.png)

- 用于追踪客户端与服务器之间请求和响应报文的传输路径，报文经过代理或网关时附加服务器信息，可避免请求回环，如示例中展示了经过多个代理服务器时 Via 首部如何记录服务器信息，常与 TRACE 方法配合使用，增强对网络传输路径的监控能力。

### 3.9 Warning

- 通常告知用户与缓存相关问题的警告，格式为 warning:[警告码][警告主机：端口号]"[警告内容]"([日期时间])，HTTP/1.1 定义了 7 种警告，如 110（响应已过期）、111（再验证失败）等，警告码具有扩展性，方便对缓存异常情况进行提示和处理。

![Warning字段](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103140243857.png)

## 4. 请求首部字段

1. **媒体类型与优先级相关字段**
    - **Accept**：通知服务器用户代理能够处理的媒体类型及相对优先级，采用 type/subtype 形式指定，如 text/html、image/jpeg 等，可通过 q 值（0-1）设置权重，服务器优先返回权重高的类型。

    ![Accept](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103140653294.png)

    - **TE**：告知服务器客户端能够处理响应的传输编码方式及优先级，类似 Accept-Encoding，但针对传输编码，还可指定分块传输编码及 trailers。
2. **字符集与编码相关字段**
    - **Accept-Charset**：用来通知服务器用户代理支持的字符集及优先顺序，可一次性指定多种字符集，应用于内容协商机制的服务器驱动协商，也可用 q 值表示优先级。

    ![Accept-Charset](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103140731796.png)

    - **Accept-Encoding**：告知服务器用户代理支持的内容编码及优先级顺序，如 gzip、deflate 等，支持通配符 *，同样用 q 值表示优先级。

    ![Accept-Encoding](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103140800502.png)
3. **语言与认证相关字段**
    - **Accept-Language**：告知服务器用户代理能够处理的自然语言集及相对优先级，如 zh-cn、en-us 等，按权重值 g 来表示优先级，服务器据此返回相应语言资源。
    - **Authorization**：在用户代理需要通过服务器认证时使用，用于告知服务器认证信息（证书值），通常在收到 401 状态码响应后添加到请求中，缓存对其处理存在差异。
4. **其他重要请求首部字段**
    - **Expect**：客户端用此告知服务器期望出现的特定行为，如 100-continue，若服务器无法回应期望则返回 417 Expectation Failed。
    - **From**：告知服务器使用用户代理的用户的电子邮箱地址，常用于显示搜索引擎等用户代理负责人的联系方式，使用代理时应尽量包含。
    - **Host**：必须包含在 HTTP/1.1 请求内，告知服务器请求资源所处的互联网主机名和端口号，与虚拟主机工作机制密切相关，确保服务器能准确识别请求对应的域名。
    - **If-Match**：属条件请求字段，告知服务器匹配资源所用的实体标记（ETag）值，服务器比对后决定是否执行请求，可使用 * 忽略 ETag 值，仅支持强 ETag 值。
    - **If-Modified-Since**：同样是条件请求字段，若字段值早于资源更新时间，服务器处理请求，否则返回 304 Not Modified，用于确认本地资源有效性，依据 Last-Modified 确定更新时间。
    - **If-None-Match**：与 If-Match 作用相反，指定的实体标记（ETag）值与请求资源的 ETag 不一致时，告知服务器处理请求，常用于获取最新资源。
    - **If-Range**：条件请求字段，若指定的字段值（ETag 值或时间）和请求资源的 ETag 值或时间一致时，作为范围请求处理，否则返回全体资源，可减少处理步骤。
    - **If-Unmodified-Since**：与 If-Modified-Since 相反，指定资源在字段值日期时间后未更新时才处理请求，否则返回 412 Precondition Failed。
    - **Max-Forwards**：在使用 TRACE 或 OPTIONS 方法发送请求时，指定可经过的服务器最大数目，服务器转发前减 1，为 0 时返回响应，有助于调查请求转发问题。
    - **Proxy-Authorization**：当客户端收到代理服务器的认证质询时，用此告知服务器认证所需信息，与客户端和服务器之间的认证类似，但发生在客户端与代理之间。
    - **Range**：用于范围请求，告知服务器资源的指定字节范围，如 bytes = 5001-10000，服务器根据情况返回 206 Partial Content 或 200 OK 及全部资源。
    - **Referer**：告知服务器请求的原始资源的 URI，但在浏览器地址栏输入 URI 或出于安全考虑时可能不发送，防止原始资源 URI 中保密信息泄露。
    - **User-Agent**：传达创建请求的浏览器和用户代理名称等信息，网络爬虫可能添加作者邮箱，请求经过代理时可能被添加代理服务器名称。 

## 5. 响应首部字段

1. **缓存与范围请求相关字段**
    - **Accept-Ranges**：告知客户端服务器是否能处理范围请求，可取值为 bytes 表示能处理，none 表示不能处理，使客户端了解服务器对部分资源获取的支持情况。
    
    ![Accept-Ranges](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103141358996.png)
    
    - **Age**：能告知客户端源服务器创建响应的时间或缓存后再次认证的时间，单位为秒，代理创建响应时必须添加该字段，帮助客户端判断响应的时效性。
2. **资源标识与重定向字段**
    - **ETag**：为资源提供唯一标识，以字符串形式呈现，资源更新时 ETag 值随之改变，且有强、弱 ETag 值之分，强 ETag 值对实体细微变化敏感，弱 ETag 值仅在资源根本改变时更新，有助于客户端识别资源状态。
    - **Location**：主要配合 3xx 重定向响应，将响应接收方引导至与请求 URI 不同的资源，大多数浏览器会自动访问重定向资源，实现页面跳转等功能。
    
    ![Location](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103141313562.png)
3. **认证与等待时间字段**
    - **Proxy-Authenticate**：由代理服务器向客户端发送认证信息，其认证过程与客户端和服务器之间的 HTTP 访问认证类似，但发生在客户端与代理之间，促使客户端提供认证信息以获取资源。
    - **Retry-After**：告知客户端应在多久之后再次发送请求，常配合 503 Service Unavailable 响应或 3xx Redirect 响应使用，字段值可指定为具体日期时间或创建响应后的秒数，指导客户端合理安排请求时间。
    
    ![Retry](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103141238612.png)
4. **服务器信息与缓存控制字段**
    - **Server**：告知客户端当前服务器上安装的 HTTP 服务器应用程序信息，包括软件名称、版本号和可能的安装可选项，让客户端了解服务器的基本情况。
    
    ![Server](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103141219136.png)
    
    - **Vary**：用于控制缓存，源服务器通过该字段向代理服务器传达本地缓存使用方法的命令，代理服务器根据请求中 Vary 指定首部字段的值决定是否返回缓存，确保缓存的准确性。
    
    ![Vary](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103141125815.png)
5. **访问认证字段**：**WWW-Authenticate**：在 HTTP 访问认证中发挥关键作用，告知客户端适用于访问请求 URI 所指定资源的认证方案（如 Basic 或 Digest）和带参数提示的质询，在 401 Unauthorized 响应中必定出现，引导客户端进行认证操作。 

## 6. 实体首部字段

实体首部字段是包含在请求报文和响应报文中的实体部分所使用的首 部，用于补充内容的更新时间等与实体相关的信息。

![实体首部字段](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103141547741.png)

1. **资源方法与编码相关字段**
    - **Allow**：通知客户端能够支持 Request-URI 指定资源的所有 HTTP 方法。当服务器接收到不支持的 HTTP 方法时，会以状态码 405 Method Not Allowed 响应，并在该首部字段中列出所有支持的方法，确保客户端了解资源的可用操作方式。
    
    ![Allow](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103141722084.png)
    
    - **Content-Encoding**：告知客户端服务器对实体主体部分选用的内容编码方式，如常见的 gzip、deflate 等，在不丢失实体信息的前提下进行压缩，接收方需根据此信息进行相应解压操作。
2. **语言、大小与位置相关字段**
    - **Content-Language**：明确告知客户端实体主体使用的自然语言，如中文（zh-CN）等，方便客户端正确处理和展示内容。
    - **Content-Length**：精确表明实体主体部分的大小，单位是字节，在实体主体未进行内容编码传输时使用，为客户端接收和处理数据提供重要参考，但内容编码传输时不适用。
    - **Content-Location**：给出与报文主体部分相对应的 URI，与 Location 首部字段不同，它主要表示报文主体返回资源对应的 URI，在某些特定请求（如使用首部字段 Accept-Language 的服务器驱动型请求）中，当返回页面内容与实际请求对象不同时，会在此字段写明 URI。
3. **完整性与范围相关字段**
    - **Content-MD5**：是通过 MD5 算法生成并经 Base64 编码的值，用于检查报文主体在传输过程中是否保持完整以及确认传输到达。客户端会对接收的报文主体执行相同 MD5 算法并与该字段值比较，但此方法无法检测恶意篡改，因为若内容被篡改，Content-MD5 也可能被重新计算篡改。
    
    ![Content-MD5](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103141853872.png)
    
    - **Content-Range**：针对范围请求，在返回响应时使用，能告知客户端作为响应返回的实体的哪个部分符合范围请求，字段值以字节为单位，表示当前发送部分及整个实体大小，使客户端准确获取所需部分资源。
4. **媒体类型与时间相关字段**
    - **Content-Type**：详细说明实体主体内对象的媒体类型，采用 type/subtype 形式赋值，如 text/html，并可通过参数 charset 指定字符集，如 UTF-8，让客户端了解数据的类型和编码格式。
    - **Expires**：将资源失效的日期告知客户端，缓存服务器在该日期之前会以缓存应答请求，超过指定时间后，缓存服务器会转向源服务器请求资源。若源服务器不希望缓存，可设置与首部字段 Date 相同的时间值，且当首部字段 Cache-Control 有指定 max-age 指令时，优先处理 max-age 指令。
    - **Last-Modified**：指明资源最终修改的时间，一般是 Request-URI 指定资源被修改的时间，但在某些动态数据处理（如使用 CGI 脚本）时，可能变为数据最终修改的时间，帮助客户端判断资源的新旧程度。 

## 7. 为Cookie服务的首部字段

![Cookie](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103142118157.png)

1. **Set-Cookie 首部字段**
    - 当服务器准备管理客户端状态时使用，其字段值包含多个重要属性。
    - **NAME = VALUE**：为必需项，赋予Cookie的名称和对应的值，是Cookie的核心标识信息。
    - **expires = DATE**：指定Cookie的有效期。若不明确指定，其有效期仅维持到浏览器关闭前，即会话（Session）时间段内。一旦Cookie从服务器发送至客户端，服务器无法直接删除，但可通过覆盖已过期的Cookie实现对客户端Cookie的实质性删除操作。
    - **path = PATH**：用于限制Cookie的发送范围到服务器上的特定文件目录。若不指定，默认为文档所在的文件目录，但存在可避开此限制的方法，其作为安全机制的效果有限。
    - **domain =域名**：指定Cookie适用的域名，若不指定则默认为创建Cookie的服务器的域名，且可实现与结尾匹配一致，如指定example.com后，www.example.com等相关域名也可发送Cookie，但不指定此属性相对更安全。
    - **Secure**：限制Web页面仅在HTTPS安全连接时才发送Cookie，如Set - Cookie: name = value; secure，在https://www.example.com/（HTTPS）连接时才回收Cookie，http://www.example.com/（HTTP）则不会。省略此属性时，HTTP和HTTPS下都会回收Cookie。
    - **HttpOnly**：使Cookie不能被JavaScript脚本访问，主要用于防止跨站脚本攻击（XSS）对Cookie信息的窃取，如Set - Cookie: name = value; HttpOnly，在主流浏览器（如Internet Explorer 6 SP1以上版本）中已得到支持。
2. **Cookie 首部字段**：客户端在请求中使用，用于告知服务器其从服务器接收到的Cookie信息。当客户端希望获得HTTP状态管理支持时，会在请求中包含从服务器获取的Cookie，若接收到多个Cookie，也可一并以多个Cookie形式发送给服务器，实现服务器与客户端之间的状态关联与交互。 

## 8. 其它首部字段

1. **X-Frame - Options**
    - 属于 HTTP 响应首部，主要用于控制网站内容在其他网站的 Frame 标签内的显示情况。
    - 其可指定两个字段值：DENY 表示拒绝在任何其他网站的 Frame 中显示；SAMEORIGIN 表示仅在同源域名下的页面（Top - level - browsing context）匹配时才许可显示，例如指定 http://hackr.jp/sample.html 页面为 SAMEORIGIN 时，hackr.jp 上的页面 frame 可加载该页面，而其他域名如 example.com 的页面则不行。
    - 目前主流浏览器如 Internet Explorer 8、Firefox3.6.9 +、Chrome 4.1.249.1042 +、Safari4 + 和 Opera 10.50 + 等均支持该首部字段，在 Web 服务器端预先设定好此字段值可有效防止点击劫持（clickjacking）攻击。
2. **X-XSS-Protection**
    - 为 HTTP 响应首部，是针对跨站脚本攻击（XSS）的一种对策。
    - 可指定的字段值有 0 和 1，其中 0 表示将 XSS 过滤设置成无效状态，1 则表示将 XSS 过滤设置成有效状态，通过此设置可控制浏览器的 XSS 防护机制开关，增强网页的安全性。
3. **DNT**
    - 是 HTTP 请求首部，DNT 为 Do Not Track 的简称。
    - 其可指定的字段值为 0 或 1，0 表示同意被追踪，1 表示拒绝被追踪，主要用于表示用户拒绝个人信息被收集，是一种拒绝精准广告追踪的有效方法。由于其功能的有效性，Web 服务器需要对 DNT 进行相应的支持，以尊重用户的隐私选择。
4. **P3P**
    - 属于 HTTP 响应首部，利用 P3P（The Platform for Privacy Preferences，在线隐私偏好平台）技术来保护用户隐私。
    - 要实现 P3P 的功能，需要按照特定步骤进行操作：首先创建 P3P 隐私，接着创建 P3P 隐私对照文件并保存命名为 /w3c/p3p.xml，最后从 P3P 隐私中新建 Compact policies 并输出到 HTTP 响应中。通过这些步骤，可将 Web 网站上的个人隐私转换为仅供程序可理解的形式，从而达到保护用户隐私的目的，具体规范标准可参考 The Platform for Privacy Preferences 1.0 (P3P1.0) Specification（http://www.w3.org/TR/P3P/）。 