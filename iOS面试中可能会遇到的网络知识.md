![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200630104145.png)

计算机网络是计算机科学与技术专业的必修课，也是移动端，前端，后端都会涉及到的知识点，同时它也是iOS面试中大概率会出现的问题。所以准备面试的话，网络相关的知识点一定不能错过。这里总结了一些我认为有用的和最近面试遇到的网络相关知识点。

去年写过一篇[《图解TCP/IP》总结](https://zhangferry.com/2019/08/31/diagram_tcpip_concepts/)的文章，也可以对着看下。

### 计算机网络是如何分层的

网络有两种分层模型，一种是ISO（国际标准化组织）制定的OSI（Open System Interconnect）模型，它将网络分为七层。一种是TCP/IP的四层网络模型。OSI是一种学术上的国际标准，理想概念，TCP/IP是事实上的国际标准，被广泛应用于现实生活中。两者的关系可以看这个图：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200629174612.png)

注：也有说五层模型的，它跟四层模型的区别就是，在OSI模型中的数据链路层和物理层，前者将其作为两层，后者将其合并为一层称为网络接口层。一般作为面试题的话都是需要讲出OSI七层模型的。

各个层的含义以及它们之间的关系可以看这张图：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200629174637.png)

### Http协议

#### http协议特性

* HTTP 协议构建于 TCP/IP 协议之上，是一个应用层协议，默认端口号是 80
* 灵活：HTTP允许传输任意类型的数据对象。正在传输的类型由Content-Type加以标记。
* 无状态：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。
* 无状态：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传。



#### 请求方法

* GET：请求获取Request-URI标识的资源，请求参数附加在url上，明文展示。

* POST：在Request-URI所标识的资源后附加新的数据，常用于修改服务器资源或者提交资源到服务器。POST请求体是放到body中的，可以指定编码方式，更加安全。

* HEAD：请求获取由Request-URI所标识的资源的响应消息报头。

* PUT：请求服务器存储一个资源，并用Request-URI作为其标识。

* DELETE：请求服务器删除Request-URI所标识的资源。

* TRACE：请求服务器回送收到的请求信息，主要用于测试或诊断。

* OPTIONS：请求查询服务器的性能，或者查询与资源相关的选项和需求。



#### 请求和响应报文

以该链接为例：https://zhangferry.com/2019/08/31/diagram_tcpip_concepts/

在Chrome查看其请求的Headers信息。

**General**

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200630103717.png)

这里标记了请求的URL，请求方法为GET。状态码为304，代表文件未修改，可以直接使用缓存的文件。远程地址为185.199.111.153:443，此IP为Github 服务器地址，是因为我的博客是部署在GitHub上的。

除了304还有别的状态码，分别是：

- `200 OK` 客户端请求成功
- `301 Moved Permanently` 请求永久重定向
- `302 Moved Temporarily` 请求临时重定向
- `304 Not Modified` 文件未修改，可以直接使用缓存的文件。
- `400 Bad Request` 由于客户端请求有语法错误，不能被服务器所理解。
- `401 Unauthorized` 请求未经授权。这个状态代码必须和WWW-Authenticate报头域一起使用
- `403 Forbidden` 服务器收到请求，但是拒绝提供服务。服务器通常会在响应正文中给出不提供服务的原因
- `404 Not Found` 请求的资源不存在，例如，输入了错误的URL
- `500 Internal Server Error` 服务器发生不可预期的错误，导致无法完成客户端的请求。
- `503 Service Unavailable` 服务器当前不能够处理客户端的请求，在一段时间之后，服务器可能会恢复正常。

**Response Headers**：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200629174317.png)

content-encoding：用于指定压缩算法

content-length：资源的大小，以十进制字节数表示。

content-type：指示资源的媒体类型。图中所示内容类型为html的文本类型，文字编码方式为utf-8

last-modified：上次内容修改的日期，为6月8号

status：304 文件未修改状态码



注：其中content-type在响应头中代表，需要解析的格式。在请求头中代表上传到服务器的内容格式。

**Request Headers**：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200629174421.png)

:method：GET请求

:path：url路径

:scheme：https请求

accept：通知服务器可以返回的数据类型。

accept-encoding：编码算法，通常是压缩算法，可用于发送回的资源

accept-language：通知服务器预期发送回的语言类型。这是一个提示，并不一定由用户完全控制:服务器应该始终注意不要覆盖用户的显式选择(比如从下拉列表中选择语言)。

