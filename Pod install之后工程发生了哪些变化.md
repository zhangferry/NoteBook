# Pod install之后工程发生了哪些变化

CocoaPods是iOS开发中经常被用到的第三方库管理工具，我们有必要深入了解一下它都引入了哪些东西，以及它是如何将第三方库跟我们的项目做结合的。



## 使用pod安装三方库

我们新建一个不带测试模块的名为FFDemo的Swift项目，它的目录结构是这样的

```
├── FFDemo
│   ├── AppDelegate.swift
│   ├── Assets.xcassets
│   ├── Base.lproj
│   ├── Info.plist
│   ├── SceneDelegate.swift
│   └── ViewController.swift
└── FFDemo.xcodeproj
    ├── project.pbxproj
    ├── project.xcworkspace
    └── xcuserdata
```

然后我们执行`pod init`创建一个Podfile模板，在里面引入这两个三方库：

```
target 'FFDemo' do
  # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!

  # Pods for FFDemo
  pod 'MJRefresh', '~> 3.5.0'
  pod 'Moya'

end
```

成功执行`pod install`之后我们就将这两个库引入到了项目，这时项目目录变成了这样：

```
├── FFDemo
│   ├── AppDelegate.swift
│   ├── Assets.xcassets
│   ├── Base.lproj
│   ├── Info.plist
│   ├── SceneDelegate.swift
│   └── ViewController.swift
├── FFDemo.xcodeproj
│   ├── project.pbxproj
│   ├── project.xcworkspace
│   └── xcuserdata
├── FFDemo.xcworkspace
│   └── contents.xcworkspacedata
├── Podfile
├── Podfile.lock
└── Pods
    ├── Alamofire
    ├── Headers
    ├── Local\ Podspecs
    ├── MJRefresh
    ├── Manifest.lock
    ├── Moya
    ├── Pods.xcodeproj
    └── Target\ Support\ Files
```

从目录看，除了pod init引入了Podfile，其余三部分内容：FFDemo.xcworkspace、Podfile.lock、Pods目录都是由pod install之后生成的。我们下面重点讲下这三部分内容。

## xcworkspace文件

该文件下包含一个叫`contents.xcworkspacedata`的文件，它的内容是这样的：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Workspace
   version = "1.0">
   <FileRef
      location = "group:FFDemo.xcodeproj">
   </FileRef>
   <FileRef
      location = "group:Pods/Pods.xcodeproj">
   </FileRef>
</Workspace>
```

使用`xml`格式将依赖包含在<workspace>标签内。

xcworkspace是一个项目容器，当有多个project需要相互依赖时可以用xcworkspace将它们组织起来。pod在首次安装三方库时会生成一个叫`Pods.xcodeproj`的project管理三方库，然后将该project和主项目的project通过workspace进行关联。这样我们就可以在主工程里引入三方库了，而且三方库由Pods.xcodeproj统一管理，不会对我们原项目产生任何干扰。

## Podfile.lock

Podfile.lock文件的内容是这样的：

```
PODS:
  - Alamofire (5.3.0)
  - MJRefresh (3.5.0)
  - Moya (14.0.0):
    - Moya/Core (= 14.0.0)
  - Moya/Core (14.0.0):
    - Alamofire (~> 5.0)

DEPENDENCIES:
  - MJRefresh (~> 3.5.0)
  - Moya

SPEC REPOS:
  trunk:
    - Alamofire
    - MJRefresh
    - Moya

SPEC CHECKSUMS:
  Alamofire: 2c792affbdc2f18016e08fdbcacd60aebe1ba593
  MJRefresh: 6afc955813966afb08305477dd7a0d9ad5e79a16
  Moya: 5b45dacb75adb009f97fde91c204c1e565d31916

PODFILE CHECKSUM: 073f3d6d9f03e6a76838ca3719df48ae6cc01450

COCOAPODS: 1.9.3
```

因为Podfile文件里可以不指定版本号，而版本信息又很重要，于是就有了Podfile.lock，它里面记录完整的版本信息和依赖关系。它的内容包含以下几大块

### PODS

`PODS`是指当前引用库的具体版本号，可以发现我们并没有引入Alamofire，但在PODS里确有它。这是因为Moya中依赖了它，Moya里定义了一个subspec叫Core，这是Moya/Core写法的由来。pod是通过各个库的podspec文件找到对应依赖的，这里可以简单看下Moya的部分podspeec文件内容[Moya.podspec](https://github.com/Moya/Moya/blob/master/Moya.podspec)：

```
Pod::Spec.new do |s|
  s.default_subspecs = "Core"

  s.subspec "Core" do |ss|
    ss.source_files  = "Sources/Moya/", "Sources/Moya/Plugins/"
    ss.dependency "Alamofire", "~> 5.0"
    ss.framework  = "Foundation"
  end
