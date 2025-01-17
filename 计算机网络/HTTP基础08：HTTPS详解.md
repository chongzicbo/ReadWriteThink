## 1. HTTP的缺点

1. **通信使用明文可能会被窃听**
    - HTTP 本身缺乏加密功能，其报文以明文形式发送。在 TCP/IP 网络环境下，通信线路上的设备众多且非私有，这使得通信内容极易被恶意窥视。例如，利用抓包工具 Wireshark 就能轻松获取并解析 HTTP 协议的请求和响应内容。
    - 为防止窃听，常用的手段是加密技术。在 HTTP 通信中，可与 SSL 或 TLS 组合使用来加密通信内容，形成 HTTPS；也可直接对 HTTP 报文里所含的内容进行加密，但这种方式下内容仍有被篡改风险，因为它未像 SSL/TLS 那样对整个通信线路加密。
    - ![通信线路加密](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103161909747.png)

    ![通信内容加密](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103161943697.png)

2. **不验证通信方的身份就可能遭遇伪装**

    ![image-20250103162045717](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103162045717.png)

    - HTTP 协议在通信时不确认通信方，任何人都能发起请求，服务器通常也会对接收的请求返回响应（在无访问限制时）。这就导致了诸多隐患，如无法确定服务器是否按真实意图响应、客户端是否为实际接收方、对方是否有访问权限、请求来源以及无法抵御 DoS 攻击等。
    - SSL 协议可解决此问题，它通过证书来确定通信方。证书由第三方机构颁发，伪造难度大。客户端在通信前确认服务器证书，可判断服务器的真实性，减少个人信息泄露风险，同时客户端证书也可用于自身身份确认和网站认证。

    ![证书](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103162147252.png)

3. **无法证明报文完整性，可能已遭篡改**

    ![中间人攻击](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103162223365.png)

    - 由于 HTTP 无法证明报文完整性，在请求或响应传输过程中，若内容被篡改，接收方无法察觉。例如从网站下载文件时，无法确定客户端获取的文件与服务器上的原文件是否一致，这种在传输途中被攻击者拦截篡改的情况称为中间人攻击。
    - 虽有 MD5、SHA - 1 散列值校验和数字签名等方法来防止篡改，但这些方法存在不便和不可靠之处，如需要用户手动检查且无法保证结果正确。而 HTTPS 借助 SSL 的认证、加密和摘要功能，能有效解决报文完整性问题，弥补 HTTP 的不足。 

## 2. HTTP+ 加密 + 认证 + 完整性保护 =HTTPS

### 2.1 HTTP 加上加密处理和认证以及完整性保护后即是 HTTPS

HTTPS（HTTP Secure）是HTTP协议的安全版本，通过在HTTP协议上加入加密处理、身份认证和完整性保护机制，解决了HTTP协议存在的安全问题。以下是HTTPS的核心组成部分及其作用：

![HTTPS](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103164849780.png)

#### 1. **加密处理**

- **问题**：HTTP协议使用明文传输数据，容易被窃听。
- **解决方案**：HTTPS通过SSL/TLS协议对通信内容进行加密，确保数据在传输过程中即使被截获，也无法被破解。加密的对象包括HTTP请求和响应的内容，防止敏感信息（如信用卡号、密码等）泄露。

#### 2. **身份认证**

- **问题**：HTTP协议不验证通信方的身份，可能导致伪装攻击（如中间人攻击）。
- **解决方案**：HTTPS使用数字证书来验证通信方的身份。数字证书由可信的第三方机构（CA）颁发，用于证明服务器或客户端的真实性。客户端通过验证服务器的证书，确保正在通信的服务器是可信的，而不是伪装者。

#### 3. **完整性保护**

- **问题**：HTTP协议无法确保数据在传输过程中是否被篡改。
- **解决方案**：HTTPS通过消息认证码（MAC）来确保数据的完整性。MAC是一种基于密钥的散列函数，用于生成报文的摘要。接收方在收到报文后，会重新计算MAC并与报文中的MAC进行比较，如果一致，则说明报文在传输过程中未被篡改。

### 2.2  HTTPS 是身披 SSL 外壳的 HTTP

