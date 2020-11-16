# CocoaPods对三方库的管理探究

CocoaPods是iOS开发中经常被用到的第三方库管理工具，我们有必要深入了解一下它对项目产生了什么影响，以及它是如何工作的。

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

## CocoaPods安装的内容

### xcworkspace文件

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

### Podfile.lock

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

#### PODS

`PODS`是指当前引用库的具体版本号，可以发现我们并没有引入Alamofire，但在PODS里确有它。这是因为Moya中依赖了它，Moya里定义了一个subspec叫Core，这是Moya/Core写法的由来。pod是通过各个库的podspec文件找到对应依赖的，这里可以简单看下Moya的部分podspeec文件内容[Moya.podspec](https://github.com/Moya/Moya/blob/master/Moya.podspec)：

```ruby
Pod::Spec.new do |s|
  s.default_subspecs = "Core"

  s.subspec "Core" do |ss|
    ss.source_files  = "Sources/Moya/", "Sources/Moya/Plugins/"
    ss.dependency "Alamofire", "~> 5.0"
    ss.framework  = "Foundation"
  end
end
```

#### DEPENDENCIES

DEPENDENCIES为pod库的描述信息，这里内容是同Podfile里的写法。因为我们指定了MJRefresh的版本号，并没有指定Moya的版本号，所以这里内容也是一样的。

#### SPEC REPOS

这里描述的是仓库信息，即安装了哪些三方库，他们来自于哪个仓库。

trunk是共有仓库的名称，它的地址是`https://github.com/CocoaPods/Specs.git`，外部使用的三方库大都来自于这里。通常我们还会依赖一些公司内部的私有库，私有库的信息也会显示在这里。

#### SPEC CHECKSUM

这里描述的是各个三方库的校验和，校验和的算法是对当前安装版本的三方库的podspec文件求SHA1。比如MJRefresh的校验和：`6afc955813966afb08305477dd7a0d9ad5e79a16`。我们安装的MJRefresh的版本为3.5.0，它在本地的podspec文件路径为：`~/.cocoapods/repos/trunk/Specs/0/f/b/MJRefresh/3.5.0/MJRefresh.podspec.json`。

这个路径可以通过在安装库时增加` --verbose`参数在输出日志里查看。我们对该文件内容通过openssl求sha1摘要：

```shell
$ pod ipc spec ~/.cocoapods/repos/trunk/Specs/0/f/b/MJRefresh/3.5.0/MJRefresh.podspec.json | openssl sha1
$ 6afc955813966afb08305477dd7a0d9ad5e79a16
```

因为是对podspec.json内容求sha1，所以只要内容发生一点变化，得出的校验和就将大不相同，而这也是校验和设计的目的：podspec文件发生变化意味着版本信息发生了变化，就需要重新同步代码。

大家可能注意到了，我们通常制作私有pod，控制配置信息的文件是podspec格式的，为什么本地文件变成了json格式？

这是因为json格式兼容性更高也更容易批量处理，官方Spec仓库的所有库配置文件都是被转成json格式的。在我们制作私有库的时候是可以直接以podspec的格式推到远程仓库的，但后续解析文件时pod内部检索还是会把它转成json格式。上面的命令是包含了podsepc转json的命令的，转json命令如下：

```shell
$ pod ipc spec ModuleName.podspec
```

#### PODFILE CHECKSUM

这个校验和是针对Podfile内容的校验和，如果Podfile内容改变了，该值也会跟着改变。计算方法为：

```shell
$ openssl sha1 filePath/Podfile
```

#### COCOAPODS: 1.9.3

这个代表当前使用的CocoaPod版本号，远程版本管理应该要保证大家使用的pod版本号一致。

### Pods

#### Manifest.lock

Manifest.lock是Podfile.lock的副本，它是在Pods目录里面。它的作用是这样的，我们通常是不把Pods文件放到版本管理里面，而把Podfile.lock放到版本管理里面。这时对于拉取代码之后是否需要更新pod，就可以通过对比本地的Manifest.lock和远程Podfile.lock是否相同即可。

#### Targets Support Files

Pods安装的依赖是这样的组织形式

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201114143715.png)

一个Pods的Project下面有三个Targets，其中三个是安装的依赖库，最后一个Pods-FFDemo是关联三个库的Framework，也即是Pods这个Project的Targets。

##### Pods-Demo Framework

先看这个Demo的Framework，它会被用于工程项目的引用依赖

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201114145200.png)

这个库不会被打进包里，因为`Do Not Embed`代表并不是包含的关系。

这个工程下的配置文件有这些：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201114133927.png)

**许可协议文件**
两个以acknowledgements命名的文件是用于管理pod库的许可协议，即三方库必须带有的`LICENSE`文件，这也是为什么我们在制作pod时会要求我们指定软件协议。

**Framework文件**
这里还包含了用于管理Module的modulemap和umbrella.h文件。modulemap是对Module的声明文件，制作Framework我们总是需要该文件，它的内容如下：

```
framework module Pods_FFDemo {
  umbrella header "Pods-FFDemo-umbrella.h"

  export *
  module * { export * }
}
```

