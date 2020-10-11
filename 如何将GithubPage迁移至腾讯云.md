![](https://gitee.com/zhangferry/Images/raw/master/gitee/博客1920x1080.jpeg)

## 前置说明

一直以来都是用GithubPage搭建的博客，因为服务器在国外，访问速度一直比较慢，再后来有一批服务器被墙掉了导致国内网络环境直接无法访问。这里可以多说一句，GithubPage跟Github用的可以不是同一IP地址服务器，被墙很多是通过锁定IP进行的，GithubPage搭建博客有些可以访问有些不可以，就是因为某些IP被封了，如果更换IP也是可以解决的，但访问速度还是不够快。前段时间看到腾讯云服务器打折，就考虑如果想保证国内的访问速度还是要将服务器迁移到国内来。

这篇文章的作用是对搭建博客做一个整体介绍，对博客中用到的方案进行一些比较，有些会写的比较简略还需要再转到别文章去看，另外重点会写一下迁移腾讯云及域名备案需要做的具体操作。

博客的搭建一般需要关注这五个部分：博客框架、域名、服务器、图床、Markdown工具。各个环节都有免费和付费版本的选择，一般来说免费能满足基本需求，付费则能提供更优质的体验，如何选择就看大家的需求了。这五部分的对比如下：

|              | 付费版本                  | 免费版本             |
| ------------ | ------------------------- | -------------------- |
| 博客框架     | Wordpress（部分内容付费） | Hexo/Jekyll          |
| 域名         | 万网/腾讯云               | freenom/dot.tk       |
| 内容托管平台 | 阿里云/腾讯云             | GithubPage/GiteePage |
| 图床         | 七牛/又拍                 | Github/Gitee         |
| Markdown工具 | MWeb                      | Typora               |

接下来会围绕这五部分展开说明。
<!--more-->

## 博客框架

当前热门的方案是[Hexo](https://hexo.bootcss.com/ "Hexo")和[Jekyll](https://www.jekyll.com.cn/)，Hexo是基于Node.js开发的，Jekyll是基于Ruby开发的，他们的作用都是将Markdown语法的内容转译成HTML，并根据配置的主题进行显示。

[Wordpress](https://cn.wordpress.org/) 功能更强大，且相比前两个方案更简单，它可以不依赖Markdown语法，直接选用模板进行线上编辑即可。我对Wordpress摸索的较少，它的优缺点不便于多说，但还是推荐大家拿来试试。

这里介绍几个使用上述方案搭架的博客，大家可以参考看下：

Jekyll：https://onevcat.com/，https://draveness.me/

Hexo：http://blog.sunnyxx.com/，https://zhangferry.com/

Wordpress：http://blog.cnbang.net/

网站样式的差别不是由框架决定的，而是主题，以上三者都有大量的主题可供选择。我使用的是Hexo，使用方法可以参考这篇：[基于Hexo搭建自己的博客小屋](https://zhangferry.com/2016/12/20/build-blog-by-hexo/ "基于Hexo搭建自己的博客小屋")。

## 域名

### 域名选择

#### 免费版本

免费版本域名，像[freenom](https://www.freenom.com/zh/index.html "freenom")和[dot.tk](http://www.dot.tk/zh/index.html "dot.tk")，一般都有固定的使用时间，一般为一年左右，超过一年还想使用就需要付费了。

如果依赖GithubPage 或者 GiteePage 这样的托管服务，我们是可以设置像`username.github.io`或`username.gitee.io`这样的域名。

如果想尝试GithubPage的话，也可以参考这篇[基于Hexo搭建自己的博客小屋](https://zhangferry.com/2016/12/20/build-blog-by-hexo/ "基于Hexo搭建自己的博客小屋")

#### 付费版本

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201007144122.png)

付费版本的域名可以是我们常见的`.com`、`.cn`、`.net`等，可以到[万网](https://wanwang.aliyun.com/domain/)或者[腾讯云](https://buy.cloud.tencent.com/domain?from=console)进行购买，一般可选1-10年使用期限。

### 域名备案

不是所有域名都需要备案的，如果网站服务器在国外是不需要备案的，在国内是需要备案的。GithubPage服务器在国外，使用域名不需要备案；如果要使用国内服务器，腾讯云或者阿里云等则需要备案。

以下是腾讯云的域名备案流程：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201007145756.png)

主要步骤都可以通过备案小程序进行。

备案申请条件首先会提交到腾讯云后台，由他们进行一个初审，如果没有通过他们会联系你，并告知你哪里需要修改，修改之后可以再次提交。

我在初审时遇到了两个问题：

1、域名备案时需要是不可访问状态，如果当前正在使用，需要停止解析。

2、网站描述不能包含博客字样，可以用学习笔记代替。

腾讯云的初审通过之后，他们会将备案信息提交至管局。管局的审核会久一些，一般是20个工作日之内，我的审核时间用了16天。

#### 备案号

审核通过之后会获得一个备案号，我们需要在网站底部加上这个备案号并附带链接到域名信息备案管理系统，备案号可以在腾讯云的备案管理界面查看。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201008202332.png)

这里会会有两个地方显示备案号：主体信息和已备案网站。主体信息是自然人或单位的意思，已备案网站是针对网站的，因为一个主体可能会用到多个网站，所以网站备案号是以主体备案号为基础加上自然序号。如果再次备案域名，新的备案号会是`主体备案号-2`的形式。我们网站中出现的备案号应该是网站备案号，即`京ICP备2020035857号-1`。

接下来需要在网站中添加备案号，如果你使用的是Hexo或者Jeklly框架，可以在主题文件目录下搜索`Powered by`，找到对应的底栏配置文件。我使用的[Icarus](https://github.com/ppoffice/hexo-theme-icarus "Icarus")主题，其底部文件为`footer.jsx`，找到对应位置添加：

```html
<a href="https://beian.miit.gov.cn/" target="_blank" rel="noopener">&nbsp;京ICP备2020035857号-1</a>
```

效果如下：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201008204551.png)

#### 公安备案

在网站开通后的30日之内还需要进行公安备案，登录网站[全国互联网安全管理服务平台](http://www.beian.gov.cn/portal/index.do)，填写信息。因为是政府网站，开代理一般无法访问，需要关闭代理，如果仍无法访问，可能是DNS污染了，可以将DNS服务器设置成`114.114.114.114`再次刷新网站。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201008224024.png)

登录之后注册、登录并填写开办者信息：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201008205528.png)

这里会有很多坑，我尝试了很多次均已失败告终，直到现在也没成功，如果有知道怎么搞定的小伙伴可以告诉我，以下记录一些解决的坑：

* 图片选择时需要开启Adobe Flash
* 对身份证的识别好像是通过图片比例进行的，拍出来的身份证照片需要把多余的内容剪裁掉，只保留身份证主体内容
* 手持身份证照片如果不能验证通过可以按照实例图片剪裁成一定比例或者换个白色背景拍照
* 常住地址第二和第三个选项框有时无法弹出，这个只能不断刷新尝试，我隔几天再试地区弹框才能展开
* 填完之后点提交审核一直加载，1分钟之后还是卡着不动，试了几次都是同样的问题，这一步一直没成功

公安备案我看很多网站也没有写，所以暂时也不打算管了。这里可以告诉大家公安备案是指什么，以[掘金](https://juejin.im/ "掘金")为例：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201008205134.png)

上面箭头指向的`津ICP备`是网站备案号，`京公网安备`指的是公安备案，下面这个即是公安的备案号，很多博客网站都没有写好像重要性不是很大。

### 域名解析

域名解析可以使用腾讯的[DNSPod](https://console.dnspod.cn/dns/list)，因为我之前做过从GithubPage服务器的域名解析，这里只需将原来指向GithubPage的服务器IP地址该为自己的服务器地址即可。

关于域名的配置可以参考我之前的一篇博客：[为博客设一个自定义域名](https://zhangferry.com/2017/01/05/custom-domain-blog/ "为博客设一个自定义域名")

以下是我的域名解析配置，被擦掉的内容为服务器IP地址：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201007162947.png)

全球递归DNS服务器的刷新不超过72小时，我们验证解析是否生效可以通过

`ping zhangferry.com`命令进行，如果展示的ip地址是我们修改后的ip地址，就证明域名解析已经生效。

## 服务器

### 云服务器选择

分为免费和付费版本，其实免费版不属于服务器，只不过起到一个服务器托管的作用。

#### 免费版本

Github的GithubPage和码云的GiteePage是两个比较常见的静态网页托管服务。这里简单总结下两者的特点：

| 对比项     | GithubPage                           | GiteePage                  |
| ---------- | ------------------------------------ | -------------------------- |
| 默认域名   | username.github.io                   | username.gitee.io          |
| 自定义域名 | 支持                                 | 免费版不支持，付费版支持   |
| 访问速度   | 国内无法访问，翻墙之后访问速度也较慢 | 国内可访问，但访问速度较慢 |
| 热度       | 较高，有很多开发者使用这个方案       | 较低，使用者较少           |

如果你刚打算写博客，想尝尝鲜，我的建议是先从这俩平台中选一个，玩一玩。如果有长期维护的打算，并期望保证一定的用户体验，还是得购买服务器的。

#### 付费版本

可供选择的[阿里云](https://cn.aliyun.com/price/product#/ecs/detail)和[腾讯云](https://cloud.tencent.com/act/seckill?from=13337)，我选的是腾讯云，阿里云也是同理。

购买时我们需要指定操作系统，服务器的话一般都是Linux。有两种比较流行的Linux发行版：Ubuntu和CentOS，这两者在包管理工具上是不一样，在后面的安装工具过程中需要注意这一点。Ubuntu上用的包管理工具是`apt-get`，CentOS用的包管理工具是`yum`。

我选择的是CentOS，服务器创建完之后就可以登录了。腾讯云有两种登录方式

1、通过腾讯云网页端SSH登录，首次登录需设置密码，登录成功之后会进入一个远程终端页面，这里的登录时通过root用户登录的。

2、本机SSH登录，通过`ssh root@host-ip`的形式进行登录，之后需要输入首次登录创建的root用户密码。本机SSH可选择其他用户登录。

### 服务器配置

之后的配置和命令设置有些是在远程服务器操作的，有些是本地操作，下面会以`remote$`和`local$`开头进行区分，ip地址以`host-ip`的形式展示，用户名以`zhangferry`的形式展示。

#### 安装依赖与Git

首先登录服务器，安装一些依赖工具。

1、安装依赖库

```shell
remote$ yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
```

2、安装编译工具

```shell
remote$ yum install gcc perl-ExtUtils-MakeMaker package
```

3、安装Git

虽然Linux发行版一般都装有Git，但是很多版本都比较旧，我们需要一个相对新的Git版本，所以需要安装最新版本的Git。

查看Git版本

```shell
remote$ git --version
```

如果看到的版本是2.0之前的，那就升下级吧。

删除旧版Git

```shell
remote$ yum remove git
```

由于yum跟git版本更新不一致，可以选择源码编译升级，如果嫌麻烦也可以选择yum升级。当前最新版本是[2.28.0](https://git-scm.com/downloads "git download")。如果选择`yum`升级可以使用`yum install git`，逃过源码升级这一步即可。

**源码升级**

选择一个目录来存放下载下来的 git 安装包。这里选择了`/usr/local/src` 目录，我们进入到这个目录

```shell
remote$ cd usr/local/src
```

下载Git压缩包至这个目录

```shell
remote$ wget http://ftp.ntu.edu.tw/software/scm/git/git-2.28.0.tar.gz
```

解压到当前目录

```shell
remote$ tar -zvxf git-2.28.0.tar.gz
```

进入解压的目录，并编译

```shell
remote$ cd git-2.28.0
remote$ make prefix=/usr/local/git all
```

安装 git 到 /usr/local/git 目录下

```shell
remote$ make prefix=/usr/local/git install
```

设置Git环境变量，即可以在命令行找到Git命令，你可以试下

```shell
remote$ git --version
```

如果看到的版本号是我们安装的2.28.0，即证明当前环境变量设置正确，如果现实`comond not found`，我们需要修改环境变量使终端能够找到git命令。

使用vim打开

```shell
remote$ vi /etc/profile
```

按i进入编辑模式，在文件末尾增加下面内容

```shell
PATH=$PATH:/usr/local/git/bin   # git 的目录
export PATH
```

保存退出再次查看git版本号，应该就变成我们安装的版本了。

#### 创建新用户

创建用户的目的是为了便于分配权限，该用户用于远程登录，并推送网站内容。我们也可以直接用root用于进行直接登录，如果嫌创建用户麻烦，可以跳过这一步。

创建用户`zhangferry`，并设置连接密码：

```shell
remote$ adduser zhangferry
remote$ passwd zhangferry #该命令回车输入密码，有二次确认
```

获取sudoers文件的编辑权限

```shell
remote$ chmod 740 /etc/sudoers
remote$ vim /etc/sudoers
```

按 `i` 键进入文件的编辑模式，按向下键找到如下字段：

```
root    ALL=(ALL)       ALL
```

在其后面增加一句：

```
zhangferry     ALL=(ALL)       ALL
```

即我们新建的用户拥有了超级用户的权限，之后退回sudoers文件的编辑权限：

```shell
remote$ chmod 400 /etc/sudoers
```

切换用户的命令是：

```shell
remote$ su zhangferry #su root切换root用户
```

这时我们可以在本机使用ssh命令，试下是否可以通过zhangferry用户成功登录，需要我们输入刚才为zhangferry用户设置的密码：

```shell
local$ ssh zhangferry@host-ip
```

每次输入密码很麻烦，可以通过秘钥，进行免密登录。

#### SSH登录

这一步是非必须的，但推荐这样做，这会省去我们后面输入密码的操作。

1、本地生成秘钥，一般指定RSA加密算法，会生成一对公钥私钥。

```shell
local$ ssh-keygen -t rsa -C "zhangferry11@gmail.com" #这里邮箱需要换成自己的
```

如果已经生成过，不要再次生成了，因为它会覆盖我们原有的秘钥，导致之前的秘钥配置失效。检验方式是：

```shell
local$ cat ~/.ssh/id_rsa.pub
```

秘钥的位置是在.ssh文件夹内，如果有输出内容说明我们已经生成过。

2、复制ssh公钥至远程主机的`~/ .ssh/authorized_key`文件里

```shell
local$ ssh-copy-id root@host-ip
```

如果提示失败或者无文件，需要手动生成同名文件，将内容复制进去即可。

3、测试连接

```shell
local$ ssh zhangferry@host-ip
```

如果命令行左侧内容显示为带有[centos]或者标记我们远程服务器的字样即代表登录成功。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201008093524.png)

#### 创建git仓库和网站目录

1、这一步的目的是创建git仓库，本地博客内容需要推送至这里。

```shell
remote$ su root #在root权限里操作
remote$ mkdir /home/zhangferry
remote$ mkdir /home/hexo #网站目录
remote$ cd /home/zhangferry
remote$ git init --bare blog.git #创建git裸仓库
```

裸仓库的意思就是只保留git提交记录，不保留文件内容。

2、创建一个git的post hook，用于自动部署。这一步的作用是在我们往服务器推送仓库时，将内容转移到网站目录，因为裸仓库是不包含内容的。

```shell
remote$ vi blog.git/hooks/post-receive
```

hooks文件是git自动生成的目录，里面有很多用于编辑hook的示例，其中`post-receive`用于在`push`操作的hook。它的内容如下：

```shell
#!/bin/sh
git --work-tree=/home/hexo --git-dir=/home/zhangferry/blog.git checkout -f
```

这里需要注意，work-tree对应工作区的目录即网站目录，git-dir对应git仓库位置，如果你有自定义的命名，记得修改这两处地方。

这里大家可以想一下为什么要将git目录和网站目录分开处理，如果不分开也是可以的。我的理解是分开是为了便于权限管理，git的操作一般会是一个单独的用户权限，很多人都可以操作git，它对网站的影响是间接的，而网站内容的权限应该更高一些，只有root才可以直接操作。所以就将两者分开了。

3、修改git仓库权限

只需要将git仓库放权给zhangferry用户，网站目录的更新由post hook完成。

```shell
remote$ cd /home/zhangferry
remote$ chown zhangferry:zhangferry -R blog.git # blog.git的拥有者为zhangferry
remote$ chmod +x blog.git/hooks/post-receive # 修改post-receive为可执行
```

### 安装Nginx

Nginx是一个高性能的HTTP和反向代理web服务器，这里我们只将其作为HTTP web服务器使用。

1、安装nginx

```shell
remote$ yum install -y nginx
```

2、修改配置文件

```shell
remote$ vi /etc/nginx/nginx.conf
```

修改内容如下：

```
server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  zhangferry.com;   #博客域名
    root         /home/hexo;       #网站目录
    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;
    
    location / {
    }
        
    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

我们只需要修改server里的server_name和root选项，他们分别代表博客域名和网站目录。

listen代表监听端口，http默认监听的是80端口。error_page代表出错之后的展示页面，无数据的40x.html和服务器错误的50x.html都可以在这里指定。

3、nginx的几个命令

```shell
remote$ start nginx #启动nginx
remote$ nginx -t #检测配置文件是否有语法错误
remote$ nginx -s reopen #重启nginx
remote$ nginx -s stop #退出nginx
```

正常的流程应该是当我们修改完nginx的配置文件，先用`nginx -t`检测下是否有错误的配置，它还会检测一些待修复但不影响运行的警告信息，如果有错误需要修改再次检测。当配置没有问题了需要启动nginx：`start nginx`，之后的配置修改需要重启nginx：`nginx -s reopen`。

4、启动检测

nginx启动之后我们就可以在本地浏览器输入http://host-ip:80 进行访问，这时我们看到的应该是一个nginx的缺省页，因为`/home/hexo`目录并没有任何内容。

## 上传Hexo博客内容至腾讯云

修改博客根目录里`_config.yml`文件的deploy配置：

```yml
deploy:
    type: git
    repo: zhangferry@host-ip:/home/zhangferry/blog.git #上面配置的裸git目录
    branch: master
```

部署hexo，并推送：

```shell
local$ hexo g
local$ hexo d #发布hexo会触发push操作，将repo推送至我们配置的git路径
```

有可能你会出现`bash: git-receive-pack: command not found`这样的错误，`command not found`是一种常见的问题，上面的意思是在服务器方git-receive-pack命令无法找到。

> 简单扩展下，git push内部分为两步，本地运行send-pack，它会判断哪些提交记录是它有但服务器没有的，然后它告知服务器端的receive-pack本次更新内容，receive-pack接收推送来的数据。

网上对这一问题的解决方式是通过`ln`关联`git-receive-pack`：

```shell
remote$ sudo ln -s /usr/local/git/bin/git-receive-pack  /usr/bin/git-receive-pack
```

`/usr/local/git/bin/git-receive-pack`为实际的命令路径，`/usr/bin/git-receive-pack`为创建的快捷路径，但我并不能通过这个方法解决。

但从错误本身出发，命令安装了却找不到一般都是环境变量的问题，我直接修改环境变量：

```shell
remote$ su zhangferry #如果使用子用户推送的，需要切换至该用户
remote$ vi ~/.bashrc
```

在`.bashrc`底部增加`export PATH=$PATH:/usr/local/git/bin`，然后更新环境变量：

```shell
remote$ source ~/.bashrc
```

再次部署hexo，应该就会成功了。

```shell
local$ hexo d
```

我一开始想到了修改环境变量，但是在root下修改的，而root和zhangferry是两个隔离的环境变量配置！

关于环境变量文件之间的关系有一张图可供参考：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201008122558.png)

`/etc/bashrc`目录下的配置是全局的，`~/.bashrc` 下的配置只针对当前登录的用户。所以对于上面环境变量的修改，我们在全局或者zhangferry用户下修改都是可以的。

推送成功之后再次浏览器访问`http://host-ip:80`，如果我们域名DNS解析生效，直接访问http://zhangferry.com也一样，如果能正常显示博客首页即证明我们迁移成功了。

## HTTPS支持

这一步是非必须的，如果你不打算使网站支持https到这里就可以结束了。支持HTTPS需要三步：

1、申请SSL证书

2、将证书上传至服务器

3、配置Nginx关联SSL证书

以下是对这三步的详细说明。

### 申请SSL证书

腾讯云有免费的https证书可以申请，有效期只有一年，过期需要再申请。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200926113928.png)

这个申请流程很快，一般1个小时左右，如果时间过长，比如超过了一天，可以再次申请，这样他们会很快通过你的申请，并打回最早那次申请（我就是遇到了这种情况0。0）。

### 上传证书至服务器

证书通过之后有一个下载按钮，下载下来是一个包含多种服务器类型的证书文件。我是使用Nginx搭建的服务器，所以找到Nginx目录里的证书文件，它有两个：.crt后缀和.key后缀。

1、我们还需要在Nginx目录创建一个文件夹，用于存放这些证书：

```shell
remote$ cd /etc/nginx/
remote$ mkdir ssl
```

2、使用scp命令上传本地证书文件至服务器：

```shell
local$ scp /path/filename zhangferry@host-ip:/etc/nginx/ssl/
```

### 修改Nginx配置

网上相关教程很多，都是上来就告诉你怎么改，并没有说明其中的关系和原因，这样很容易因为一些配置环境不同导致错误。

我简单总结下：nginx有一个默认配置是通过`nginx.conf`控制，我们可以将证书配置写到这里面。但一个服务器允许配置多个域名，只有一个配置文件是不够的，解决方案是在`conf.d`目录下建一个以域名命令的`.conf`配置文件，它可以用来管理单独的域名，这两个配置文件是同级别的会一起生效。

我是移除了原有`nginx.conf`的配置项内容，将端口和证书的配置都写到了`conf.d`下的`zhangferry.conf`里。

nginx.conf内容：

```
server {
    listen       81 default_server;
    listen       [::]:81 default_server;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
    }
        
    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

这里移除了`server_name`和`root`，因为我们要在zhangferry.conf里配置监听，所以这里的配置项都去掉；将监听端口改成了81，是为了避免两处配置一样的端口引起警告。

zhangferry.conf内容：

```
server {
    listen 80;
    server_name zhangferry.com;
    root         /home/hexo;
    #重定向设置，如果使用http访问会重定向至同路径的https链接
    rewrite ^(.*)$ https://$host$1 permanent;
}

server {
    listen 443 ssl;
    #证书位置，需跟自己证书位置对应
    ssl_certificate /etc/nginx/ssl/1_host.com_bundle.crt;
    ssl_certificate_key /etc/nginx/ssl/2_host.com.key;
    #绑定域名
    server_name zhangferry.com;
    root        /home/hexo;
        #ssl配置，以下几项必须要有
    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;

    location / {
    }
}
```

分别为80和443接口进行配置，配置完之后通过以下命令验证配置是否有问题：

```shell
remote$ nginx -t -c /etc/nginx/nginx.conf
```

如果出现`test is successful`字样即是没问题。

重启nginx服务器：

```shell
remote$ nginx -s reopen
```

访问https://zhangferry.com和http://zhangferry.com验证是否生效，如果出现加锁即证明配置成功了：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200926122008.png)

## 图床

### 图床选择

图床一定要考虑持久性，像是免费的临时方案和存到一些非图床的网站上都是不靠谱的。我们访问某些博客时应该遇到过图片开裂无法查看的情况，这些都是因为之前使用的图片存储被删除或者被更改了位置导致的。我的博客之前是用的简书的图床，现在很多也都无法查看了。所以下面介绍的方案会优先考虑存储的稳定性和持久性。

#### 免费版本

免费版有Github和Gitee可供选择，即把git仓库当做图片存储。

Github做图床时加载速度会比较慢，这个可以通过 [jsDeliver](https://www.jsdelivr.com/ "jsDeliver") 这个CDN服务进行解决，我之前很多github的图片链接都换成了通过jsDeliver的CDN链接。

另外一个方案是Gitee，这个正常使用就可以了，因为服务器在国内，访问速度还是比较快的。但Gitee有一个限制，图片超过1M就无法显示了，所以对于一些特定图片我们需要做好剪裁和缩放。

Github+jsDeliver和Gitee都是比较推荐的图床方案。

#### 付费版本

有[七牛云](https://www.qiniu.com/products/kodo "七牛云 Kodo")，[又拍云](https://www.upyun.com/products/file-storage "又拍云存储")，[腾讯云](https://console.cloud.tencent.com/cos5 "腾讯云存储")可供选择，付费版能提供更快的访问速度和更高的稳定性，但因为上面的免费方案已经满足我的需求了，所以就没做这些尝试，不同图床方案使用起来基本一致。

### 图床工具

图床的使用最好搭配一些趁手的工具，使我们可以通过截屏，拖动等操作轻易的上传指定图片至图床，这个工具就是[PicGo](https://github.com/Molunerfinn/PicGo "PicGo")。有一个配置教程可以参考：[一次艰难的图床选择经历(MWeb+PicGo+Github)](https://juejin.im/post/6844903779955900424 "一次艰难的图床选择经历")。

这个是一年前的文章，现在我使用的方案`Typora+PicGo+Gitee`，他们之间并没有太大的变化。

## Markdown工具

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201009214325.png)

我之前使用的是[MWeb](https://zh.mweb.im/ "MWeb")，但MWeb功能有点太多了，直到发现[Typora](https://www.typora.io/ "Typora")，我一眼就相中了这个界面简洁的Markdown编辑工具，关键还是免费的，爱了爱了。别看Typora界面简洁其实功能不少，它可以搭配PicGo插件一起使用，使图片的管理更加简便。因为是工具型东西，就不写教程了，大家多多摸索应该就会了。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/wechat_official.png)
