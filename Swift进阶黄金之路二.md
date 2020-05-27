上期遗留一个问题：为什么 `rethrows` 一般用在参数中含有可以 `throws` 的方法的高阶函数中。

在Swift的官方文档中对`rethrows`有以下说明：

> A function or method can be declared with the `rethrows` keyword to indicate that it throws an error only if one of its function parameters throws an error. These functions and methods are known as *rethrowing functions* and *rethrowing methods*. Rethrowing functions and methods must have at least one throwing function parameter.

可以说rethrows这个关键词就是针对带throws的参数而创建的，因为它要求至少有一个参数是可throw异常的才可以用它。

https://docs.swift.org/swift-book/ReferenceManual/Attributes.html



这一期会介绍一些新的关键词，他们出现在一些三方库或者系统库里，了解他们我们能够体会swift的魅力。

## Attributes

### available

`@available`： 可用来标识计算属性、函数、类、协议、结构体、枚举等类型的生命周期。（依赖于特定的平台版本 或 Swift 版本）。它的后面一般跟至少两个参数，参数之间以逗号隔开。其中第一个参数是固定的，代表着平台和语言，可选值有以下这几个：

- `iOS`
- `iOSApplicationExtension`
- `macOS`
- `macOSApplicationExtension`
- `watchOS`
- `watchOSApplicationExtension`
- `tvOS`
- `tvOSApplicationExtension`
- `swift`

可以使用`*`指代支持所有这些平台。

有一个我们常用的例子，当需要关闭scrollView的自动调整inset功能时：

```swift
// 指定该方法仅在iOS11及以上的系统设置
if #available(iOS 11.0, *) {
  scrollView.contentInsetAdjustmentBehavior = .never
} else {
  automaticallyAdjustsScrollViewInsets = false
}
```

还有一种用法是放在函数、结构体、枚举、类或者协议的前面，表示当前类型仅适用于某一平台：

```swift
@available(iOS 12.0, *)
func adjustDarkMode() {
  /* code */
}
@available(iOS 12.0, *)
struct DarkModeConfig {
  /* code */
}
@available(iOS 12.0, *)
protocol DarkModeTheme {
	/* code */
}
```

 版本和平台的限定可以写多个：
```swift
@available(OSX 10.15, iOS 13, tvOS 13, watchOS 6, *)
public func applying(_ difference: CollectionDifference<Element>) -> ArraySlice<Element>?
```



**注意：作为条件语句的`available`前面是`#`，作为标记位时是`@`**

刚才说了，available后面参数至少要有两个，后面的可选参数这些：

* `deprecated`：从指定平台标记为过期，可以指定版本号

- `obsoleted=版本号`：从指定平台某个版本开始废弃（注意弃用的区别，`deprecated`是还可以继续使用，只不过是不推荐了，`obsoleted`是调用就会编译错误）该声明
- `message=信息内容`：给出一些附加信息
- `unavailable`：指定平台上是无效的
- `renamed=新名字`：重命名声明



我们看几个例子，这个是Array里`flatMap`的函数说明：

```swift
@available(swift, deprecated: 4.1, renamed: "compactMap(_:)", message: "Please use compactMap(_:) for the case where closure returns an optional value")
public func flatMap<ElementOfResult>(_ transform: (Element) throws -> ElementOfResult?) rethrows -> [ElementOfResult]
```

它的含义是针对swift语言，该方式在swift4.1版本之后标记为过期，对应该函数的新名字为`compactMap(_:)`，如果我们在4.1之上的版本使用该函数会收到编译器的警告，即`⚠️Please use compactMap(_:) for the case where closure returns an optional value`。



在Realm库里，有一个销毁NotificationToken的方法，被标记为`unavailable`：

```swift
extension RLMNotificationToken {
    @available(*, unavailable, renamed: "invalidate()")
    @nonobjc public func stop() { fatalError() }
}
```

标记为`unavailable`就不会被编译器联想到。这个主要是为升级用户的迁移做准备，从可用`stop()`的版本升上了，会红色报错，提示该方法不可用。因为有`renamed`，编译器会推荐你用`invalidate()`，点击`fix`就直接切换了。所以这两个标记参数常一起出现。



### @discardableResult

带返回的函数如果没有处理返回值会被编译器警告⚠️。但有时我们就是不需要返回值的，这个时候我们可以让编译器忽略警告，就是在方法名前用`@discardableResult`声明一下。可以参考Alamofire中`request`的写法：

```swift
@discardableResult
public func request(
    _ url: URLConvertible,
    method: HTTPMethod = .get,
    parameters: Parameters? = nil,
    encoding: ParameterEncoding = URLEncoding.default,
    headers: HTTPHeaders? = nil)
    -> DataRequest
{
    return SessionManager.default.request(
        url,
        method: method,
        parameters: parameters,
        encoding: encoding,
        headers: headers
    )
}
```



### @inlinable



###  @warn_unqualified_access

```swift
extension IndexingIterator where Self.Element : Comparable {
  @warn_unqualified_access
	@inlinable public func min() -> Elements.Element?
  
  @warn_unqualified_access
   @inlinable public func max() -> Elements.Element?
}

```



###  @objc @nonobjc



### @NSApplicationMain



### @objcMembers



### @testable



### @unknown



## 声明修饰符

### dynamic



### final static



### lazy



### unowned weak



编译器block



https://docs.swift.org/swift-book/ReferenceManual/Statements.html



https://docs.swift.org/swift-book/ReferenceManual/Expressions.html

文字表达式

|   Literal    |        Type        |                            Value                             |
| :----------: | :----------------: | :----------------------------------------------------------: |
|   `#file`    |      `String`      |          The name of the file in which it appears.           |
|   `#line`    |       `Int`        |             The line number on which it appears.             |
|  `#column`   |       `Int`        |            The column number in which it begins.             |
| `#function`  |      `String`      |       The name of the declaration in which it appears.       |
| `#dsohandle` | `UnsafeRawPointer` | The DSO (dynamic shared object) handle in use where it appears. |

print函数定义





Key-Path 表达式

