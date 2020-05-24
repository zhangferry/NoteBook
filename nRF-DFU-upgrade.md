---
title: nRF芯片设备DFU升级（适配Xcode10.2.1）
date: 2016-12-28 22:17:59
tags: 
    - 蓝牙
    - DFU
categories: 蓝牙总结
comments: true
---
![Nordic.png](https://user-gold-cdn.xitu.io/2019/4/17/16a2972de5721a39?w=712&h=262&f=png&s=18797)

>这里主要参考这个项目：[iOS-nRF-Toolbox](https://github.com/NordicSemiconductor/IOS-nRF-Toolbox)，它是Nordic公司开发的测试工程，包含一整套nRF设备的测试解决方案。

<!--more-->

项目是用Swift写的，不过之前还是有OC版本的，但是后来由于一些**（不可描述的问题），才变成了现在的纯Swift版本。对于使用Swift开发的人员，直接仿照Demo操作即可。如果你是用Swift开发的，那下面的内容你可以不用看了。接下来我就讲一下针对OC引用DFU升级的操作步骤和我遇到的问题。

## 代码研究
nRF-Toolbox项目包含BGM，HRM，HTM，DFU等多个模块，我们今天只关注其中的DFU升级模块。打开项目，在对应的`NORDFUViewController.swift`中我们能够看到有三个引用库
`import UIKit`,`import CoreBluetooth`,`import iOSDFULibrary`，这里的[iOSDFULibrary](https://github.com/NordicSemiconductor/IOS-Pods-DFU-Library)就是DFU升级的库，也是解决DFU升级最重要的组件。我们只要把这个库集成到我们的项目中，就能够完成nRF设备的DFU升级了。

## 集成步骤
有两种方案集成：
* 通过cocoapods集成
* 编译出framework然后把库导入项目

第一种方案是作者推荐的，但是我试了很久，引入DFULibrary会出现头文件找不到等一系列问题，无奈只能放弃，如果有人通过这种方式成功，还望告知。下面讲的是通过第二种方案的集成。

**第一步：导出iOSDFULibrary**

这一步是最关键也是最容易出问题的，这个库也是由Swift写成的，我们将这个库clone到本地，然后选择iOSDFULibrary进行编译

![01.png](https://user-gold-cdn.xitu.io/2019/4/17/16a2972c0da5d4ef?w=410&h=175&f=png&s=40355)

最后生成两个framework:
* iOSDFULibrary.framework
* Zip.framework

这时库内的代码已经变成了我们熟悉的OC语言。理论上这个库应该是没问题的了，但是事实还是有问题的，见[issues#39](https://github.com/NordicSemiconductor/IOS-Pods-DFU-Library/issues/39)。作者给出的解决方法是：
>1、On your mac please install carthage ([instructions](https://github.com/Carthage/Carthage#installing-carthage))
   2、Create a file named cartfile anywhere on your computer
   3、add the following content to the file:

   >     github "NordicSemiconductor/IOS-Pods-DFU-Library" ~> 2.1.2
   >     github "marmelroy/Zip" ~> 0.6
 1、Open a new terminal and cd to the directory where the file is
 2、Enter the command carthage update --platform iOS
 3、Carthage will now take care of building your frameworks, the produced .framework files will be found in a newly created directory called Carthage/Build/iOS,copy over iOSDFULibrary.framework and Zip.framework to your project and you are good to go.

[carthage](https://github.com/Carthage/Carthage#installing-carthage)是一种和cocoapods相似的的类库管理工具，如果不会使用的话可以参照Demo，将framework文件导入到自己的项目。

**第二步、导入framework**
Target->General
![128F494E-C863-49E7-AC44-A7B53B3EB463.png](https://user-gold-cdn.xitu.io/2019/4/17/16a2972c0c8a0efa?w=718&h=283&f=jpeg&s=15809)

直接拖入项目默认只会导入到`Linked Frameworks and Libraries`，我们还需要在Embeded Binaries中引入。

**第三步、使用iOSDFULibrary**
```Objective-c
//create a DFUFirmware object using a NSURL to a Distribution Packer(ZIP)
DFUFirmware *selectedFirmware = [[DFUFirmware alloc] initWithUrlToZipFile:url];// or
//Use the DFUServiceInitializer to initialize the DFU process.
DFUServiceInitiator *initiator = [[DFUServiceInitiator alloc] initWithCentralManager: centralManager target:selectedPeripheral];
[initiator withFirmware:selectedFirmware];
// Optional:
// initiator.forceDfu = YES/NO; // default NO
// initiator.packetReceiptNotificationParameter = N; // default is 12
initiator.logger = self; // - to get log info
initiator.delegate = self; // - to be informed about current state and errors 
initiator.progressDelegate = self; // - to show progress bar
// initiator.peripheralSelector = ... // the default selector is used

DFUServiceController *controller = [initiator start];
```
库中有三个代理方法`DFUProgressDelegate`，`DFUServiceDelegate`，`LoggerDelegate`，它们的作用分别为监视DFU升级进度，DFU升级及蓝牙连接状态，打印状态日志。

## 常见问题
1、__selectedFirmware返回nil__
```Objective-c
DFUFirmware *selectedFirmware = [[DFUFirmware alloc] initWithUrlToZipFile:url];
```
需要在`General`的`Embeded Binaries`选项卡里导入那`Zip.framework`和`iOSDFULibrary.framework`
2、__崩溃报错__
```ruby
dyld: Library not loaded: @rpath/libswiftCore.dylibReferenced from: /private/var/containers/Bundle/Application/02516D79-BB30-4278-81B8-3F86BF2AE2A7/XingtelBLE.app/Frameworks/iOSDFULibrary.framework/iOSDFULibraryReason: image not found
```
需要改两个地方
![error1.png](https://user-gold-cdn.xitu.io/2019/4/17/16a2972c0dc16473?w=1068&h=324&f=png&s=45942)

![erro2.png](https://user-gold-cdn.xitu.io/2019/4/17/16a2972c0deed50a?w=1148&h=308&f=png&s=46299)
如果不起作用，将Runpath Search Paths的选项内容删掉再重新添加一遍即可
3、__打包上架时报ERROR IT MS-90087等问题__
问题描述：

```ruby
ERROR ITMS-90087: "Unsupported Architectures. The executable for ***.app/Frameworks/SDK.framework contains unsupported architectures '[x86_64, i386]'."
ERROR ITMS-90362: "Invalid Info.plist value. The value for the key 'MinimumOSVersion' in bundle ***.app/Frameworks/SDK.framework is invalid. The minimum value is 8.0"
ERROR ITMS-90209: "Invalid Segment Alignment. The app binary at '***.app/Frameworks/SDK.framework/SDK' does not have proper segment alignment. Try rebuilding the app with the latest Xcode version."
ERROR ITMS-90125: "The binary is invalid. The encryption info in the LC_ENCRYPTION_INFO load command is either missing or invalid, or the binary is already encrypted. This binary does not seem to have been built with Apple's linker."
```
解决方法，添加Run Script Phase

![error3.png](https://user-gold-cdn.xitu.io/2019/4/17/16a2972c0dc15f54?w=1220&h=390&f=png&s=70833)
Shell脚本内容填写如下内容，再次编译即可

```shell
APP_PATH="${TARGET_BUILD_DIR}/${WRAPPER_NAME}"

# This script loops through the frameworks embedded in the application and
# removes unused architectures.
find "$APP_PATH" -name '*.framework' -type d | while read -r FRAMEWORK
do
FRAMEWORK_EXECUTABLE_NAME=$(defaults read "$FRAMEWORK/Info.plist" CFBundleExecutable)
FRAMEWORK_EXECUTABLE_PATH="$FRAMEWORK/$FRAMEWORK_EXECUTABLE_NAME"
echo "Executable is $FRAMEWORK_EXECUTABLE_PATH"

EXTRACTED_ARCHS=()

for ARCH in $ARCHS
do
echo "Extracting $ARCH from $FRAMEWORK_EXECUTABLE_NAME"
lipo -extract "$ARCH" "$FRAMEWORK_EXECUTABLE_PATH" -o "$FRAMEWORK_EXECUTABLE_PATH-$ARCH"
EXTRACTED_ARCHS+=("$FRAMEWORK_EXECUTABLE_PATH-$ARCH")
done

echo "Merging extracted architectures: ${ARCHS}"
lipo -o "$FRAMEWORK_EXECUTABLE_PATH-merged" -create "${EXTRACTED_ARCHS[@]}"
rm "${EXTRACTED_ARCHS[@]}"

echo "Replacing original executable with thinned version"
rm "$FRAMEWORK_EXECUTABLE_PATH"
mv "$FRAMEWORK_EXECUTABLE_PATH-merged" "$FRAMEWORK_EXECUTABLE_PATH"

done
```
## 完整OC项目
这个是对应Swift版本用OC写的完整项目，应该是OC停止维护之前的版本。会有一些bug。在将DFUFramework更新之后，我把它搬到了我的github上，有需要的同学可以下载研究：[OC-nRFTool-box](https://github.com/zhangferry/nRF-Toolbox)。

---
以下为更新内容，时间：2017.12.26
收到很多关于无法适配Xcode9.2的反馈，因为最近比较忙没时间处理，不好意思啦，今天抽出时间来把代码更新了一下。
## Xcode9.2 出现的问题
> 1、dyld: Library not loaded: @rpath/libswiftCore.dylib
Referenced from: /private/var/containers/Bundle/Application/02516D79-BB30-4278-81B8-3F86BF2AE2A7/XingtelBLE.app/Frameworks/iOSDFULibrary.framework/iOSDFULibrary
Reason: image not found
2、DFUFirmware *selectedFirmware = [[DFUFirmware alloc] initWithUrlToZipFile:url]; 返回为空或者崩溃问题

我的测试结果是更新iOSDFULibrary. framework和Zip.framework可以解决以上问题。
## 解决方案 Carthage
因为这两个库都是通过Swift维护的，所以更新framework最好还是要用适用Swift的方式，包括以后的更新也一样。所以我推荐用Carthage更新这俩库，下面是使用Carthage的简单介绍，详细的可以看这里[Carthage的安装和使用](https://www.jianshu.com/p/a734be794019)。
另外[OC-nRFTool-box](https://github.com/zhangferry/nRF-Toolbox)也已经更新，里面的Framework可以直接拿来用。

1、安装brew

```shell
 /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
2、brew更新

```shell
$ brew update
```
3、安装Carthage

```shell
$ brew install carthage
```
4、使用Carthage
      
```shell
$ cd ~/路径/项目文件夹 /**进入项目文件夹下*/
$ touch Cartfile /**创建carthage文件*/
$ open Cartfile /**打开carthage文件*/
# /**输入以下内容*/
$ github "NordicSemiconductor/IOS-Pods-DFU-Library" ~> 4.1
$ github "marmelroy/Zip" ~> 1.1
```
5、运行Carthage

```shell
$ carthage update --platform iOS /**编译出iOS版本*/
```
6、更新framework

```shell
$ cd Carthage/Build/iOS  /**framework输出位置，将老的framework替换掉*/
```

**注意**
nRF Toolbox项目方法变更

```ObjectiveC
[initiator withFirmwareFile:selectedFirmware];/** 旧版本方法 */
initiator = [initiator withFirmware:selectedFirmware];/** 新版本方法 */
```

## 更新：2019-7-14

针对之前常出的这种问题：
```ruby
dyld: Library not loaded: @rpath/libswiftCore.dylib
Referenced from: /private/var/containers/Bundle/Application/CDB2F4ED-C49C-4303-BE1F-5D9D990380F3/nRF Toolbox.app/Frameworks/Zip.framework/Zip
Reason: image not found
```
均是由Swift库版本不一致引起的，`iOSDFULibrary`目前已经支持到`Swift 5`，所以我们应该升级一下版本。为了方便使用，我将`Carthage`集成到了项目里，如果以后需要再升级，更新Cartfile文件里的版本号，执行更新命令：
```bash
$ carthage update --platform iOS
```

如果你想要将DFU的framework集成到你自己的项目里，可以在`Carthage/Build/iOS/`中找到`iOSDFULibrary.framework`, `ZIPFoundation.framework` 将其拖到项目中即可。
### 上线注意事项（由@jianxiong1997提供）
![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/9fd65f10e4416cff5fe61ea9cf1ca4f0.jpg)
需要删除`ZIPFoundation.framework`中的
* libswiftRemoteMirror.dylib
* Frameworks

[github](https://github.com/zhangferry/nRF-Toolbox)项目同步更新。


