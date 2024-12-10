FLV（Flash Video）是一种由Adobe公司推出的流媒体格式，广泛应用于互联网视频传输。它以轻量级、易于封装和播放等特性著称，非常适合在网络上传输音视频内容。

# FLV文件结构

FLV文件总体上由两大部分组成：文件头（Header）和文件体（Body）。文件头部分用于标识文件为FLV类型，并提供有关后续音视频流的基本信息；文件体则由一系列标签（Tag）构成，这些标签包含了实际的音频、视频或脚本数据

![LFV文件](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/1-1.png)

![FLV文件头](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241210110320523.png)

![FLV文件体](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241210110338528.png)

### FLV 文件头（Header）

FLV文件头占用9个字节，其结构如下：

![FLV文件头](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241210110857435.png)

- **Signature**：前三个字节总是“FLV”的ASCII码值（0x46, 0x4C, 0x56），用于识别文件格式。
- **Version**：第四个字节表示FLV版本号，目前固定为1（0x01）。
- **TypeFlags**：第五个字节包含标志位，用来指示该FLV文件是否包含音频流（bit[2]）或视频流（bit[0]），其他位保留且应设为0。
- DataOffset：最后四个字节是一个32位无符号整数，表示从文件头到第一个Tag之间的偏移量，在FLV v1中通常为9。

例如，对于一个同时包含音频和视频的FLV文件，它的头部可能是这样的二进制序列：“46 4C 56 01 05 00 00 00 09”，其中“05”表示存在音频和视频两种类型的tag。

### FLV 文件体（Body）

![FLV文件体](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/1-1-1-3.png)

文件体由多个标签及其大小组成。每个标签之前都有一个4字节的字段`Previous Tag Size`，记录了前一个标签的总长度（包括其自身的11字节头部）。标签本身又分为三类：音频标签（Audio Tag）、视频标签(Video Tag)以及脚本标签(Script Tag)。

每类标签包含两个部分：Tag Header + Tag Data

![image-20241210112245195](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241210112245195.png)

#### Script Tag

##### Tag Header

- TagType为18表示Script Tag
- DataSize=372 表示后面的Tag Data大小为372字节
- TimeStamp: 3个字节，表示当前这一帧视频或者音频的解码时间
- TimestampExtended: 1个字节，表示解码时间扩展，因为 TimeStamp 字段只有 3 字节，如果存不下 dts（解码时间），就需要用 TimeStampExtended 来扩展，TimeStampExtended 会作为最高位的字节。
- StreamId: 位于Tag Header的最后三个字节,是一个24位（3字节）的无符号整数,在实际应用中，这个字段总是被设置为0，并且在大多数情况下并不会被使用。StreamID的设计初衷可能是为了支持多路复用（即在一个FLV文件中同时携带多个独立的音频或视频流），但在FLV的实际使用过程中，这一功能并未得到广泛应用和支持。因此，在标准的FLV文件里，我们看到的所有Tag中的StreamID都固定为0。

##### Tag Data

脚本标签主要用于存储元数据（Metadata），这些信息可以帮助播放器更好地理解和处理FLV文件。`Tag Data`通常包括两个AMF（Action Message Format）包：

- 第一个AMF包是一个字符串，通常是固定的字符串`"onMetaData"`，用来标识这是一个元数据标签。
- 第二个AMF包是一个数组，里面包含了各种键值对形式的参数，如`duration`（持续时间）、`width`（宽度）、`height`（高度）等。这些参数提供了有关视频的基本信息，有助于播放器进行预加载和初始化设置

`Tag Data` 的字段不是固定的，而是根据 `Tag Header` 里面的 `TagType` 决定的。

对于Script Tag来说，`Tag Data` 里面全部都是 `AMF` 包，`AMF` 全称是 Action Message Format（信息表）。如下图中的AMF1和AMF2

![1-2](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/1-2.png)

上图中的 `Metadata` 也是属于 `AMF2` 包的。

`AMF1` 的 `type` 等于 2，代表这是一个 **`String`** 包，String size 等于 10，代表这个字符串是 10 个字节，而 `onMetaData` 刚好就是 10 字节了。

`AMF2` 的 `type` 等于 8，代表这是一个 **数组** 包，metadata count 等于 16，代表这个数组的长度是 16，可以看到 `MetaData` 里面刚好有 16 个 Key-value 键值对。

