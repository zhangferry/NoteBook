## HLS

HLS为HTTP Live Streaming的缩写，是由苹果公司提出的基于HTTP的流媒体网络传输协议。它的大致原理是将一整条视频分成多段小的视频，播放时是下载这一个个片段然后拼接成一个完整的视频。

## M3U8

如果要实现HLS的方案就需要有这么一个东西，它包含多个短的播放片段的，并标记片段的顺序，这个东西就是M3U文件，该文件格式也是有苹果制定的，而我们熟知的M3U8是Unicode版本的M3U，用UTF-8编码。

我们以wwdc里的一条视频为例，来看下它的视频格式是什么样的，页面地址为这个：https://developer.apple.com/videos/play/wwdc2019/507

我们可以开启Charles进行抓包，看下视频播放过程中发生了哪些请求。

m3u8 有两种形式，一种是作为主播放列表（Master Playlist），它里面包含了视频播放相关内容视频，音频，有些还会有字幕。主文件里面对视频，或者音频的指引是另一个m3u8文件指示的。这个m3u8文件里存放的才是片段（ts）文件，片段文件是真正的音视频内容。

m3u8文件内容的每一行要么是空行，要么是一个URI，要么是以#开头的字符串，内置标签都是#EXT开头的字符串。

### Master M3U8
我们先看下该主m3u8文件`hls_vod_mvp.m3u8`内容，它的头部是这样的

```
#EXTM3U
#EXT-X-VERSION:7
#EXT-X-INDEPENDENT-SEGMENTS
```
`#EXTM3U`表明该文件是一个m3u8字符。
`#EXT-X-VERSIOn`指示播放列表的兼容版本。
`#EXT-X-INDEPENDENT-SEGMENTS`该标签表明对于一个媒体片段中的所有媒体样本均可独立进行解码，而无须依赖其他媒体片段信息。

在往下是一些字幕说明，字幕内容不是必须的。
```
#EXT-X-MEDIA:TYPE=SUBTITLES,GROUP-ID="subs",NAME="English",DEFAULT=YES,AUTOSELECT=YES,FORCED=NO,LANGUAGE="eng",URI="subtitles/eng/prog_index.m3u8"
#EXT-X-MEDIA:TYPE=SUBTITLES,GROUP-ID="subsC",NAME="English",DEFAULT=YES,AUTOSELECT=YES,FORCED=NO,LANGUAGE="eng",URI="subtitles/engc/prog_index.m3u8"


#EXT-X-MEDIA:TYPE=SUBTITLES,GROUP-ID="subs",NAME="Japanese",DEFAULT=YES,AUTOSELECT=YES,FORCED=NO,LANGUAGE="jpn",URI="subtitles/jpn/prog_index.m3u8"
#EXT-X-MEDIA:TYPE=SUBTITLES,GROUP-ID="subsC",NAME="Japanese",DEFAULT=YES,AUTOSELECT=YES,FORCED=NO,LANGUAGE="jpn",URI="subtitles/jpnc/prog_index.m3u8"

#EXT-X-MEDIA:TYPE=SUBTITLES,GROUP-ID="subs",NAME="Chinese",DEFAULT=YES,AUTOSELECT=YES,FORCED=NO,LANGUAGE="zho",URI="subtitles/zho/prog_index.m3u8"
#EXT-X-MEDIA:TYPE=SUBTITLES,GROUP-ID="subsC",NAME="Chinese",DEFAULT=YES,AUTOSELECT=YES,FORCED=NO,LANGUAGE="zho",URI="subtitles/zhoc/prog_index.m3u8"
```

`#EXT-X-MEDIA`用于指定相同内容的可替换的多语言翻译播放媒体列表资源。
该值对应了一个KEY为TYPE的键值对，VALUE可选：AUDIO、VIDEO、SUBTITLES、CLOSED-CAPTIONS。
上面内容设置的是TYPE=SUBTITLES，即为字幕类型。

后面的GROUP-ID为多语言翻译所属组，为必选参数，之后的NAME为翻译流可读的描述信息，该值对应AVMediaSelectionOption的displayName。

DEFAULT，AUTOSELECT，FORCED为三个BOOL值分别对应如果缺少必要信息时是否默认选中该翻译流，用户没有显示设置时播放该播放流，FORCED只针对字幕类型有效，用于标记当前自动选择该翻译流。

LANGUAGE用于指定主流使用语言。
URI为该资源的定位信息，这里可以填相对路径也可以填绝对路径，他们对应的都是片段m3u8。

通过以上信息，我们可以分析上面的格式信息为：当前视频支持三种字幕：英文，日文，中文，但每种语言都有两条EXT信息，他们的区别是分组不同，一个在`subs`分组，一个在`subsC`分组。为啥有两个分组，这个后面再说。

