---
title: '音视频开发11：视频封装格式MPEG-TS详解'
categories:
  - [开发,音视频,基础]
tags:
  - 音视频开发
  - 音视频基础
date: 2024-12-14 12:00:00
---

# 音视频开发11：视频封装格式MPEG-TS详解

先讲一下 **MPEG** 是什么，MPEG 全称 Moving Picture Experts Group （动态影像专家小组），该专家组是联合技术委员会（Joint Technical Committee， JTC1）的一部分，JTC1 是由 ISO（国际标准化组织）和 IEC（国际电工委员会）建立的。JTC1 负责信息技术，在 JTC1 中，下设有负责**“音频、图像编码以及多媒体和 超媒体信息”**的子组 SG29。在 SG29 子组中，又设有 多个工作小组，其中就包括 JPEG（联合图片专家组）和 负责活动图像压缩的工作组 WG11 。因此，可以认为 MPEG 是 ISO/IEC JTC1/SG29/WG11，成立于1988年。

把 MPEG 理解成一个组织就行。 MPEG 主要制定影音压缩及传输的规格标准。MPEG 组织制定的标准目前一共有五个 ，MPEG-1、MPEG-2、MPEG-4、MPEG-7及MPEG-21，而 **MPEG-TS** 封装格式 的定义在 **MPEG-2** 标准 里面。

下面简单介绍一下 MPEG-2 标准，MPEG-2 目前广泛用于互联网传输协议，以及早期的有线数字电视，无线数字电视，卫星电视， DVB，DVD，等等。

MPEG-2 标准目前分为10个部分，统称为 ISO/IEC 13818 国际标准，标题 是 “GENERIC CODING OF MOVING PICTURES AND ASSOCIATED AUDIO”。

| 编码    | 标题                     | 描述                                                         |
| ------- | ------------------------ | ------------------------------------------------------------ |
| 13818-1 | System（系统）重点       | 描述多个 音频码流，视频码流 和 基础数据码流，合成传输码流和节目码流的方式。 |
| 13818-2 | Video（视频）            | 描述视频编码的方法                                           |
| 13818-3 | Audio（音频）            | 描述与MPEG-1音频标准反向兼容的 音频编码方法                  |
| 13818-4 | Compliance（符合性测试） | 描述如何判断一个编码码流是否符合MPEG-2码流                   |
| 13818-5 | Software（软件实现）     | 描述了MPEG-2标准的第一、二、三部分的软件实现方法             |
| 13818-6 | DSM-CC （命令与控制）    | 描述交互式多媒体网络中服务器与用户间的会话信令集             |

MPEG-2标准 剩余的四个部分，因为没有广泛使用，就不列举了。从上图的表格可以看出，MPEG-TS 封装格式的定义在 **13818-1** （System）里面。1990年 ATM视频编码专家组 跟 MPEG 组织合作，把 ISO/IEC 13818标准的 13818-1 搞成 ITU-TRec.H.220（System），把 13818-2 搞成 ITU-TRec.H.262 （Video）， ITU-TRec.H.220 跟 ITU-TRec.H.262 都是 **ITU-T** 标准的一部分。

