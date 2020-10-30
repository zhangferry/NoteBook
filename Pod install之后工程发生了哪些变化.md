# Pod install之后工程发生了哪些变化

未引入pod的项目是这样的



我们执行`pod init`创建一个Podfile模板，然后在Podfile里添加：

```
target 'FFDemo' do
  # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!

  # Pods for FFDemo
  pod 'MJRefresh', '~> 3.5.0'
  pod 'Moya'

end
```

意为需要导入MJRefresh这个库。选这个库的目的是因为它包含资源文件。

执行`pod install`之后项目变成了这样：

```

```

从外部目录看其多了xcworkspace、Podfile.lock、Pods等内容。

## xcworkspace

xcworkspace是一个父文件件，里面包含一个文件叫`contents.xcworkspacedata`，它的内容是这样的

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

一个xml格式的组织形式，在该文件内部，由<workspace>标签维护多个xcodeproj。这里除了有我们创建的FFDemo.xcodeproj还有一个新建的Pods.xcodeproj。



## Podfile.lock

lock文件的形式是这样的：

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

### PODS

上面的`PODS`是指当前引用库的版本号，上面Moya出现了一个`Moya/Core`是因为Moya里包含了一个子spec叫`Core`，这里的定义由[Moya.podspec](https://github.com/Moya/Moya/blob/master/Moya.podspec)决定。

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

DEPENDENCIES为pod库的描述信息，这里是同Podfile里的写法的，如果我们指定了版本号，那这里也会有对应版本号`MJRefresh (~> 3.5.0)`，如果没有指定就不会显示版本号`Moya`。



### SPEC REPOS

这里描述的是仓库信息，安装了哪些库，是在哪个仓库安装的。

trunk代表共有仓库的名称，所有的公共pod都会安装于这个仓库。如果有私有仓库，这里还会列出私有库及其下安装的pod库。



### SPEC CHECKSUM

Spec的校验和是针对每个库都有一个，它对应的是对podspec文件路径求sha1，比如MJRefresh的校验和：`6afc955813966afb08305477dd7a0d9ad5e79a16`。安装MJRefresh的版本为3.5.0，它的本地podspec文件路径为：`~/.cocoapods/repos/trunk/Specs/0/f/b/MJRefresh/3.5.0/MJRefresh.podspec.json`。这个路径可以通过`pod install --verbose`查看。我们可以发现原本配置的podspec格式编程了podspec.json格式，但这个json内容其实跟podspec是一样的，只不过换了一种展示格式。
校验和的生成是这样的：

```shell
$ pod ipc spec ~/.cocoapods/repos/trunk/Specs/0/f/b/MJRefresh/3.5.0/MJRefresh.podspec.json | openssl sha1
```

该命令是对json内容求sha1，所以如果改json内容有任何不同都会导致校验和不同，在不同团队直接合作的话如果出现这里的不同，需要检查对应的json内容。

### PODFILE CHECKSUM

这个校验和是针对podfile内容的校验和，如果podfile内容改变了，该值也会跟着改变。

### COCOAPODS: 1.9.3

这个代表当前使用的pod版本号，当使用不同版本



## Pods

### Manifest.lock

Manifest.lock是Podfile.lock的副本，它并没有放到跟目录而是放到了Pods目录里面。它的作用是这样的，Pods我们通常是不放到版本管理里面的，而Podfile.lock通常放到版本管理里面，这时，对于拉取代码之后是否有人修改了Podfile.lock，我们就可以比较Podfile.lock和manifest.lock进行区分。



### Build Phases

多了[CP] Check Pods Manifest.lock脚本，该脚本内容如下

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

这里的脚本会根据排列顺序，依次执行，该脚本执行内容为比较Podfile.lock和Manifest.lock文件是否相同，如果不同就输出错误信息：`error: The sandbox is not in sync with the Podfile.lock. Run 'pod install' or update your CocoaPods installation.`，并退出脚本，这会导致后续项目报错，无法继续编译。

如果编译成功就将SUCCESS赋值给变量`SCRIPT_OUTPUT_FILE_0`。





[CP] Embed Pods Frameworks

该脚本是直接运行Pods-FFDemo-frameworks.sh脚本。



