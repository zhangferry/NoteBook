![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200919231216.png)

## 背景

MP4是我们常见的视频格式，往往我们在播放服务器视频时直接就是请求的MP4视频源。但其实这样并不好，MP4头文件[ftyp+moov]较大，初始化的播放需要下载完整的头文件并进行解析，之后再下载一定长度的可播视频片段才能进行播放。另外随着视频尺寸的增大头文件也会不断变大，这个初始播放时间也会更长。针对这种情况需要一种能加快视频初始解析的方法，HLS就是苹果提出的用于解决这种问题的方案。

## HLS

[HLS](https://developer.apple.com/streaming/ "HLS")为HTTP Live Streaming的缩写，是由苹果公司提出的基于HTTP的流媒体网络传输协议，它可以同时支持直播和点播，还支持多清晰度、音视频双轨、字幕等功能。它的原理是将一整条视频分成多段小的视频，完整的播放是由这一个个片段拼接而成的。

HLS在移动端使用很广泛，当前支持HLS协议的客户端有：

* iOS 3.0及以上，AVPlayer原生支持HLS
* Android 3.0及以上
* Adobe Flash Player 11.0及以上

它的大致原理是这样的：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200919102604.png)

1、采集音视频

2、在服务器编码音视频

3、编码后以MPEG-2的传输串流形式交由切片器（Stream Segmenter）

4、切片器创建索引文件和ts播放列表，索引文件用于指示音视频位置，ts为真实的多媒体片段

5、将上一步资源放到HTTP服务器上

6、客户端请求该索引文件进行播放，可以通过索引文件找到播放内容

参考资料：[HTTP Live Streaming Document](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008332-CH1-SW1 "HTTP Live Streaming Document")

## M3U8