ISO/IEC 13818-1， ITU-TRec.H.220 标准文档压缩包下载地址：[百度网盘](https://pan.baidu.com/s/1LDGjSVdvZQ3t-ngFnQQsHQ)，提取码：f0cv

其实各个组织之间是有合作的，对一些标准文档 做 扩展，衍生，单独提取之类的工作。

------

MPEG 的背景介绍完了，下面来讲一下 MPEG-TS 的封装格式，后缀名以 .ts 结尾的文件 就是 MPEG-TS 封装格式，ts 的全称是 Transport Stream （传输流）。

其实还有一个封装格式 MPEG-PS ，ps 的 全称是 Program Stream （节目流）。注意 Program 翻译成中文是 **节目** 的意思，不是**程序**。PS 封装格式主要用在 不容易发生错误的环境，例如 DVD 光盘。

TS流的包结构是固定长度188字节的，而PS流的包结构是可变长度的。TS码流的固定包结构具有较强的抵抗**传输误码**的能力。

什么是**传输误码**？

> 误码的产生是由于在信号传输中，衰变改变了信号的电压,致使信号在传输中遭到破坏，产生误码。噪音、交流电或闪电造成的脉冲、传输设备故障及其他因素都会导致误码（比如传送的信号是1，而接收到的是0；反之亦然）。由于种种原因,数字信号在传输过程中不可避免地会产生差错。-- 百度百科

简单来说，传输误码是比较底层的情况，数字信号都是二进制流 111000 这种，发生误码就是 1 在传输过程中 变成了 0。这也是我们经常使用的 TCP ，UDP 协议为什么要有一个 **checksum** （校验和）字段 ，发生误码，底层直接丢弃数据包，不会丢上去给应用层。

实际上，对于 UDP 或者 TCP 来说，用 TS 封装格式还是 PS 封装格式 ，抵抗**传输误码**的能力是一样的，因为UDP ，TCP 这些协议帮你处理了误码情况。但是 TS 跟 PS 的标准不只是给 UDP 跟 TCP 用，录像机，DVD机，这些数字视频领域的设备，也会用到 PS 跟TS。

------

无论是 TS 跟 PS， 早期主要是用在 **数字电视** 领域，我国数字电视标准用的是 DVB，全称是 Digital Video Broadcasting（数字视频广播）。

我们以前看电视机的时候，是有很多频道的，接受天线可以切换频道，接受不同频道。一个频道里面是有很多节目的，**注意节目这个概念**，英文是 **Program** ，这个术语在 MPEG-2 文档经常出现。

例如，CCTV频道下面，在同一时刻就有CCTV1-CCTV14这些节目，每个节目都有一个音频流跟视频流，结构图如下：

![ffmpeg-principle-0-1-3-1](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-1.png)

这些 频道、节目、音视频码流如何在TS里面进行区分呢？这就是 TS 封装格式复杂的地方。早期电视机（接受设备）性能比较差，跟基站无法交互，只能被动接受基站广播的数字信号。

我们互联网 TCP/UDP 传输的 是**数字信号**，数字电视基站也是广播的**数字信号**，001001 之类的二进制流。但是 互联网里面使用 TS 封装，通常是传了 一路视频和一路音频，**所以 TS 里面很多字段我们用不着**，互联网的 TS 封装是一个比较简单的应用场景。这种单节目的TS 封装 叫做 Single Program Transport Stream (SPTS)。

如果读者需要做机顶盒之类的开发，想深入理解数字电视的知识，可以看 《Video Demystified: A Handbook for The Digital Engineer》。

------

大体的背景已经介绍完毕，下面来讲一些实战的内容，还是老套路，需要找到一个 可以解析 TS 格式的软件，下载资源如下：

1，elecard stream analyzer，支持 TS，FLV，MP4 等众多格式，可免费试用 30天。 下载地址：[百度网盘](https://pan.baidu.com/s/10zh7LCBmLdT1s_5yRj21ZQ)，提取码：sq5y

3，MPEG-2 TS packet analyser ，界面清晰简洁。下载地址：[百度网盘](https://pan.baidu.com/s/1wFPLCBHtbXEF4p68j2Dn2w)，提取码：hmxj

2，juren.ts ，本文使用的TS流文件。下载地址：[百度网盘](https://pan.baidu.com/s/1AgzIe23W9uh-7klsGv8Jog)，提取码：igs5

**PS：墙裂推荐 elecard stream analyzer ，这软件太好用了，elecard 公司出品的软件都值得试用一番。**

------

用 stream analyzer 打开 juren.ts 文件，截图如下：

![ffmpeg-principle-0-1-3-2](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-2.png)

------

先讲一下基础类型，TS 流里面其实有3种数据包。

1，ES，Elementary Stream（基础数据流），你可以把 一个 ES 包理解成 一个**H264编码后**的视频帧，或者一个音频帧，但是 ES 不只是音视频帧。

2，PES，Packetized Elementary Stream（打包的ES），往ES包上面封装一层 加上 PTS 跟 DTS 等信息。

3，TS，Transport Packet （传输包），固定188个字节，一个大的 PES 包 会拆成多个 小的 块，封装进去 TS 包进行传输。

我们从上往下看，先分析 TS包 的各个字段。如下图：

![ffmpeg-principle-0-1-3-2](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-3.png)

###### 注意一下，stream analyzer 没有按字节顺序解析各个字段，他界面显示的字段不是按字节顺序的。

从上图可以看到， Transport Packet 就是 TS 包，这种包 都是以 0x47 开始的，方便在某些场景下进行同步操作。 ISO/IEC 13818-1 标准文档给了句法讲如何解析一个 TS 包，如下：

![mux-mpeg-ts-4](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-4.png)

上图中的 sync_byte 即是 0x47，`transport_packet()` 是一个解析 TS 包的函数，**句法**如下：

注意**句法**这个词，对应的英文是 **syntax**，MPEG 的很多文档都提供了 句法，他用的句法是 类 C 语言的，如果熟悉C语言，很容易看懂他们提供的伪代码，句法 （syntax） 就是**伪代码**。

![mux-mpeg-ts-5](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-5.png)

从上图句法可以看出，TS 包开头主要有以下字段：

1，**sync_byte**，同步字段，固定是0x47。位置：第 0 ~ 7 位。如果有相同的内容也是 0x47 但不是同步字段，应该需要做转义处理吧，具体我也没细看，埋个坑。

补充：应该不用 转义 0x47，TS长度是固定的 188 字节，具体看如何解析，看解析的代码。

2，**transport_error_indicator**，传输错误标记，在 TCP/UDP 场景，这个字段应该一直是 0 。

2，**playload_unit_start_indicator**，playload 内容开始标记，因为一个 PES 包 会 分成 多个 TS 包，所以需要一个开始的标记。

3，**transport_priority** ，传输优先级，这个字段在互联网领域应该用得不多，不用关注。

4，**PID**，P 不是 Program 的缩写，我也不知道是什么。PID 可以判断 playload 里面是什么数据，标准文档有说，如下：

![mux-mpeg-ts-5-1](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-5-1.png)

5，**transport_scrambling_control**，这个是限制字段，一般用做付费节目之类的，以前卫星电视，需要插一张卡，充钱才能看某些节目。互联网场景很少使用这个字段。

6，**adaptation_field_control**，可变字段标记，这个字段有 4 个值，10 跟 11 代表 后面有扩展的字段需要解析，01 跟 11 代表 这个 TS 包有 playload 。

![mux-mpeg-ts-5-2](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-5-2.png)

7，**continuity_counter**，TS包里面如果有 playload，这个字段会递增。感觉互联网场景用得不多，这个字段我具体也不知道干什么的，埋个坑。后面填。

8，**data_byte**，从这里开始，应该就是 playload的数据了，文档的句法是 循环 N 次处理数据，用 184 减去 adaptation_field 就是 N 的大小。 把 N 理解成 playload 大小就行。实际上真正的代码实现不一定 for N 次。

------

我们现在用 notepad++ 来手动解析 一下上面这 6 个字段，如下图：

![mux-mpeg-ts-6](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-6.png)

TS 文件的 第 0 个字节肯定是 0x47 的，实际上这个 sync_byte 在互联网场景是一个冗余字段，为了兼容的。实际上如果重新设计一个在 TCP/UDP 场景的封装格式，不需要这个 0x47。TCP 传 m3u8 TS 直播流，应该不会传这个 0x47 ，具体我抓包看一下再完善这段内容。

> 在ATSC 标准中，0x47 同步字节从不编 码和传输，而用一个特定的、2电平同步脉冲代替该字 节进行传输，接收机在此位置上插入 0x47 同步字节。

然后 第 1 ~ 2 个字节是 0x40 0x11 ，这两个字节是由 transport_error_indicator，playload_unit_start_indicator，transport_priority 跟 PID 4个字段组成的。

现在 把 0x40 0x11转成 二进制 0100 0000 0001 0001 ，从左 往右看，这里没有第0位，从1开始。然后第一位是 0 ，也就是 transport_error_indicator 等于 0 ，没有传输错误。然后 第二位是 1 ，第二位是 playload_unit_start_indicator，代表这是 playload 的开头包。第三位 是 0 代表 transport_priority （优先级）为 0。

然后剩下的 13 位都是 PID 的值，0 0000 0001 0001 就是 PID ，也就是 17 。

![mux-mpeg-ts-7](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-7.png)

然后 再看 第3个字节，值是 0x10，二进制是 0001 0000，第 3 个字节 由 transport_scrambling_control，adaptation_field_control，continuity_counter 组成，如下图：

![mux-mpeg-ts-8](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-8.png)

最后看 第 4 个字节，第 4 个字节是 data_byte 值是 00 。至此，TS 包的头部的 8 个字段已经讲解完毕。

Table 2-3 图里面有个 bslbf ，这个 bslbf 的全称是 Bit string, left bit first。uimsbf 的全称是 Unsigned integer, most significant bit first.，这些在标准文档都能找到。

------

接着看 后面的 字节内容，如下图：

![mux-mpeg-ts-9](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-9.png)

上图中的 ff 这些是填充字节，要填满 188 字节大小。实际上对互联网流量来说，是一种浪费。TS 这种封装格式早期的用途不是在互联网。

怎么知道这些字节内容是 Service Description Table，应该是根据 头部的 PID 等于 0x11 知道的。然后 Service Description Table 的解析 句法如下：

![mux-mpeg-ts-10](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-10.png)

juren.ts 文件的第一个 TS 包，实际是一个自定义的东西，从 PID 等于 0x11 ，table_id 等于 0x42 可以看出来是 User private 。我直接说一下 这个开头的 ts 包是干什么用的。我看了半天，我也没看懂。这些句法文档 最好 结合一个 TS 的解析代码看，例如 FFmpeg 的 TS 部分的代码，才能理解这个文档。

------

再来看一下 第二个 TS 包，如下图：

![mux-mpeg-ts-11](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-11.png)

从上图可以看出，PID 跟 table_id 都是 0 ，所以这个 TS 包的内容是 PAT （Program Association Table）表的内容，这里提醒一下，前面说 TS 包是对 PES 包的封装，但是不只是 PES，TS 还可以是对 PSI 数据的封装，PSI 的全称 是 Program Specific Information，你可以把 PSI 理解成 节目特定信息。注意 PSI 不是一个表，PSI 是一个统称， PAT，PMT，CAT，NIT 这些都是 PSI，如下图：

![mux-mpeg-ts-11-1](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-11-1.png)

------

PSI 的知识简单讲了一下，不过 TS 的封装格式真是非常复杂，表真多，读者需要自己结合代码来深入理解 TS 。

下面再来找 一下 第一个视频帧的 所有 TS 包，通过这个线索来 理解 TS 封装格式。用 qt creator 来看一下 AVPacket 的内容，如下图：

![mux-mpeg-ts-12](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-12.png)

在 FFmpeg 的 AVPacket 里面 data 指针 指向的 是 H264 编码的实际数据， pos 字段指向的是 TS 包的开始位置，可以看到 0x47 0x41 这种 TS 的标志数字。

第一帧视频 一共是 3768 字节大小，开头是 00 00 01 09 f0 00 00 00 00 06 00 07 ，结尾是 ab 28 fd 45 f9 30 42 22 70 34 00 00 03 00 00 03 00 02 1e 00 。

因为 3768 字节 这么大，肯定是放在多个 TS 包里面的，下面就把这些 TS 包全部找出来，请看下图：

![mux-mpeg-ts-13](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts/mux-mpeg-ts-13.png)

上图有几个注意点。

1，我数了一下，大概有 20 个 TS 包，这20 个 TS 包合成一帧视频数据。

2，注意 20 个 TS 包 里面，只有第一个 TS 包 的 playload_unit_start_indicator 等于 1 ，其他的都是 0。

3，continuity_counter 字段的值不断递增。

------

至此， 本文的 MPEG-TS 封装格式分析完毕。只讲了一些简单的内容，推荐读者阅读 ISO/IEC 13818-1 标准文档，这个文档写得不错，同时结合 FFmpeg 的 TS 代码深入理解这个格式。

这里分享一个读标准文档的技巧，不要觉得读英文文档是一件很难的，自从有了 DEEPL，读英文文档就变成一件很容易的事了，我也不会只读一个文档，我是结合很多中文翻译，或者中文的分析文章来了解一个标准的，因为大家的母语都是中文，大部分情况，中文资料是足够你去了解一个标准。如果中文资料真的匮乏，我会再去看英文文章，但是那样速度会很慢，虽然有 DEEPL，但是有些地方还是需要细细琢磨。

最后讲一点扩展知识：

1，MPEG-PS 跟 MPEG-TS 都是 基于 PES 进行封装，所以两者可以相互转换。

------

参考资料：

1，[MPEG-PS封装格式](https://www.cnblogs.com/CoderTian/p/7701597.html)

2，[TS封装格式介绍及解析](https://blog.csdn.net/guoyunfei123/article/details/106319893)

3，[MPEG-2](https://blog.csdn.net/houxiaoni01/article/details/99831303)

4，[MPEG2系列协议简介](https://blog.csdn.net/aflyeaglenku/article/details/53302172)

5，《MPEG 基础和协议分析指南》- Tektronix



原文：[MPEG-TS封装格式—音视频基础知识 · FFmpeg原理](https://ffmpeg.xianwaizhiyin.net/base-knowledge/mux-mpeg-ts.html)



[封装格式--3：TS格式详解 - 简书](https://www.jianshu.com/p/e5f3456a0205)

[MPEG-TS 封装格式 - 小夕nike - 博客园](https://www.cnblogs.com/moonwalk/p/16200434.html#_label2_0)

[音视频封装：MPTG2-TS 媒体封装实例解析和说明-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1746983)

[MPEG-2 TS流结构浅析_mpeg2ts文件结构读取-CSDN博客](https://blog.csdn.net/GDUYT_gduyt/article/details/123076561)



文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)