再往下看：
```
#EXT-X-STREAM-INF:BANDWIDTH=827299,AVERAGE-BANDWIDTH=747464,CODECS="avc1.64001f,mp4a.40.2",RESOLUTION=640x360,FRAME-RATE=29.970,AUDIO="program_audio",SUBTITLES="subs"
0640/0640.m3u8
#EXT-X-I-FRAME-STREAM-INF:BANDWIDTH=360849,AVERAGE-BANDWIDTH=320932,CODECS="avc1.64001f",RESOLUTION=640x360,URI="0640/0640_I-Frame.m3u8"

#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="program_audio",LANGUAGE="eng",NAME="Alternate Audio",AUTOSELECT=YES,DEFAULT=YES,URI="audio1/audio1.m3u8"
```
`EXT-X-STREAM-INF`：该属性指定了一个备份源，即视频播放路径和一些视频的信息，以下是对应内容的配置：
`BANDWIDTH`为码率， 827299
`AVERAGE-BANDWIDTH`为平均码率，747464
`CODECS`为编码信息，`avc1.64001f,mp4a.40.2`，avc代表的是h264编码格式，后面的64001f，是由16进制表示的编码参数，64，00，1f分别代表三个不同的参数值。mp4a是一种音频编码格式，后面的40.2代表音频的编码参数。
`RESOLUTION`为视频尺寸，640x360
`FRAME-RATE`为最大帧率，29.970 代表当前播放的最大帧率为每秒29.970帧。
`AUDIO`指示对应的音频路径，program_audio，真实的音频内容需要放到这个目录里
`SUBTITLES`指示对应的字幕路径，subs，字幕文件需要放到这个目录里
`URI`为内容路径，对应的就是视频m3u8的路径。

再下面一条内容是`EXT-X-I-FRAME-STREAM-INF`，表示播放列表文件中包含的多媒体资源的I帧。因为I帧只是一个画面，所以它不包含音频内容。

第三条内容跟字幕是一样的格式，只不过TYPE=AUDIO，代表音频内容，其GROUP-ID为`EXT-X-STREAM-INF`里的AUDIO内容。

该主m3u8文件在表示0640/0640.m3u8条目说明的下面还有1920、1280、960、480等目录，分别代表对应尺寸的视频源。HLS在播放视频时会根据当前网络环境切换不同清晰度视频，它切换的就是对应到不同的视频源文件。

再后面的内容跟上面的很像了
```
#EXT-X-STREAM-INF:BANDWIDTH=1922391,AVERAGE-BANDWIDTH=1276855,VIDEO-RANGE=SDR,CODECS="hvc1.2.4.H150.B0,mp4a.40.2",RESOLUTION=640x360,FRAME-RATE=29.970,AUDIO="program_audio_0",SUBTITLES="subsC"
0640c/prog_index.m3u8
#EXT-X-I-FRAME-STREAM-INF:BANDWIDTH=1922391,AVERAGE-BANDWIDTH=1276855,CODECS="hvc1.2.4.H150.B0",RESOLUTION=640x360,URI="0640c/iframe_index.m3u8"

#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="program_audio_0",LANGUAGE="eng",NAME="Alternate Audio",AUTOSELECT=YES,DEFAULT=YES,URI="audioc/prog_index.m3u8"
```
这个格式跟上面的那一组很类似，都是代表640分辨率的视频，但有一个区别是视频的编码格式变了：`hvc1.2.4.H150.B0,mp4a.40.2`。hvc1即HEVC，也叫H265编码格式里的一种，它是由苹果推出的新一代视频编码格式，但因为兼容性问题很多播放器还无法解析该格式，这里我的理解是它仅作为备用。对应hvc1编码的视频内容提供了和avc1一样的分辨率内容各一份，另外音频和字幕也是单独一份，这也就是为什么最开始会出现两份同一语言的字幕文件了。

有一个内容可以注意到相同分辨率的视频avc1格式要比hvc1码率更小，即更小，压缩比更高。


m3u8的主文件就这些内容了，该条内容的音视频是分开处理的，其实也可以合在一起处理，将音频内容合入到视频文件里。

### 包含媒体资料的m3u8文件
以0640.m3u8这个文件为例
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
#....
#EXT-X-ENDLIST
```
前两条跟上面那个m3u8格式内容一样，分别表示文件类型标识，和版本号。
`EXT-X-TARGETDURATION`代表每个播放片段的最大时长，7，代表7秒，每个片段不能超过7s
`EXT-X-MEDIA-SEQUENCE`代表播放列表的第一个片段序号，1，代表播放片段是从1开始的
`#EXTINF`代表片段的时长，6.006表示当前片段为6.006s
`0640_00001.ts`为片段路径，当前展示的相对路径，ts文件代表一段视频或者音频，它可以是ts，mp4，aac等格式。因为前面已经制定了从1开始，所以这里序号是0640_00001。
`#EXT-X-ENDLIST`为媒体内容的结束标识，因为m3u8即可以表示点播也可以表示直播，如果m3u8问价末尾有这个标识即为点播，如果没有即为直播，播放会一直持续下去。

音频文件audio1.m3u8，字幕文件pro_index.m3u8的内容也是一样的。