其指向了一个umbrella的头文件，这是制作Framework必须的头文件，modulemap和umbrella.h会在创建Module时自动生成，不建议手动修改其关系。

**dummy.m文件**

这其实是一个空的.m文件

```objective-c
#import <Foundation/Foundation.h>
@interface PodsDummy_Pods_FFDemo : NSObject
@end
@implementation PodsDummy_Pods_FFDemo
@end
```

那为什么要有这个东西呢，包括所有的三方库的包里也会包含一个dummy文件。我在[stackoverflow](https://stackoverflow.com/questions/39160655/why-do-cocoapod-create-a-dummy-class-for-every-pod "Why do cocoapod create a dummy class for every pod?")找到了一个解释：Xcode的编译是依赖.m文件的，如果一个库里没有.m文件，将不会被编译，为了防止这种情况就会在每个库里增加一个空的.m文件。

**xcconfig文件**

xcconfig文件是Build Setting配置项的文件形式，它的优先级大于Xcode内的Build Setting。看一个pod生成的debug模式下的xcconfig文件。

```shell
ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES = YES
FRAMEWORK_SEARCH_PATHS = $(inherited) "${PODS_CONFIGURATION_BUILD_DIR}/Alamofire" "${PODS_CONFIGURATION_BUILD_DIR}/MJRefresh" "${PODS_CONFIGURATION_BUILD_DIR}/Moya"
GCC_PREPROCESSOR_DEFINITIONS = $(inherited) COCOAPODS=1
HEADER_SEARCH_PATHS = $(inherited) "${PODS_CONFIGURATION_BUILD_DIR}/Alamofire/Alamofire.framework/Headers" "${PODS_CONFIGURATION_BUILD_DIR}/MJRefresh/MJRefresh.framework/Headers" "${PODS_CONFIGURATION_BUILD_DIR}/Moya/Moya.framework/Headers"
LD_RUNPATH_SEARCH_PATHS = $(inherited) '@executable_path/Frameworks' '@loader_path/Frameworks'
OTHER_LDFLAGS = $(inherited) -framework "Alamofire" -framework "CFNetwork" -framework "Foundation" -framework "MJRefresh" -framework "Moya"
OTHER_SWIFT_FLAGS = $(inherited) -D COCOAPODS
PODS_BUILD_DIR = ${BUILD_DIR}
PODS_CONFIGURATION_BUILD_DIR = ${PODS_BUILD_DIR}/$(CONFIGURATION)$(EFFECTIVE_PLATFORM_NAME)
PODS_PODFILE_DIR_PATH = ${SRCROOT}/.
PODS_ROOT = ${SRCROOT}/Pods
USE_RECURSIVE_SCRIPT_INPUTS_IN_SCRIPT_PHASES = YES
```

xcconfig还有个作用是设置参数，比如我们比较熟悉的`PODS_ROOT=${SRCROOT}/PODS`，它代表项目根目录下的PODS文件目录。另外两项用于帮助我们在项目中查找三方库的`FRAMEWORK_SEARCH_PATHS`和`HEADER_SEARCH_PATHS`也是在改文件内部定义的，这些配置会体现到Build Settings里面：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201114133440.png)

##### 三方库的Framework

![image-20201114150517801](https://cdn.jsdelivr.net/gh/zhangferry/Images/blog/image-20201114150517801.png)

各个三方库也都有一些配置文件，他们文件格式基本一致，上图是Moya的配置文件。Moya的xcconfig文件里有一行这个：

```shell
FRAMEWORK_SEARCH_PATHS = $(inherited) "${PODS_CONFIGURATION_BUILD_DIR}/Alamofire"
```

用于告诉Moya在引用Alamofire时应该去哪里找这个依赖。

## Build Phases

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201114132054.png)

这里是设置编译阶段配置的地方，当首次pod install成功之后，这里会多几个[CP]开头的配置项（CP即CocoaPods缩写），它们都是由CocoPods添加的脚本内容，执行顺序从上到下。

### New System Build

在讲编译脚本之前简单说下New Build System。