- **AMF2 Metadata Count**: 16 - 表示有16个元数据项。
- Metadata:
  - **duration**: 30.03000 - 视频的总时长。
  - **width**: 1920.00000 - 视频的宽度。
  - **height**: 1080.00000 - 视频的高度。
  - **videodatarate**: 1947.37012 - 视频的数据速率（比特率）。
  - **framerate**: 23.97600 - 视频的帧率。
  - **videocodecid**: 7.00000 - 视频编解码器ID，7表示H.264/AVC。
  - **audiodatarate**: 123.70117 - 音频的数据速率（比特率）。
  - **audiosamplerate**: 48000.00000 - 音频的采样率。
  - **audiosamplesize**: 16.00000 - 音频的样本大小（位深度）。
  - **stereo**: 1 - 表示音频是立体声。
  - **audiocodecid**: 10.00000 - 音频编解码器ID，10表示AAC。
  - **minor_version**: 512 - FLV文件的次要版本号。
  - **compatible_brands**: "isomiso2avcimp41" - 兼容的品牌标识符。
  - **encoder**: "Lavf58.76.100" - 编码器信息。
  - **filesize**: 8920570.00000 - 文件的大小（以字节为单位）。

这个 key-value 键值对的解析规则是这样的，key 的类型一定是字符串，前面 2 个字节代表字符串的长度，如下：

![image-20241210114639536](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241210114639536.png)

08 是 duration 的长度，05 是 width 的长度，06 是 height 的长度，以此类推。

```
typedef enum {
    AMF_DATA_TYPE_NUMBER      = 0x00,
    AMF_DATA_TYPE_BOOL        = 0x01,
    AMF_DATA_TYPE_STRING      = 0x02,
    AMF_DATA_TYPE_OBJECT      = 0x03,
    AMF_DATA_TYPE_NULL        = 0x05,
    AMF_DATA_TYPE_UNDEFINED   = 0x06,
    AMF_DATA_TYPE_REFERENCE   = 0x07,
    AMF_DATA_TYPE_MIXEDARRAY  = 0x08,
    AMF_DATA_TYPE_OBJECT_END  = 0x09,
    AMF_DATA_TYPE_ARRAY       = 0x0a,
    AMF_DATA_TYPE_DATE        = 0x0b,
    AMF_DATA_TYPE_LONG_STRING = 0x0c,
    AMF_DATA_TYPE_UNSUPPORTED = 0x0d,
} AMFDataType;
```

如果 `value` 的 `type` 是 `AMF_DATA_TYPE_NUMBER`，那 `value` 就占 8 字节，存储的是浮点数。

如果 `value` 的 `type` 是 `AMF_DATA_TYPE_BOOL`，那 `value` 就占 1 个字节，存储的是 0 或者 1。

如果 `value` 的 `type` 是 `AMF_DATA_TYPE_STRING`，那 `value` 就占 x 个字节，xxx

总之，`value` 的字节大小是由 `type` 决定的，具体代码解析在《flv_read_packet读取AVPacket》有讲解。

#### Video Tag

![1-5](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/1-5.png)

##### Tag Header

- Tag Type: 为9表示Video Tag
- DataSize: 表示Video Tag Data的数据大小
- 其它字段同Script



##### Tag Data

`Tag Data` 包含了视频流的具体信息。这部分数据对于正确解码和播放视频至关重要。根据FLV协议的定义，每个视频标签的数据部分从第二个字节开始为视频流数据。第一个字节用于描述视频帧的基本属性。

###### 第一个字节：视频帧类型与编码格式

- 前4位（Frame Type）

  ：标识当前帧的类型。

  - `1` 表示关键帧（Key Frame），即I帧，可以独立解码；
  - `2` 表示非关键帧（Inter Frame），如P帧或B帧，依赖于其他帧进行解码；
  - `3` 表示 disposable inter frame，通常用于提高压缩效率但不参与长期参考；
  - `4` 表示 generated key frame，由编码器生成的关键帧；
  - `5` 表示 video info/command frame，用于传递视频命令或其他元信息。

- 后4位（Codec ID）：指定视频使用的编解码器。

  - `1` 表示屏幕视频 v1；
  - `2` 表示 VP6；
  - `3` 表示 VP6 with alpha channel；
  - `4` 表示 屏幕视频 v2；
  - `7` 表示 H.264/AVC。

不过上图的 `FrameType` 虽然是 1，但他的 Tag Data 里面的是**编码信息**，而不是**关键帧**，他还要用 `AVCPacketType` 来判断的，如下：

当 `CodecID` 是 7（`AVC`）的时候，`AVCPacketType` 有 3 个值，如下：