![image-20250103165026559](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103165026559.png)

HTTPS 并非是应用层的一种新协议。只是 HTTP 通信接口部分用 SSL（Secure Socket Layer）和 TLS（Transport Layer Security）协议代 替而已。 通常，HTTP 直接和 TCP 通信。当使用 SSL 时，则演变成先和 SSL 通 信，再由 SSL 和 TCP 通信了。简言之，所谓 HTTPS，其实就是身披 SSL 协议这层外壳的 HTTP。



### 2.3 加密技术

#### 共享密钥加密的困境

共享密钥加密（对称密钥加密）使用同一个密钥进行加密和解密。这种方式的主要问题是密钥的安全传输和保管。如果在互联网上传输密钥时被窃听，那么加密就失去了意义。

![共享密钥加密](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103165345992.png)

#### 使用两把密钥的公开密钥加密

公开密钥加密（非对称密钥加密）使用一对非对称的密钥：私有密钥（private key）和公开密钥（public key）。私有密钥不能让其他人知道，而公开密钥可以随意发布。发送方使用接收方的公开密钥加密信息，接收方使用自己的私有密钥解密。这种方式不需要安全传输解密密钥，解决了共享密钥加密的困境。

![非对称加密](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103165458540.png)

#### HTTPS 采用混合加密机制

HTTPS采用共享密钥加密和公开密钥加密两者并用的混合加密机制。在交换密钥环节使用公开密钥加密方式，之后的通信阶段则使用共享密钥加密方式。公开密钥加密处理起来比共享密钥加密方式更为复杂，效率较低，因此在通信时主要使用共享密钥加密方式。

![混合加密](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103165650853.png)

### 2.4 证明公开密钥正确性的证书

![证书](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103165920732.png)

在HTTPS通信中，公开密钥加密（Public-key cryptography）是一种常用的加密方式，但它存在一个关键问题：如何确保通信双方所使用的公开密钥确实是对方合法拥有的？如果公开密钥在传输过程中被篡改或冒充，那么整个加密通信的安全性就会受到威胁。

为了解决这个问题，引入了数字证书（Digital Certificate）的概念。数字证书是由一个受信任的第三方机构（称为证书颁发机构，Certificate Authority，CA）颁发的电子文档，用于验证公开密钥的合法性。数字证书包含了以下几个关键信息：

1. **公开密钥**：证书中包含了服务器或客户端的公开密钥。
2. **证书持有者信息**：包括证书持有者的名称、组织、地理位置等信息。
3. **颁发者信息**：即证书颁发机构的名称和标识。
4. **有效期**：证书的有效起始日期和终止日期。
5. **数字签名**：由证书颁发机构使用其私有密钥对证书内容进行加密生成的签名。

当客户端接收到服务器发送的数字证书时，可以通过以下步骤验证证书的合法性：

1. **检查证书颁发者**：客户端首先检查证书中的颁发者信息，确认该证书是由一个受信任的CA颁发的。浏览器通常内置了一些受信任的CA列表，称为根证书（Root Certificate）。
2. **验证数字签名**：客户端使用CA的公开密钥（存储在浏览器的根证书中）来验证证书中的数字签名。如果签名验证成功，说明证书内容未被篡改，且确实是由该CA颁发的。
3. **检查有效期**：客户端还需要检查证书的有效期，确保当前时间在证书的有效起始日期和终止日期之间。

通过上述步骤，客户端可以确认所使用的公开密钥确实是服务器合法拥有的，从而保证加密通信的安全性。

#### 证书的类型

除了用于验证服务器公开密钥的证书外，还有其他类型的证书：

1. **客户端证书**：用于验证客户端的身份。客户端证书通常用于需要双向认证的场景，如银行的网上银行系统。客户端证书需要用户自行安装，并且通常是付费购买的。
2. **自签名证书**：由组织或个人自行颁发的证书，不经过任何第三方CA的验证。自签名证书虽然可以用于加密通信，但由于缺乏第三方CA的信任背书，浏览器会显示警告信息，提示用户无法确认连接的安全性。

#### 证书颁发机构的信誉