end
```

### DEPENDENCIES

DEPENDENCIES为pod库的描述信息，这里内容是同Podfile里的写法。因为我们指定了MJRefresh的版本号，并没有指定Moya的版本号，所以这里内容也是一样的。

### SPEC REPOS

这里描述的是仓库信息，即安装了哪些三方库，他们来自于哪个仓库。

trunk是共有仓库的名称，它的地址是`https://github.com/CocoaPods/Specs.git`，外部使用的三方库大都来自于这里。通常我们还会依赖一些公司内部的私有库，私有库的信息也会显示在这里。

### SPEC CHECKSUM

这里描述的是各个三方库的校验和，校验和的算法是对当前安装版本的三方库的podspec文件求SHA1。比如MJRefresh的校验和：`6afc955813966afb08305477dd7a0d9ad5e79a16`。我们安装的MJRefresh的版本为3.5.0，它本地的podspec文件路径为：`~/.cocoapods/repos/trunk/Specs/0/f/b/MJRefresh/3.5.0/MJRefresh.podspec.json`。

这个路径可以通过`pod install --verbose`里的日志查看。我们对该文件内容通过openssl求sha1摘要：

```shell
$ pod ipc spec ~/.cocoapods/repos/trunk/Specs/0/f/b/MJRefresh/3.5.0/MJRefresh.podspec.json | openssl sha1
$ 6afc955813966afb08305477dd7a0d9ad5e79a16
```

因为是对json内容求sha1，所以json内容只要发生一点变化，得出的校验和就将大不相同，而这也是校验和设计的目的：用于跟踪版本信息是否发生了变化。

大家可能注意到了，控制版本信息的配置文件是podspec格式的，为什么本地文件变成了json格式？

这是因为json格式兼容性更高也更容易批量处理，Spec仓库的所有库配置文件都是被转成json格式的。在我们制作私有库的时候虽然可以直接以podspec的格式推上去，但后续解析文件时pod内部还是会把它转成json格式，所以建议私有库的配置文件也转成json格式再推送。

podspec转成json可以使用这个命令：

```shell
$ pod ipc spec ModuleName.podspec
```

### PODFILE CHECKSUM

这个校验和是针对Podfile内容的校验和，如果Podfile内容改变了，该值也会跟着改变。计算方法为：

```shell
$ openssl sha1 /Users/zhangferry/Desktop/FFDemo/Podfile
```

### COCOAPODS: 1.9.3

这个代表当前使用的CocoaPod版本号。

## Pods

### Manifest.lock

Manifest.lock是Podfile.lock的副本，它是在Pods目录里面。它的作用是这样的，我们通常是不把Pods文件放到版本管理里面，而把Podfile.lock放到版本管理里面。这时对于拉取代码之后是否需要更新pod，就可以通过对比本地的Manifest.lock和远程Podfile.lock是否相同即可。





## Build Phases

这里是设置编译阶段配置的地方，当首次pod install成功之后，这里会多几个[CP]开头的配置项（CP即CocoaPods缩写），它们都是由CocoPods安装的，他们都是脚本内容，执行顺序从上到下。

#### [CP] Check Pods Manifest.lock

该脚本位于较上方，如果没有Dependencies，开始编译就会执行该脚本，它的内容如下：

```shell
diff "${PODS_PODFILE_DIR_PATH}/Podfile.lock" "${PODS_ROOT}/Manifest.lock" > /dev/null
if [ $? != 0 ] ; then
    # print error to STDERR
    echo "error: The sandbox is not in sync with the Podfile.lock. Run 'pod install' or update your CocoaPods installation." >&2
    exit 1
fi
# This output is used by Xcode 'outputs' to avoid re-running this script phase.
echo "SUCCESS" > "${SCRIPT_OUTPUT_FILE_0}"
```

作用是比较`Podfile.lock`和`Manifest.lock`文件是否相同，如果不同就输出错误信息：`error: The sandbox is not in sync with the Podfile.lock. Run 'pod install' or update your CocoaPods installation.`，并执行退出，这会导致后续项目报错，无法继续编译。

该错误较常见，出现于拉取远端代码，远端pod依赖于本地不一致的情况。这时我们可以根据提示，执行`pod install`命令，根据Podfile及远端Podfile.lock生成新的Manifest.lock文件。



#### [CP] Resources





#### [CP] Embed Pods Frameworks

该脚本是直接运行Pods-FFDemo-frameworks.sh脚本。

pod如何构建。



input 和 output是Xcode 10自带功能，用于指定脚本的输入输出。这是配合New Build System推出的功能，用于提升编译效率。

### Targets Support Files

#### modulemap



#### acknowledgements



#### xcconfig