1. `AVCPacketType` 等于 0，代表后面的数据是 AVC 序列头
2. `AVCPacketType` 等于 1，代表后面的数据是 AVC NALU 单元
3. `AVCPacketType` 等于 2，代表 AVC 序列结束。

------

Video Data 里面的 CompositionTime Offset 是时间补偿，是用来计算有 B 帧场景的 PTS 的，Tag Header 里面的 TimeStamp 是解码时间，需要加上 CompositionTime Offset 才是 PTS。

#### Audio Tag 

![1-6](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/1-6.png)

##### Tag Header

- Tag Type: 为8时表示Audio Tag
- Data Size: Audio Tag Data的大小
- 其它字段同上

##### Tag Data

###### 第一个字节：音频参数信息

这个字节包含了关于音频的基本属性，具体如下：

- **前4位（Sound Format）**：标识音频编码格式。
  - `0` 表示线性PCM，平台默认字节序；
  - `1` 表示 ADPCM；
  - `2` 表示 MP3；
  - `3` 表示线性PCM，小端字节序；
  - `4` 表示 Nellymoser 16 kHz 单声道；
  - `5` 表示 Nellymoser 8 kHz 单声道；
  - `6` 表示 Nellymoser；
  - `7` 表示 G.711 A-law 对数PCM；
  - `8` 表示 G.711 mu-law 对数PCM；
  - `9` 保留；
  - `10` 表示 AAC；
  - `11` 表示 Speex；
  - `14` 表示 MP3 8 kHz；
  - `15` Device - specific sound， 表示设备特定声音。
- **接下来2位（Sound Rate）**：表示音频采样率。
  - `0` 表示 5.5 kHz；
  - `1` 表示 11 kHz；
  - `2` 表示 22 kHz；
  - `3` 表示 44 kHz。
- **第7位（Sound Size）**：表示音频样本大小。
  - `0` 表示 8位；
  - `1` 表示 16位。
- **最后一位（Sound Type）**：表示音频声道数。
  - `0` 表示单声道；
  - `1` 表示立体声。

- 当 `SoundFormat` 等于 10 （AAC）的时候，`AACAUDIODATA` 里面的 `AACPacketType` 有 2 个值：
  1. 当 `AACPacketType` 等于 0，代表这是 `AudioSpecificConfig`（序列头），`AudioSpecificConfig` 只出现在第一个 `Audio Tag` 中
  2. 当 `AACPacketType` 等于 1，代表这是 AAC Raw frame data，也就是AAC 的裸流

###### 后续数据：音频流数据

从第二个字节开始，就是实际的音频流数据。对于不同类型的音频编码，这部分数据的内容和结构也会有所不同。例如，对于AAC编码的音频，后续数据可能是原始的AAC音频帧或`AudioSpecificConfig`。

# FLV格式优缺点

### FLV格式的优点

1. **文件体积小**：FLV格式的一个显著优点是其能够生成相对较小的文件体积，这使得它非常适合于网络上传输和存储。例如，清晰的FLV视频每分钟大约只有1MB左右，一部完整的电影大约为100MB，仅为普通视频文件体积的三分之一。这种特性极大地减少了带宽占用，并提高了加载速度。

2. **高效的压缩比**：FLV支持多种编码方式，如H.264或VP6视频编码以及MP3或AAC音频编码，这些先进的编解码技术可以实现较高的压缩比例而不明显损失画质。这意味着即使是在较低的数据传输速率下，用户也能享受到较好的观看体验。

3. **易于集成到Web页面中**：由于FLV利用了网页上广泛使用的Adobe Flash Player平台，因此网站访问者只要能查看Flash动画，就可以直接播放FLV格式的视频内容，无需额外安装其他插件。此外，创建包含FLV视频的Flash动画也非常简单，开发者可以通过Adobe提供的工具轻松地将FLV文件嵌入到SWF文件中。

4. **良好的兼容性和普及度**：FLV不仅得到了Adobe自家产品的支持，还获得了许多第三方媒体播放器的支持，比如VLC、RealPlayer等。同时，在线视频分享网站如YouTube、优酷、土豆网等都普遍使用FLV作为默认的视频格式。因此，无论是在PC端还是移动端，FLV都有很好的兼容性。

5. **支持实时流媒体传输**：FLV格式允许通过RTMP协议进行实时直播，同时也支持HTTP上的“伪流”模式。这意味着用户可以在不下载整个文件的情况下立即开始观看视频，这对于提供即时性的在线服务非常重要。

