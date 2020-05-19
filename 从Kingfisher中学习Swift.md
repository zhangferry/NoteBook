kingfisher是一个非常优秀的用Swift语言写的图片处理框架，而喵神本人也是国内Swift的推广者。写过多本Swift相关书籍。所以喵神主导的这个Kingfisher对我们了解Swift会有很大帮助。这篇文章视不是讲Kingfisher的使用技巧和单纯源码分析的，而是尝试谈一下Kingfisher背后的设计思路，以帮助大家了解Swift。

当前Kingfisher版本：5.14.0

![image-20200515171446916](/Users/zhangferry/Library/Application Support/typora-user-images/image-20200515171446916.png)



### 多平台兼容

通常一个受欢迎的框架往往会考虑多平台使用，iOS，macOS，watchOS等。这三个平台不同控件的定义不同，如何实现api共享呢，在OC平台你需要通过宏定义去区分和判定：

```objective-c
#if TARGET_OS_OSX
    #define SD_MAC 1
#else
    #define SD_MAC 0
#endif

#if SD_MAC
    #import <AppKit/AppKit.h>
    #ifndef UIImage
        #define UIImage NSImage
    #endif
    #ifndef UIImageView
        #define UIImageView NSImageView
    #endif
    #ifndef UIView
        #define UIView NSView
    #endif
    #ifndef UIColor
        #define UIColor NSColor
    #endif
#else
    #if SD_UIKIT
        #import <UIKit/UIKit.h>
    #endif
    #if SD_WATCH
        #import <WatchKit/WatchKit.h>
        #ifndef UIView
            #define UIView WKInterfaceObject
        #endif
        #ifndef UIImageView
            #define UIImageView WKInterfaceImage
        #endif
    #endif
#endif
```

SDWebImage的做法是将各个平台的命名都使用iOS系统的命名方式，比如UIImageView，它在iOS系统上就是UIImageView，在MacOS上它是NSImageVIew，在watchOS上就是WKInterfaceImage。这样很不直观。

换到Swift里面上面的哪些代码就可以这么去写：

```swift
#if os(macOS)
import AppKit
public typealias KFCrossPlatformImage = NSImage
public typealias KFCrossPlatformView = NSView
public typealias KFCrossPlatformColor = NSColor
public typealias KFCrossPlatformImageView = NSImageView
public typealias KFCrossPlatformButton = NSButton
#else
import UIKit
public typealias KFCrossPlatformImage = UIImage
public typealias KFCrossPlatformColor = UIColor
#if !os(watchOS)
public typealias KFCrossPlatformImageView = UIImageView
public typealias KFCrossPlatformView = UIView
public typealias KFCrossPlatformButton = UIButton
#else
import WatchKit
#endif
#endif
```



### 命名空间

在OC中没有命名空间，我们对ImageView等系统框架扩展方法时往往都是通过下划线进行的，像这样：

```swift
imageView.sd_setImage(with: URL(string: "http://www.domain.com/path/to/image.jpg"), placeholderImage: UIImage(named: "placeholder.png"))
```



如果使用Kingfisher对一个视图展示网络图片，可以这样：

```swift
let url = URL(string: "https://example.com/image.png")
imageView.kf.setImage(with: url)
```

本身，UIImageView是没有.kf的属性的，那如何才能获取使我们的UIImageView获取`kf`这个命名空间呢？

```swift
// 定义一个包装器
public struct KingfisherWrapper<Base> {
    public let base: Base
    public init(_ base: Base) {
        self.base = base
    }
}
// 定义一个协议
public protocol KingfisherCompatible: AnyObject { }
public protocol KingfisherCompatibleValue {}

extension KingfisherCompatible {
    /// Gets a namespace holder for Kingfisher compatible types.
    public var kf: KingfisherWrapper<Self> {
        get { return KingfisherWrapper(self) }
        set { }
    }
}

extension KingfisherCompatibleValue {
    /// Gets a namespace holder for Kingfisher compatible types.
    public var kf: KingfisherWrapper<Self> {
        get { return KingfisherWrapper(self) }
        set { }
    }
}
// 让UI视图实现该协议
extension KFCrossPlatformImage: KingfisherCompatible { }
extension KFCrossPlatformImageView: KingfisherCompatible { }
extension KFCrossPlatformButton: KingfisherCompatible { }
```



### Result



