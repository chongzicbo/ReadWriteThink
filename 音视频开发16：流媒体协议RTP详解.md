## RTP数据包

RTP数据包是RTP协议的核心，它负责封装音视频数据，并附加一些必要的信息，如时间戳、序列号等。RTP数据包由两部分组成：**RTP头部**和**RTP负载**。

### RTP Header解析

![RTP Header](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/c6008d4f6caad784799ed9cfeffb63cd.jpg)

RTP头部包含了用于同步、排序和识别数据包的信息。以下是RTP头部的具体字段：

```cpp
/**
 *    0                   1                   2                   3
 *    7 6 5 4 3 2 1 0|7 6 5 4 3 2 1 0|7 6 5 4 3 2 1 0|7 6 5 4 3 2 1 0
 *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 *   |V=2|P|X|  CC   |M|     PT      |       sequence number         |
 *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 *   |                           timestamp                           |
 *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 *   |           synchronization source (SSRC) identifier            |
 *   +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
 *   |            contributing source (CSRC) identifiers             |
 *   :                             ....                              :
 *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 */
struct RtpHeader
{
    /* byte 0 */
    uint8_t csrcLen     : 4; // CSRC计数器，占4位，指示CSRC 标识符的个数。
    uint8_t extension   : 1; // 占1位，如果X=1，则在RTP报头后跟有一个扩展报头。
    uint8_t padding     : 1; // 填充标志，占1位，如果P=1，则在该报文的尾部填充一个或多个额外的八位组，它们不是有效载荷的一部分。
    uint8_t version     : 2; // RTP协议的版本号，占2位，当前协议版本号为2。

    /* byte 1 */
    uint8_t payloadType : 7; // 有效载荷类型，占7位，用于说明RTP报文中有效载荷的类型，如GSM音频、JPEM图像等。
    uint8_t marker      : 1; // 标记，占1位，不同的有效载荷有不同的含义，对于视频，标记一帧的结束；对于音频，标记会话的开始。

    /* bytes 2,3 */
    uint16_t seq; // 占16位，用于标识发送者所发送的RTP报文的序列号，每发送一个报文，序列号增1。接收者通过序列号来检测报文丢失情况，重新排序报文，恢复数据。

    /* bytes 4-7 */
    uint32_t timestamp; // 占32位，时戳反映了该RTP报文的第一个八位组的采样时刻。接收者使用时戳来计算延迟和延迟抖动，并进行同步控制。

    /* bytes 8-11 */
    uint32_t ssrc; // 占32位，用于标识同步信源。该标识符是随机选择的，参加同一视频会议的两个同步信源不能有相同的SSRC。

    /**
     * 标准的RTP Header 还可能存在 0-15个特约信源(CSRC)标识符
     * 每个CSRC标识符占32位，可以有0～15个。每个CSRC标识了包含在该RTP报文有效载荷中的所有特约信源
     */
};

struct RtpPacket
{
    struct RtpHeader rtpHeader;
    uint8_t          payload[0];
};
```