实现HLS的一个关键步骤是上面的第四步，即索引文件和ts播放列表的组织。这里用到的就是M3U8格式。M3U8是Unicode版本的[M3U](https://zh.wikipedia.org/wiki/M3U)，8代表使用的是UTF-8编码，M3U和M3U8都是多媒体列表的文件格式。

接下来我们以一条WWDC里的视频为例，看下M3U8格式是什么样子的。

播放页面为：https://developer.apple.com/videos/play/wwdc2019/507 ，通过Charles进行抓包，我们可以得到视频播放过程中的M3U8文件。
![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200919163827.png)

在分析这个路径格式前，我们需要知道 M3U8 是有两种格式的，一种是作为主播放列表（Master Playlist）存在，它里面包含了音视频、字幕的一些说明和路径，主列表指示的路径是另一个M3U8文件，即另一个格式，作为播放存在的，它里面也有路径，指示的是片段（ts）文件，片段文件是真正的多媒体内容。

看抓包内容，`hls_vod_mvp.m3u8`为主列表文件，上面的`0640.m3u8`为视频列表文件。

### M3U8格式说明
有时做测试，或者一些特殊情况时我们可能需要手动修改M3U8文件内容，所以需要对它的格式有一定的了解。该格式的定义写在[RFC 8216](https://tools.ietf.org/html/rfc8216 "RFC 8216")号文件里，以下是一些注意事项：
* M3U8文件必须以UTF-8进行编码，不能使用 Byte Order Mark（BOM）字节序， 不能包含 utf-8 控制字符（U+0000 ~ U_001F 和 U+007F ~ u+009F）
* M3U8文件内容的每一行要么是空行，要么是一个URI，要么是以`#`开头的字符串，不能出现空白字符。
* 内置标签都是`#EXT`开头的字符串，大小写敏感。
* URI为内容路径，可以是相对路径也可以是绝对路径

### Master M3U8 列表文件

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200919164450.png)

主M3U8索引文件，一般用于指定多个索引源。我们先分析下该主m3u8文件`hls_vod_mvp.m3u8`的内容，它的头部是这样的

#### 头部格式
```
#EXTM3U
#EXT-X-VERSION:7
#EXT-X-INDEPENDENT-SEGMENTS
```
`#EXTM3U`表明该文件是一个M3U格式，所有的M3U格式文件都应该把该内容放置到第一行。

`#EXT-X-VERSIOn`指示播放列表的兼容版本，当前为7。

`#EXT-X-INDEPENDENT-SEGMENTS`该标签表明对于一个媒体片段中的所有媒体样本均可独立进行解码，而无须依赖其他媒体片段信息。

#### 字幕格式
再往下的内容是一些字幕说明，字幕内容不是必须的。
```
#EXT-X-MEDIA:TYPE=SUBTITLES,GROUP-ID="subs",NAME="English",DEFAULT=YES,AUTOSELECT=YES,FORCED=NO,LANGUAGE="eng",URI="subtitles/eng/prog_index.m3u8"
#EXT-X-MEDIA:TYPE=SUBTITLES,GROUP-ID="subsC",NAME="English",DEFAULT=YES,AUTOSELECT=YES,FORCED=NO,LANGUAGE="eng",URI="subtitles/engc/prog_index.m3u8"

#EXT-X-MEDIA:TYPE=SUBTITLES,GROUP-ID="subs",NAME="Japanese",DEFAULT=YES,AUTOSELECT=YES,FORCED=NO,LANGUAGE="jpn",URI="subtitles/jpn/prog_index.m3u8"
#EXT-X-MEDIA:TYPE=SUBTITLES,GROUP-ID="subsC",NAME="Japanese",DEFAULT=YES,AUTOSELECT=YES,FORCED=NO,LANGUAGE="jpn",URI="subtitles/jpnc/prog_index.m3u8"

#EXT-X-MEDIA:TYPE=SUBTITLES,GROUP-ID="subs",NAME="Chinese",DEFAULT=YES,AUTOSELECT=YES,FORCED=NO,LANGUAGE="zho",URI="subtitles/zho/prog_index.m3u8"
#EXT-X-MEDIA:TYPE=SUBTITLES,GROUP-ID="subsC",NAME="Chinese",DEFAULT=YES,AUTOSELECT=YES,FORCED=NO,LANGUAGE="zho",URI="subtitles/zhoc/prog_index.m3u8"
```
`#EXT-X-MEDIA`用于指定相同内容的多语言媒体列表资源。

`TYPE`为资源类型，可选内容有：AUDIO、VIDEO、SUBTITLES、CLOSED-CAPTIONS。

上面内容设置的是`TYPE=SUBTITLES`，即为字幕类型。

`GROUP-ID`为多语言翻译所属组，为必选参数

`NAME`为翻译流可读的描述信息，该值对应`AVMediaSelectionOption`的`displayName`。

`DEFAULT`，`AUTOSELECT`，`FORCED`为三个BOOL值分别对应如果缺少必要信息时是否默认选中该翻译流，用户没有显示设置时播放该播放流，FORCED只针对字幕类型有效，用于标记当前自动选择该翻译流。

`LANGUAGE`用于指定语言类型，它是根据[ISO 639 语言码](https://www.w3.org/WAI/ER/WD-AERT/iso639.htm “ISO 639 语言码”)标准设置的。系统默认的播放器在选择字幕时，展示的字幕列表名称是根据这个值设定的。

`URI`为该资源的定位信息，在这里其对应的是一条字幕的M3U8文件。`subtitles/eng/prog_index.m3u8`是一个相对路径，

通过以上信息，我们可以分析出上面内容的含义为：当前视频支持三种字幕：英文，日文，中文。但每种语言都有两条`EXT-X-MEDIA`信息，他们的区别是分组不同，一个在`subs`分组，一个在`subsC`分组。为啥有两个分组，这个后面再说。

#### 视频格式
再往下看，为视频内容的索引：
```
#EXT-X-STREAM-INF:BANDWIDTH=827299,AVERAGE-BANDWIDTH=747464,CODECS="avc1.64001f,mp4a.40.2",RESOLUTION=640x360,FRAME-RATE=29.970,AUDIO="program_audio",SUBTITLES="subs"
0640/0640.m3u8
#EXT-X-I-FRAME-STREAM-INF:BANDWIDTH=360849,AVERAGE-BANDWIDTH=320932,CODECS="avc1.64001f",RESOLUTION=640x360,URI="0640/0640_I-Frame.m3u8"
```
`EXT-X-STREAM-INF`：该属性指定了一个备份源，即视频播放路径和一些视频的信息，以下是对应内容的配置：
`BANDWIDTH`为峰值比特率， 827299，为827299bit/s，即最高峰值时每秒消耗流量101KB。

`AVERAGE-BANDWIDTH`为平均比特率，747464

`CODECS`为编码信息，`avc1.64001f,mp4a.40.2`，avc代表的是h264编码格式，后面的64001f，是由16进制表示的编码参数，64，00，1f分别代表三个不同的参数值。mp4a是一种音频编码格式，后面的40.2代表音频的编码参数。

`RESOLUTION`为视频分辨率，当前一条视频源分辨率为640x360。

`FRAME-RATE`为最大帧率，29.970 代表当前播放的最大帧率为每秒29.970帧。

`AUDIO`是音频所在组，`program_audio`为对应音频组的名称。

`SUBTITLES`指示对应的字幕分组，`subs`为对应字幕组的名称。上面的字幕信息有个`GROUP-ID`，该值与之对应。

`URI`为内容路径，`0640/0640.m3u8`对应的就是该视频源的m3u8文件路径。这个可以在抓包信息里看到。

在`EXT-X-STREAM-INF`下面是`EXT-X-I-FRAME-STREAM-INF`，表示播放列表文件中包含的多媒体资源的I帧（关键帧）。因为I帧只是一个画面，所以它不包含音频内容，其余参数跟视频内容格式一致。

在之后就是对应不同分别率的视频源，1920x1080、1280x720、960x540、480x270，因为HLS会根据网络情况自行切换清晰度，所以一般会准备多个清晰度以供选择。根据抓包数据分析，播放的第一个片段是640清晰度的，之后的第2-8个片段为480清晰度，再之后又切换到了640清晰度。

#### 音频格式
再往下看是对应音频的索引
```
#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="program_audio",LANGUAGE="eng",NAME="Alternate Audio",AUTOSELECT=YES,DEFAULT=YES,URI="audio1/audio1.m3u8"
```
`#EXT-X-MEDIA`上面出现过，为多语言没提列表。

`TYPE=AUDIO`，这次类型为音频。

`GROUP-ID`为分组ID，对应`EXT-X-STREAM-INF`里的`AUDIO`内容。

`URI=audio1/audio1.m3u8`对应音频路径。

#### 不同编码格式的备用源
在该主M3U8文件中我们还能看到一条640分辨率的视频源，它与上面的640分辨率还不一样，它的内容是这样的：
```
#EXT-X-STREAM-INF:BANDWIDTH=1922391,AVERAGE-BANDWIDTH=1276855,VIDEO-RANGE=SDR,CODECS="hvc1.2.4.H150.B0,mp4a.40.2",RESOLUTION=640x360,FRAME-RATE=29.970,AUDIO="program_audio_0",SUBTITLES="subsC"
0640c/prog_index.m3u8
#EXT-X-I-FRAME-STREAM-INF:BANDWIDTH=1922391,AVERAGE-BANDWIDTH=1276855,CODECS="hvc1.2.4.H150.B0",RESOLUTION=640x360,URI="0640c/iframe_index.m3u8"

#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="program_audio_0",LANGUAGE="eng",NAME="Alternate Audio",AUTOSELECT=YES,DEFAULT=YES,URI="audioc/prog_index.m3u8"
```
`CODECS`编码格式为`hvc1.2.4.H150.B0,mp4a.40.2`，音频编码格式没变，但视频编码格式变了。`hvc1`是HEVC（H265）编码格式里的一种，它是由苹果推出的新一代视频编码格式，因为兼容性问题很多客户端还无法解析该格式，所以并不是很普及，该格式的视频源出现在这里应该是一种备用。对比相同分辨率的两条内容，还能发现hvc1格式会比avc1格式比特率更高，这说明相同分辨率下hvc1的内容更大，avc1的压缩比更高。

对应hvc1格式的视频源，它的字幕内容分组和音频内容分组也都变了，这也是为什么上面的字幕同一语种会有两份，他们分别对应avc1和hvc1格式的视频源。

M3U8的主列表就这些内容了，该条内容的音视频是分开处理的，其实也可以合在一起。

### 包含媒体资料的m3u8文件
以`0640.m3u8`这个文件为例
```
#EXTM3U
#EXT-X-VERSION:4
#EXT-X-TARGETDURATION:7
#EXT-X-MEDIA-SEQUENCE:1
#EXT-X-PLAYLIST-TYPE:VOD
#EXTINF:6.006,
0640_00001.ts
#EXTINF:6.006,
0640_00002.ts
#EXTINF:6.006,
0640_00003.ts
....
#EXT-X-ENDLIST
```

`#EXTM3U`，`#EXT-X-VERSION`，分别为M3U文件头和兼容版本号，这种格式是早期的所以版本号比主文件低一些。

`EXT-X-TARGETDURATION`代表每个播放片段的最大时长，7，代表7秒，该目录下的片段不能超过7s。

`EXT-X-MEDIA-SEQUENCE`代表播放列表的第一个片段序号，1，代表播放片段是从1开始的。

`#EXTINF`代表片段的时长，6.006表示当前片段为6.006s。视频总时长的信息是通过该值累加获取的。

`0640_00001.ts`为片段的相对路径，ts文件代表一段视频或者音频，它可以是ts，mp4，aac等格式。因为前面已经指定了从1开始，所以这里序号是0640_00001。

`#EXT-X-ENDLIST`为媒体内容的结束标识，因为m3u8即可以表示点播也可以表示直播，区分点播还是直播就看文件末尾是否有这个标识符。如果没有的话就代表直播，播放会一直持续下去。

音频文件`audio1.m3u8`，字幕文件`pro_index.m3u8`的内容也是类似的，区别在于他们的切片内容一个是acc的音频文件，一个是webvtt的字幕文件。

包含切片内容的M3U8也可以作为独立的视频链接存在，这时切片内容就需要同时包含音视频内容了。

### 文件加密

HLS协议支持加密，如果索引文件中包含了一个密钥文件的信息，那接下来的媒体文件就必须使用密钥解密后才能解密打开了。当前的 HLS 支持使用16-octet 类型密钥的 AES-128 加密。这个密钥格式是一个由这在二进制格式中的16个八进制组的数组打包而成的。

加密的配置模式通常包含三种：

 模式一：允许你在磁盘上制定一个密钥文件路径，切片器会在索引文件中插入存在的密钥文件的 URL。所有的媒体文件都使用该密钥进行加密。 

模式二：切片器会生成一个随机密钥文件，将它保存在指定的路径，并在索引文件中引用它。所有的媒体文件都会使用这个随机密钥进行加密。

模式三：每 n 个片段生成一个随机密钥文件，并保存到指定的位置，在索引中引用它。这个模式的密钥处于轮流加密状态。每一组 n 个片段文件会使用不同的密钥加密。

参考：[HLS-iOS视频播放服务架构深入探究（一）](http://yangchao0033.github.io/blog/2016/01/29/hls-1/ "HLS-iOS视频播放服务架构深入探究（一）")