### FLV格式的缺点

1. **视频质量可能不如其他格式**：尽管FLV格式在压缩效率方面表现出色，但在某些情况下，尤其是在高分辨率视频中，它的视觉效果可能会略逊于一些现代视频格式，如MP4。这是因为FLV为了保持较小的文件大小而牺牲了一定程度的画面细节。

2. **对非编软件不够友好**：对于专业视频编辑人员来说，FLV并不是一个理想的选择，因为大多数非线性编辑软件都不能直接将FLV文件拖入时间线进行编辑。如果需要对FLV视频进行复杂的后期处理，则通常需要先将其转换成另一种更适合编辑的格式。

3. **并非所有播放器都支持**：虽然FLV拥有广泛的播放器支持，但仍然存在部分老旧或特定类型的设备及软件无法正确解析FLV文件的问题。这限制了FLV在某些环境下的适用范围。

4. **依赖Flash技术**：随着HTML5的兴起以及浏览器逐渐停止对Flash的支持，FLV格式面临着一定的挑战。尽管FLV本身并不完全依赖于Flash，但早期版本确实紧密关联于此插件，这可能导致未来兼容性问题。

综上所述，FLV格式以其小巧的文件大小、高效的压缩能力、良好的网络适应性和广泛的平台支持成为在线视频传输的理想选择之一；但是，它在视频质量和编辑便利性等方面仍有改进空间，并且随着技术的发展，其长期前景也可能受到影响。对于开发者而言，在选择视频格式时应综合考虑应用场景和个人需求来决定是否采用FLV格式。

# FLV的应用与展望

FLV（Flash Video）格式自推出以来，在音视频传输领域占据了重要地位，尤其是在早期互联网视频应用中扮演了关键角色。随着技术的发展，FLV格式的应用场景也在不断演变和扩展。

### 早期的广泛应用

在浏览器普遍支持Flash插件的时代，FLV格式的视频非常流行。由于其体积小、加载速度快的特点，使得网络观看视频文件成为可能，并且有效地解决了视频文件导入Flash后导致SWF文件体积庞大的问题。因此，它成为了当时主流视频网站如YouTube、优酷等的选择之一，这些平台利用FLV格式为用户提供流畅的在线视频体验。

### 直播领域的持续优势

尽管随着HTML5的普及以及Flash Player逐渐被淘汰，FLV格式在点播视频中的地位有所下降，但在直播领域仍然保持强劲的竞争优势。这主要归因于RTMP推流、HTTP-FLV播放整套方案低延时特性，以及服务端普遍提供的HTTP Web服务能够更广泛地兼容HTTP-FLV。这意味着开发者可以通过简单的配置实现高效的实时视频传输，满足用户对于即时性的需求。

### 现代化转型与融合

面对新技术带来的挑战，FLV格式并未停滞不前，而是积极适应变化，融入新的编码技术和协议。例如，现代视频编码技术如HEVC已经开始应用于FLV格式，进一步提高了压缩效率和传输质量。此外，为了应对移动互联网的发展趋势，许多开发者正在探索将FLV与其他新兴技术相结合，以拓展其应用场景并提升性能。

### 实际案例分析

一个具体的例子是千帆大模型开发与服务平台的支持。该平台不仅允许用户轻松处理包括FLV在内的多种音视频格式，还提供了丰富的音视频处理功能，如编码、解码、转码等，从而更好地服务于多样化的需求。通过这种方式，即使是在移动设备或资源受限环境中，也能保证高质量的内容分发和服务响应。

### 未来展望

虽然当前市场环境下出现了更多先进的替代方案，但FLV凭借其独特的优点——高效性、兼容性和流式传输特性——依然在特定场合下发挥着不可替代的作用。特别是在那些对成本敏感或者需要快速部署解决方案的情况下，FLV仍然是许多项目首选的视频格式之一。同时，随着行业标准和技术的进步，我们可以期待FLV继续演进，与更多先进技术相融合，共同推动音视频技术向前发展。

综上所述，FLV格式从最初的网页嵌入式视频到如今专注于直播及其他特定应用场景，始终保持着灵活性和适应力。无论是在过去还是现在，它都证明了自己的价值，并且在未来也有望继续保持一定的市场份额和技术影响力。

参考： [FLV封装格式—音视频基础知识 · FFmpeg原理](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-flv.html)

文章合集：https://github.com/chongzicbo/ReadWriteThink

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)