| 字段名              | 长度（位） | 描述                                                         |
| :------------------ | :--------- | :----------------------------------------------------------- |
| 版本（Version）     | 2          | 标识RTP的版本号，目前为2。                                   |
| 填充（Padding）     | 1          | 如果为1，表示数据包末尾有填充字节。填充一个或多个额外的八位组，它们不是有效载荷的一部分 |
| 扩展（X）           | 1          | 如果为1，在RTP报头后跟有一个扩展报头                         |
| CSRC计数（CC）      | 4          | 表示CSRC标识符的数量。                                       |
| 标记（Marker）      | 1          | 不同的有效载荷有不同的含义，对于视频，标记一帧的结束；对于音频，标记会话的开始。 |
| 负载类型（PT）      | 7          | 有效荷载类型，占7位，用于说明RTP报文中有效载荷的类型，如GSM音频、JPEM图像等，在流媒体中大部分是用来区分音频流和视频流，这样便于客户端进行解析。 |
| 序列号（Seq）       | 16         | 占16位，用于标识发送者所发送的RTP报文的序列号，每发送一个报文，序列号增1。这个字段当下层的承载协议用UDP的时候，网络状况不好的时候可以用来检查丢包。当出现网络抖动的情况可以用来对数据进行重新排序。序列号的初始值是随机的，同时音频包和视频包的sequence是分别计数的。 |
| 时间戳（Timestamp） | 32         | 占32位，必须使用90kHZ时钟频率（程序中的90000）。时戳反映了该RTP报文的第一个八位组的采样时刻。接受者使用时戳来计算延迟和延迟抖动，并进行同步控制。可以根据RTP包的时间戳来获得数据包的时序。 |
| SSRC                | 32         | 用于标识同步信源。同步信源是指产生媒体流的信源，他通过RTP报头中的一个32为数字SSRC标识符来标识，而不依赖网络地址，接收者将根据SSRC标识符来区分不同的信源，进行RTP报文的分组。 |
| CSRC列表            | 32×CC      | 贡献源标识符列表，标识参与混合的多个源。可以有0~15个CSRC。每个CSRC标识了包含在RTP报文有效载荷中的所有提供信源。CSRC用来标识对一个RTP混合器产生的新包有贡献的所有RTP包的源。是指当混合器接收到一个或多个同步信源的RTP报文后，经过混合处理产生一个新的组合RTP报文，并把混合器作为组合RTP报文的SSRC，将原来所有的SSRC都作为CSRC传送给接收者，是接受者知道组成组合报文的各个SSRC。 |

- 第一个字节：V、P、X、CC
- 第二个字节：M、PT
- 第三和第四个字节：序列号
- 第5-8个字节：时间戳
- 第9-12个字节：SSRC
- 第13-16个字节：CSRC

#### 示例1

**取一段码流如下：**

![image-20241213212239608](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241213212239608.png)

- 第一个字节 **80** 表示V、P、X、CC
  - 80 转换为二进制：1000 0000，则V是10，P是0，X是0，CC是0000
- 第二个字节 **e0** 表示M、PT
- 第三、四个字节 **00 1e** 表示序列号
- 以此类推

#### 示例2

假设我们有一个音频流，使用AAC编码，RTP头部的具体内容可能如下：

- **版本**：2
- **填充**：0（无填充）
- **扩展**：0（无扩展）
- **CSRC计数**：0（无混合源）
- **标记**：0（无特殊标记）
- **负载类型**：10（AAC编码）
- **序列号**：12345
- **时间戳**：34567890
- **SSRC**：12345678

### RTP Payload(负载)

RTP负载指的是RTP数据包中实际传输的音视频数据。RTP负载是RTP数据包中除了头部信息之外的部分，它包含了实际的音视频数据。RTP协议的主要作用是将音视频数据封装成数据包，并通过网络传输。RTP负载就是这些数据包中实际的音视频内容。

#### RTP负载的作用

RTP负载的作用是传输实际的音视频数据。RTP协议本身并不关心负载的具体内容，它只负责将负载封装成数据包，并附加一些必要的信息（如时间戳、序列号等），以便接收端能够正确解码和播放音视频数据。

- **负载的类型**：RTP负载可以是音频数据（如AAC、PCM）或视频数据（如H.264、VP8）。
- **负载的格式**：负载的格式由RTP头部的**负载类型（Payload Type，PT）**字段决定。

#### RTP负载的类型

RTP负载的类型由RTP头部的**负载类型（PT）**字段指定。不同的负载类型对应不同的音视频编码格式。以下是一些常见的RTP负载类型及其对应的编码格式：

| 负载类型（PT） | 编码格式 | 描述              |
| :------------- | :------- | :---------------- |
| 0              | PCMU     | G.711 μ律音频编码 |
| 8              | PCMA     | G.711 A律音频编码 |
| 10             | AAC      | AAC音频编码       |
| 34             | H.263    | H.263视频编码     |
| 96             | H.264    | H.264视频编码     |
| 97             | VP8      | VP8视频编码       |