cookie：浏览器cookie

user-agent：用户代理，标记系统和浏览器内核



更多请求头的字段含义可以参考这里：[HTTP headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)

### TCP三次握手和四次挥手的过程以及为什么要有三次和四次

在了解TCP握手之前我们先看下TCP的报文样式：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200629175217.png)

其中控制位（Control Flag）标记着握手阶段的各个状态。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200629175356.png)

#### TCP三次握手

示意图如下：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200630103422.png)

三次握手是指建立一个TCP连接时，需要客户端和服务器总共发送3个数据包。

1、第一次握手（SYN=1, seq=x）

客户端发送一个 TCP 的 SYN 标志位置1的包，指明客户端打算连接的服务器的端口，以及初始序号 X,保存在包头的序列号(Sequence Number)字段里。

发送完毕后，客户端进入 `SYN_SEND` 状态。

2、第二次握手（SYN=1, ACK=1, seq=y, ACKnum=x+1）

服务器发回确认包(ACK)应答。即 SYN 标志位和 ACK 标志位均为1。服务器端选择自己 ISN 序列号，放到 Seq 域里，同时将确认序号(Acknowledgement Number)设置为客户的 ISN 加1，即X+1。 发送完毕后，服务器端进入 `SYN_RCVD` 状态。

3、第三次握手（ACK=1, ACKnum=y+1）

客户端再次发送确认包(ACK)，SYN 标志位为0，ACK 标志位为1，并且把服务器发来 ACK 的序号字段+1，放在确定字段中发送给对方，并且在数据段放写ISN的+1

发送完毕后，客户端进入 `ESTABLISHED` 状态，当服务器端接收到这个包时，也进入 `ESTABLISHED` 状态，TCP 握手结束。



**问题一：为什么需要三次握手呢？**

在谢希仁著的《计算机网络》里说，『为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误』。怎么理解呢，我们假设一种情况，有一个建立连接的第一次握手的报文段因为滞留到网络中过了较长时间才发送到服务端。这时服务器是要做ACK应答的，如果只有两次握手就代表连接建立，那服务器此时就要等待客户端发送建立连接之后的数据。而这只是一个因滞留而废弃的请求，是不是白白浪费了很多服务器资源。

从另一个角度看这个问题，TCP是全双工的通信模式，需要保证两端都已经建立可靠有效的连接。在三次握手过程中，我们可以确认的状态是：

第一次握手：服务器确认自己接收OK，服务端确认客户端发送OK。

第二次握手：客户端确认自己发送OK，客户端确认自己接收OK，客户端确认服务器发送OK，客户端确认服务器接收OK。

第三次握手：服务器确认自己发送OK，服务器确认客户端接收OK。

只有握手三次才能达到全双工的目的：确认自己和对方都能够接收和发送消息。



#### TCP四次挥手

示意图如下：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200630103444.png)

四次挥手表示要发送四个包，挥手的目的是断开连接。

1、第一次挥手（FIN=1, seq=x）

假设客户端想要关闭连接，客户端发送一个 FIN 标志位置为1的包，表示自己已经没有数据可以发送了，但是仍然可以接受数据。

发送完毕后，客户端进入 `FIN_WAIT_1` 状态。

2、第二次挥手（ACK=1，ACKnum=x+1）

服务器端确认客户端的 FIN 包，发送一个确认包，表明自己接受到了客户端关闭连接的请求，但还没有准备好关闭连接。

发送完毕后，服务器端进入 `CLOSE_WAIT` 状态，客户端接收到这个确认包之后，进入 `FIN_WAIT_2` 状态，等待服务器端关闭连接。

3、第三次挥手（FIN=1，seq=y）

服务器端准备好关闭连接时，向客户端发送结束连接请求，FIN 置为1。

发送完毕后，服务器端进入 `LAST_ACK` 状态，等待来自客户端的最后一个ACK。

4、第四次挥手（ACK=1，ACKnum=y+1）

客户端接收到来自服务器端的关闭请求，发送一个确认包，并进入 `TIME_WAIT`状态，等待可能出现的要求重传的 ACK 包。

服务器端接收到这个确认包之后，关闭连接，进入 `CLOSED` 状态。

客户端等待了某个固定时间（两个最大段生命周期，2MSL，2 Maximum Segment Lifetime）之后，没有收到服务器端的 ACK ，认为服务器端已经正常关闭连接，于是自己也关闭连接，进入 `CLOSED` 状态。



