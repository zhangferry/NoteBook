---
title: 将黑苹果系统升级至macOS 10.14.4(Mojave)
date: 2019-05-17 17:32:57
tags: 
- 黑苹果
comments: true
---

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20190517203841.png)
## 原有配置
操作系统: macOS 10.13.6
主板：AORUS MASTER z390
CPU: intel core i7-8700
显卡：NVIDIA GTX 750
内存：Geil 8G * 2
硬盘：tigo SSD 240G

如果要升10.14的系统，首先需要确认的是当前显卡是否支持。
<!--more-->
可以参照这个表[Mojave硬件支持列表（持续更新中）](https://blog.daliansky.net/Mojave-Hardware-Support-List.html)

可以看到GTX 750已经无法驱动了，所以显卡要换。最好换成免驱的，我这里选了RX 560D。

**注意事项**
gtx750是即插即用，而rx560d有专门的供电线，所以更换显卡时一定要注意别忘了插rx560d的供电线。

## 升级准备
不管是装黑苹果还是升级黑苹果都需要有一个启动U盘，它可以在我们系统配置错误无法进入的时候，帮助我们通过U盘进入，然后我们再把配置改回来就行了。

### 制作启动U盘（大于8G)

1、插入U盘
2、打开 /Applications/Utilities/Disk Utility（磁盘工具）
3、选中U盘
![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20190517181447.png)
4、点击上方 Erase 选项按钮
5、你可以修改U盘名称
6、Format：选择Mac OS Extended(Journaled)，中文对应：Mac OS扩展(日志)；
Scheme：选择GUID Partition Map，中文对应：GUID 分区映射
![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20190517182006.png)
7、点击Erase按钮
![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20190517182411.png)
8、下载[UniBeast](https://www.tonymacx86.com/resources/unibeast-9-2-0-mojave.426/)
UniBeast版本要跟系统版本对应，需要注册tonymacx86账号才能下载。
9、安装UniBeast，需要把系统需要设置成英文才能进行安装。设置完毕，一路Continue。
![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20190517183045.png)
10、根据提示，选择Installation Type/Bootloader Configuration/Graphics Configuration然后完成，开始Copy系统文件。
![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20190517183045.png)
11、把[MultiBeast](https://www.tonymacx86.com/resources/multibeast-11-2-1-mojave.425/)拖进U盘

翻译自[tonymacx86](https://www.tonymacx86.com/threads/unibeast-install-macos-mojave-on-any-supported-intel-based-pc.259381/)
### 升级Clover
如果想要安装macOS Mojave 10.14，它要求你的`Clover Bootloader`版本不低于r4515。最新的Clover版本可以在这[下载](https://github.com/Dids/clover-builder/releases)。

## 升级系统
下载好升级程序之后，直接进行安装。会重启两次，之后是较长一段时间的等待（20-30分钟），跟正常macbook升级一样的流程。如果没有意外，那么恭喜你，黑苹果升级成功了。
![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20190517203514.png)
## 可能的问题
### 关机无法断电
这个问题网上有的说在config文件中的Acpi将FixShutdown选为true，有的说是增加电量修复的efi文件。我都试过，均无效，我看这方面的回答时间都比较久，应该是旧版本的解决方案。新版本只需要将FixShutdown制为false即可。

### 安装过程卡在最后2分钟或者卡在最开始18分钟
在EFI的drivers64UEFI文件中增加`OsxAptioFixDrv3-64.efi`文件即可。
完整的[EFI](https://pan.baidu.com/s/1faSAQm7RbTGRqc6_y6P9Ug)文件，提取码：awxl。

### 如果因为配置出错无法进入系统
在BIOS界面选择U盘启动，即可通过U盘配置的EFI进入系统。然后更改正确设置即可。

### 重要文件记得备份
* Time Machine 免费
* [Carbon Copy Cloner](https://bombich.com/) ￥290.15
* [Super Duper](https://www.shirt-pocket.com/SuperDuper/SuperDuperDescription.html) $27.95

Time Machine因为是苹果自带的功能，而且还免费，比较推荐使用这个。附一份[教程](https://support.apple.com/zh-cn/HT201250)
### 其他问题
当然配置黑苹果的机型组合有很多种，可能会遇到各式各样的问题。这里再贴几个可以参考的链接：
[macOS Mojave 10.14安装中常见的问题及解决方法](https://blog.daliansky.net/Common-problems-and-solutions-in-macOS-Mojave-10.14-installation.html)
[Hackintosh黑苹果驱动Clover](https://github.com/tsingui/clover-efi)