## RTP封装H264

### 整体封装流程

将一个 H.264 编码的视频封装成 RTP 协议格式的视频，是一个涉及多个步骤的过程。以下是整体的封装流程，从 H.264 编码数据的生成到最终封装为 RTP 包的全过程。

---

#### 1. **H.264 编码数据的生成**
在封装为 RTP 之前，首先需要对原始视频进行 H.264 编码，生成 H.264 码流。H.264 码流由多个 **NALU（Network Abstraction Layer Unit）** 组成，每个 NALU 包含一个 NAL Header 和 NAL Payload。

##### H.264 编码的输出：
- **NALU 类型**：
  - **SPS（Sequence Parameter Set）**：包含视频序列的全局参数（如分辨率、帧率等）。
  - **PPS（Picture Parameter Set）**：包含图像级别的参数（如量化参数、参考帧配置等）。
  - **IDR 帧**：关键帧，解码器可以从中开始解码。
  - **P 帧**：预测帧，依赖于之前的帧。
  - **SEI（Supplemental Enhancement Information）**：辅助信息，如时间戳、版权信息等。

---

#### 2. **RTP 封装的整体流程**
RTP 封装 H.264 的过程可以分为以下几个步骤：

---

##### 2.1 **步骤 1：NALU 的提取**
- **输入**：H.264 码流。
- **输出**：单个 NALU。
- **过程**：
  - 从 H.264 码流中提取出一个个 NALU。
  - 每个 NALU 包含一个 NAL Header 和 NAL Payload。
  - 根据 NAL Header 中的 `nal_unit_type` 字段，判断 NALU 的类型（如 SPS、PPS、IDR 帧、P 帧等）。

---

##### 2.2 **步骤 2：选择封装模式**
根据 NALU 的大小和网络传输的需求，选择合适的 RTP 封装模式：
1. **Single NALU Mode（单 NALU 模式）**：
   - 每个 RTP 包只封装一个 NALU。
   - 适用于 NALU 较小的情况。
2. **Aggregation Packet Mode（聚合包模式）**：
   - 一个 RTP 包封装多个 NALU。
   - 适用于需要提高传输效率的场景。
3. **Fragmentation Unit Mode（分片单元模式）**：
   - 当 NALU 的大小超过 MTU（最大传输单元，通常为 1500 字节）时，将 NALU 分片，封装到多个 RTP 包中。

---

##### 2.3 **步骤 3：RTP 包的封装**
根据选择的封装模式，将 NALU 封装到 RTP 包中。

###### 2.3.1 **Single NALU Mode（单 NALU 模式）**
- **输入**：单个 NALU。
- **输出**：一个 RTP 包。
- **过程**：
  - 将 NALU 直接放入 RTP 包的负载部分。
  - RTP 头部包含以下信息：
    - **Payload Type**：指定负载类型（如 96 表示 H.264）。
    - **Sequence Number**：序列号，用于排序。
    - **Timestamp**：时间戳，表示帧的时间。
    - **SSRC**：同步源标识符，用于标识数据源。
  - 示例：
    ```
    RTP Header: [Payload Type=96, Sequence Number=123, Timestamp=456, SSRC=789]
    RTP Payload: [NALU Data]
    ```

###### 2.3.2 **Aggregation Packet Mode（聚合包模式）**
- **输入**：多个 NALU。
- **输出**：一个 RTP 包。
- **过程**：
  - 在 RTP 负载中，首先添加一个聚合包头部（如 STAP-A 或 STAP-B）。
  - 每个 NALU 前添加一个长度字段，指示 NALU 的大小。
  - 示例：
    ```
    RTP Header: [Payload Type=97, Sequence Number=123, Timestamp=456, SSRC=789]
    RTP Payload: [STAP-A Header] [NALU1 Length] [NALU1 Data] [NALU2 Length] [NALU2 Data]
    ```