New Build System是Xcode10之后苹果推出的新的构建系统，新的构建系统对编译流程的[优化](https://nathanwong.co.uk/post/xcode-buildphases/ "Speeding up your custom Xcode build scripts")做了很多工作，虽然到Xcode12仍兼容旧版的Legacy Build System，但其已经被标记为移除，我们的项目和库都应该使用新版的构建系统进行构建。和新的构建系统随之而来的是在运行脚本时增加的输入输出列表。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201114130520.png)

这是为了控制是否每次编译都需要执行对应脚本，input和output文件可以是单个文件形式，如果文件过多可以放到格式为`xcfilelist`的文件列表里。

如果没有提供input和output，则每次构建都会运行该脚本。如果提供了，则会在以前从未运行过、某个输入文件被更改或某个输出文件丢失的情况下再次运行。

注意这些是构建脚本的默认逻辑，Xcode还提供了Run Scripts的自定义行为，默认勾选项：Based on dependency analysis，即代表上述逻辑。如果提供了输入输出还需要每次运行，关闭该选项即可。

### [CP] Check Pods Manifest.lock

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



### [CP] Copy Pods Resources

这个一般在以静态库引入的三方库切里面包含资源的话会添加该脚本，其作用是将三方库的资源文件拷贝至项目中。

它的完成是通过运行以下脚本进行的：

```shell
"${PODS_ROOT}/Target Support Files/Pods-FFDemo/Pods-FFDemo-resources.sh"
```

Pods-FFDemo-resources.sh文件在Pods目录内，该脚本内有个关键函数`install_resource`：

```shell
install_resource()
{
  if [[ "$1" = /* ]] ; then
    RESOURCE_PATH="$1"
  else
    RESOURCE_PATH="${PODS_ROOT}/$1"
  fi
  if [[ ! -e "$RESOURCE_PATH" ]] ; then
    cat << EOM
error: Resource "$RESOURCE_PATH" not found. Run 'pod install' to update the copy resources script.
EOM
    exit 1
  fi
  case $RESOURCE_PATH in
    *.storyboard)
      ibtool --reference-external-strings-file --errors --warnings --notices --minimum-deployment-target ${!DEPLOYMENT_TARGET_SETTING_NAME} --output-format human-readable-text --compile "${TARGET_BUILD_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}/`basename \"$RESOURCE_PATH\" .storyboard`.storyboardc" "$RESOURCE_PATH" --sdk "${SDKROOT}" ${TARGET_DEVICE_ARGS}
      ;;
    *.xib)
      ibtool --reference-external-strings-file --errors --warnings --notices --minimum-deployment-target ${!DEPLOYMENT_TARGET_SETTING_NAME} --output-format human-readable-text --compile "${TARGET_BUILD_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}/`basename \"$RESOURCE_PATH\" .xib`.nib" "$RESOURCE_PATH" --sdk "${SDKROOT}" ${TARGET_DEVICE_ARGS}
      ;;
    *.framework)
      echo "mkdir -p ${TARGET_BUILD_DIR}/${FRAMEWORKS_FOLDER_PATH}" || true
      mkdir -p "${TARGET_BUILD_DIR}/${FRAMEWORKS_FOLDER_PATH}"
      echo "rsync --delete -av "${RSYNC_PROTECT_TMP_FILES[@]}" $RESOURCE_PATH ${TARGET_BUILD_DIR}/${FRAMEWORKS_FOLDER_PATH}" || true
      rsync --delete -av "${RSYNC_PROTECT_TMP_FILES[@]}" "$RESOURCE_PATH" "${TARGET_BUILD_DIR}/${FRAMEWORKS_FOLDER_PATH}"
      ;;
    *.xcassets)
      ABSOLUTE_XCASSET_FILE="$RESOURCE_PATH"
      XCASSET_FILES+=("$ABSOLUTE_XCASSET_FILE")
      ;;
    *)
      echo "$RESOURCE_PATH" || true
      echo "$RESOURCE_PATH" >> "$RESOURCES_TO_COPY"
      ;;
  esac
}
```

删除了一部分日志内容，其内部主要是一个switch语句，根据资源文件的类型进行不同的同步操作。这里重点说下几种重要格式文件的处理方式。

**storyboard和xib格式**

这两项资源文件是需要编译处理的，利用ibtool命令分别转成sotryboardc和nib格式。

**xcassets格式**

这里的图片最终会被打包到Assets.car供程序使用，需要使用actool。

**Bundle、plist、png等资源**

其他类的资源是会走到switch语句最后出口，进行资源路径赋值给`$RESOURCES_TO_COPY`，在后面的代码中通过`rsync`命令，将资源同步到构建包的目录。

该脚本会打印很多日志，在使用CocoaPods时如果遇到资源相关的问题都可以遵循错误日志来这里推测定位错误原因。

### [CP] Embed Pods Frameworks

该处脚本是直接运行`Pods-FFDemo-frameworks.sh`。

```shell
"${PODS_ROOT}/Target Support Files/Pods-FFDemo/Pods-FFDemo-frameworks.sh"
```

可能你还记得上面说的pod会把多个库的依赖做成一个合并的库，但该库是以依赖的形式引入主工程，但是程序的运行时需要这些库，我们打包时就需要将各个库Embed到项目里，而做这个工作的就是该脚本。

```shell
# Copies and strips a vendored framework
install_framework()
{
  rsync --delete -av "${RSYNC_PROTECT_TMP_FILES[@]}" --links --filter "- CVS/" --filter "- .svn/" --filter "- .git/" --filter "- .hg/" --filter "- Headers" --filter "- PrivateHeaders" --filter "- Modules" "${source}" "${destination}"

  # other code...

  # Strip invalid architectures so "fat" simulator / device frameworks work on device
  if [[ "$(file "$binary")" == *"dynamically linked shared library"* ]]; then
    strip_invalid_archs "$binary"
  fi

  # Resign the code if required by the build settings to avoid unstable apps
  code_sign_if_enabled "${destination}/$(basename "$1")"
}
```

脚本内容主要是调用`install_framework`函数，将framework内容同步到构建包里。在该函数里还有几个关键方法，`strip_invalid_archs`用于去除无用架构，`code_sign_if_enabled`用于framwork签名。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/wechat_official.png)