SSL机制中介入认证机构之所以可行，是因为建立在其信用绝对可靠这一大前提下的。然而，证书颁发机构的信誉问题仍然存在。例如，2011年荷兰的DigiNotar认证机构遭到黑客入侵，导致google.com和twitter.com等网站的伪造证书事件。这一事件严重影响了SSL的可信度，因为伪造证书上有正规认证机构的数字签名，浏览器会判定该证书是正当的。

#### 证书吊销机制

即使证书是由受信任的CA颁发的，也存在证书被吊销的可能性。例如，证书颁发机构可能会在发现证书持有者的私钥泄露后吊销该证书。浏览器可以通过证书吊销列表（Certificate Revocation List，CRL）或在线证书状态协议（Online Certificate Status Protocol，OCSP）来检查证书是否被吊销。

总之，数字证书在HTTPS通信中起着至关重要的作用，通过验证公开密钥的合法性，确保了加密通信的安全性。然而，证书颁发机构的信誉和证书吊销机制仍然是需要关注的问题。

### 2.5　HTTPS 的安全通信机制

![HTTPS通信](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20250103170039570.png)

- 步骤 1： 客户端通过发送 Client Hello 报文开始 SSL 通信。报文中包 含客户端支持的 SSL 的指定版本、加密组件（Cipher Suite）列表（所 使用的加密算法及密钥长度等）
- 步骤 2： 服务器可进行 SSL 通信时，会以 Server Hello 报文作为应答。和客户端一样，在报文中包含 SSL 版本以及加密组件。服务器的 加密组件内容是从接收到的客户端加密组件内筛选出来的。 
- 步骤 3： 之后服务器发送 Certificate 报文。报文中包含公开密钥证 书。 
- 步骤 4： 最后服务器发送 Server Hello Done 报文通知客户端，最初阶 段的 SSL 握手协商部分结束。 
- 步骤 5： SSL 第一次握手结束之后，客户端以 Client Key Exchange 报 文作为回应。报文中包含通信加密中使用的一种被称为 Pre-master secret 的随机密码串。该报文已用步骤 3 中的公开密钥进行加密。 
- 步骤 6： 接着客户端继续发送 Change Cipher Spec 报文。该报文会提 示服务器，在此报文之后的通信会采用 Pre-master secret 密钥加密。
-  步骤 7： 客户端发送 Finished 报文。该报文包含连接至今全部报文的 整体校验值。这次握手协商是否能够成功，要以服务器是否能够正确 解密该报文作为判定标准。 
- 步骤 8： 服务器同样发送 Change Cipher Spec 报文。 
- 步骤 9： 服务器同样发送 Finished 报文。 
- 步骤 10： 服务器和客户端的 Finished 报文交换完毕之后，SSL 连接 就算建立完成。当然，通信会受到 SSL 的保护。从此处开始进行应用 层协议的通信，即发送 HTTP 请求。
-  步骤 11： 应用层协议通信，即发送 HTTP 响应。 
- 步骤 12： 最后由客户端断开连接。断开连接时，发送 close_notify 报 文。上图做了一些省略，这步之后再发送 TCP FIN 报文来关闭与 TCP 的通信。

## 3. **HTTPS的优势**

- **安全性**：HTTPS通过加密、认证和完整性保护，有效防止了数据窃听、伪装和篡改。
- **用户信任**：使用HTTPS的网站会在浏览器地址栏显示锁标志，增加了用户对网站的信任。

## 4. **HTTPS的挑战**

- **性能开销**：HTTPS的加密和解密过程会消耗更多的CPU和内存资源，导致通信速度变慢。
- **成本**：HTTPS需要购买数字证书，对于一些小型网站或个人网站来说，成本可能较高。

## 5. HTTPS的性能优化

虽然HTTPS提供了更高的安全性，但其加密和解密过程会带来额外的性能开销。为了优化HTTPS的性能，可以采用以下措施：

- **使用SSL加速器**：专用的硬件设备，用于加速SSL/TLS的计算过程。
- **会话恢复**：通过会话ID或会话票证（Session Ticket）来减少握手过程的重复，提高连接建立的速度。
- **HTTP/2**：HTTP/2协议在HTTPS上运行，通过多路复用和头部压缩等技术，进一步提升性能。