###### 2.3.3 **Fragmentation Unit Mode（分片单元模式）**
- **输入**：单个 NALU。
- **输出**：多个 RTP 包。
- **过程**：
  - 将 NALU 分片，每个分片封装到一个 RTP 包中。
  - 每个 RTP 包的负载包含一个分片单元头部（如 FU-A 或 FU-B）和分片数据。
  - 分片单元头部包含以下字段：
    - `S（Start）`：指示是否为分片的起始部分。
    - `E（End）`：指示是否为分片的结束部分。
    - `Type`：NALU 的类型。
  - 示例：
    ```
    RTP Header: [Payload Type=98, Sequence Number=123, Timestamp=456, SSRC=789]
    RTP Payload: [FU-A Header (S=1, E=0)] [Fragment Data]
    ```

---

##### 2.4 **步骤 4：RTP 包的发送**
- **输入**：封装好的 RTP 包。
- **输出**：通过网络传输的 RTP 包。
- **过程**：
  - 将 RTP 包通过 UDP 协议发送到目标地址。
  - 如果需要传输控制信息（如 RTCP），可以同时发送 RTCP 包。

---

#### 3. **RTP 封装 H.264 的示例**
假设有一个 H.264 视频流，包含以下 NALU：
1. SPS（100 字节）
2. PPS（50 字节）
3. IDR 帧（4000 字节）

##### 封装过程：
1. **SPS 和 PPS**：
   - 使用聚合包模式（STAP-A）封装到一个 RTP 包中。
   - RTP 包结构：
     ```
     RTP Header: [Payload Type=97, Sequence Number=1, Timestamp=0, SSRC=123]
     RTP Payload: [STAP-A Header] [100] [SPS Data] [50] [PPS Data]
     ```

2. **IDR 帧**：
   - 由于 IDR 帧大小为 4000 字节，超过 MTU（1500 字节），使用分片单元模式（FU-A）封装到多个 RTP 包中。
   - 分片过程：
     - 第一个 RTP 包：
       ```
       RTP Header: [Payload Type=98, Sequence Number=2, Timestamp=100, SSRC=123]
       RTP Payload: [FU-A Header (S=1, E=0)] [Fragment Data (1499 bytes)]
       ```
     - 第二个 RTP 包：
       ```
       RTP Header: [Payload Type=98, Sequence Number=3, Timestamp=100, SSRC=123]
       RTP Payload: [FU-A Header (S=0, E=0)] [Fragment Data (1499 bytes)]
       ```
     - 第三个 RTP 包：
       ```
       RTP Header: [Payload Type=98, Sequence Number=4, Timestamp=100, SSRC=123]
       RTP Payload: [FU-A Header (S=0, E=1)] [Fragment Data (1002 bytes)]
       ```

---

#### 4. **总结**
RTP 封装 H.264 的整体流程包括以下步骤：
1. 从 H.264 码流中提取 NALU。
2. 根据 NALU 的大小和网络需求，选择合适的封装模式（单 NALU 模式、聚合包模式或分片单元模式）。
3. 将 NALU 封装到 RTP 包中，并添加 RTP 头部信息。
4. 通过 UDP 协议发送 RTP 包。

通过这种方式，H.264 编码的视频可以高效地封装为 RTP 格式，并通过网络进行实时传输。



## RTP封装AAC





[RTP有效负载(载荷)类型 (RTP Payload Type) - 苦涩的茶 - 博客园](https://www.cnblogs.com/liushui-sky/p/13846456.html)

[RTP封装音视频 | August](https://august295.github.io/posts/RTP封装H264/)

[H264码流格式解析及RTP打包规则整理（转） - 奋斗终生 - 博客园](https://www.cnblogs.com/ajianbeyourself/p/17304143.html)

[H264、H265码流中的SEI NALU | Clay's Blog](https://www.clayhex.com/posts/2024/05/h264-h265-stream-nalu-sei/)