**问题一：为什么挥手需要四次呢？为什么不能将ACK和FIN报文一起发送？**

当服务器收到FIN报文时，很可能并不会立即关闭SOCKET，所以只能先回复一个ACK报文，告诉客户端『你发的FIN我收到了』。只有等到服务端所有的报文都发送完了，才能发FIN报文，所以要将ACK和FIN分开发送，这就导致需要四次挥手。



**问题二：为什么TIMED_WAIT之后要等2MSL才进入CLOSED状态？**

MSL是TCP报文的最大生命周期，因为TIME_WAIT持续在2MSL就可以保证在两个传输方向上的尚未接收到或者迟到的报文段已经消失，同时也是在理论上保证最后一个报文可靠到达。假设最后一个ACK丢失，那么服务器会再重发一个FIN，这是虽然客户端的进程不在了，但是TCP连接还在，仍然可以重发LAST_ACK。

###HTTPS的流程

HTTPS = HTTP + TLS/SSL，它的建立可以用以下示意图表示：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200629162110.png)

1、客户端首次请求服务器，告诉服务器自己支持的协议版本，支持的加密算法及压缩算法，并生成一个随机数（client random）告知服务器。

2、服务器确认双方使用的加密方法，并返回给客户端证书以及一个服务器生成的随机数（server random）

3、客户端收到证书后，首先验证证书的有效性，然后生成一个新的随机数（premaster secret），并使用数字证书中的公钥，加密这个随机数，发送给服务器。

4、服务器接收到加密后的随机数后，使用私钥进行解密，获取这个随机数（premaster secret

5、服务器和客户端根据约定的加密方法，使用前面的三个随机数（client random, server random, premaster secret），生成『对话密钥』（session key），用来加密接下来的整个对话过程（对称加密）。

有一篇由浅入深介绍HTTPS的文章可以阅读一下：[看图学HTTPS](https://juejin.im/post/5b0274ac6fb9a07aaa118f49)



**问题一：为什么握手过程需要三个随机数，而且安全性只取决于第三个随机数？**

前两个随机数是明文传输，存在被拦截的风险，第三个随机数是通过证书公钥加密的，只有它是经过加密的，所以它保证了整个流程的安全性。前两个随机数的目的是为了保证最终对话密钥的『更加随机性』。

**问题二：Charles如何实现HTTPS的拦截？**

Charles要实现对https的拦截，需要在客户端安装Charles的证书并信任它，然后Charles扮演中间人，在客户端面前充当服务器，在服务器面前充当客户端。

**问题三：为什么有些HTTPS请求（例如微信）抓包结果仍是加密的，如何实现的？**

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200630100226.png)

我在聊天过程中并没有抓到会话的请求，在小程序启动的时候到是抓到了一个加密内容。我手动触发该链接会下载一个加密文件，我猜测这种加密是内容层面的加密，它的解密是由客户端完成的，而不是在HTTPS建立过程完成的。

另外在研究这个问题的过程中，又发现了一些有趣的问题：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200630101209.png)

1、图中所示的三个https请求分别对应三个不同类型的图标，它们分别代表什么意思呢？

2、第三个请求`https://mtalk.google.com:5228`图标和请求内容都加了锁，这个加锁是在https之上又加了一层锁吗？



这些问题暂时没有确切的答案，希望了解的小伙伴告知一下哈。

### DNS解析流程

DNS（Domain name system）域名系统。DNS是因特网上作为域名和IP地址相互映射的一个分布式数据库，能够使用户通过域名访问到对应的服务器（IP地址）。具体的解析流程是这样的：

1、浏览器中输入想要访问的网站域名，操作系统会检查本地hosts文件是否有这个网址的映射关系，如果有就调用这个IP地址映射，完成域名解析。没有的话就走第二步。

2、客户端回向本地DNS服务器发起查询，如果本地DNS服务器收到请求，并可以在本地配置区域资源中查到该域名，就将对应结果返回为给客户端。如果没有就走第三步。

3、根据本地DNS服务器的设置，采用递归或者迭代查询，直至解析完成。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200629170959.png)

其中递归查询和迭代查询可以用如下两图表示。

#### 递归查询

如图所示，递归查询是由DNS服务器一级一级查询传递的。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200629171133.png)

#### 迭代查询

如果所示，迭代查询是找到指定DNS服务器，由客户端发起查询。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200629171228.png)

#### DNS劫持

DNS劫持发生在DNS服务器上，当客户端请求解析域名时将其导向错误的服务器（IP）地址。

常见的解决办法是使用自己的解析服务器或者是将域名以IP地址的方式发出去以绕过DNS解析。



### Cookie和Session的区别

HTTP 是无状态协议，说明它不能以状态来区分和管理请求和响应。也就是说，服务器单从网络连接上无从知道客户身份。

可是怎么办呢？就给客户端们颁发一个通行证吧，每人一个，无论谁访问都必须携带自己通行证。这样服务器就能从通行证上确认客户身份了。这就是Cookie的工作原理。

* Cookie：Cookie是客户端保存用户信息的一种机制，用来记录用户的一些信息，实际上Cookie是服务器在**本地机器**上存储的一小段文本，并随着每次请求发送到服务器。**Cookie技术通过请求和响应报文中写入Cookie信息来控制客户端的状态。**

* Session：Session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构来保存信息。当有用户请求创建一个session时，服务器会先检查这个客户端里是否已经包含了一个Session标识（session id），如果有就通过session id把session检索出来。如果没有就创建一个对应此Session的session id。这个session id会在本次响应中返回给客户端。

两者有以下区别：

1、存储位置：Cookie存放在客户端上，Session数据存放在服务器上。

2、Session 的运行依赖 session id，而 session id 是存在 Cookie 中的，也就是说，如果浏览器禁用了 Cookie ，同时 Session 也会失效

3、安全性：Cookie存在浏览器中，可能会被一些程序复制，篡改；而Session存在服务器相对安全很多。

4、性能：Session会在一定时间内保存在服务器上，当访问增多，会对服务器造成一定的压力。考虑到减轻服务器压力，应当使用Cookie

### CDN

CDN（Content Delivery Network），根本作用是将网站的内容发布到最接近用户的网络『边缘』，以提高用户访问速度。概括的来说：CDN = 镜像（Mirror） + 缓存（Cache） + 整体负载均衡（GSLB）。

目前CDN都以缓存网站中的静态数据为主，如CSS、JS、图片和静态网页等数据。用户在从主站服务器请求到动态内容后再从CDN上下载这些静态数据，从而加速网页数据内容的下载速度，如淘宝有90%以上的数据都是由CDN来提供的。



#### CDN工作流程

一个用户访问某个静态文件（如CSS），这个静态文件的域名假如是www.baidu.com，而这个域名最终会被指向CDN全局中CDN负载均衡服务器，再由这个负载均衡服务器来最终分配是哪个地方的访问用户，返回给离这个访问用户最近的CDN节点。之后用户就直接去这个CDN节点访问这个静态文件了，如果这个节点中请求的文件不存在，就会再回到源站去获取这个文件，然后再返回给用户。


![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200629201214.png)

参考：[深入理解Http请求、DNS劫持与解析](https://juejin.im/post/59ba146c6fb9a00a4636d8b6)

### Socket

socket位于应用层和传输层之间：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200629183059.png)

它的作用是为了应用层能够更方便的将数据经由传输层来传输。所以它的本质就是对TCP/IP的封装，然后应用程序直接调用socket API即可进行通信。上文中说的三次握手和四次挥手即是通过socket完成的。

我们可以从iOS中网络库分层找到`BSD Sockets`，它是位于`CFNetwork`之下。在`CFNetwork`中还有一个`CFSocket`，推测是对`BSD Sockets`的封装。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200629184557.png)

### WebRTC

[WebRTC](https://webrtc.org/)是一个可以用在视频聊天，音频聊天或P2P文件分享等Web App中的 API。借助WebRTC，你可以在基于开放标准的应用程序中添加实时通信功能。它支持在同级之间发送视频，语音和通用数据，从而使开发人员能够构建功能强大的语音和视频通信解决方案。该技术可在所有现代浏览器以及所有主要平台的本机客户端上使用。WebRTC项目是[开源的，](https://webrtc.googlesource.com/src/)并得到Apple，Google，Microsoft和Mozilla等的支持。



### 如果某一请求只在某一地特定时刻失败率较高，会有哪些原因

这个是某公司二面时的问题，是一个开放性问题，我总结了以下几点可能：

1、该时刻请求量过大

2、该地的网络节点较不稳定

3、用户行为习惯，比如该时刻为上班高峰期，或者某个群体的特定习惯



如果有对网络方面比较熟悉的小伙伴也可以补充。
