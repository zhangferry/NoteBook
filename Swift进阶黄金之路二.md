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

这个关键词就是可内联的声明，它来源于C语言中的`inline`。C中一般用于函数前，做内联函数，它的目的是防止当某一函数多次调用造成函数栈溢出的情况。因为内联函数不会一遍又一遍的生成函数栈，而是在编译的时候将该函数用具体实现代替。

内联函数常出现在系统库中，OC的runtim中就有大量的`inline`使用：

```c
static inline id autorelease(id obj)
{
    ASSERT(obj);
    ASSERT(!obj->isTaggedPointer());
    id *dest __unused = autoreleaseFast(obj);
    ASSERT(!dest  ||  dest == EMPTY_POOL_PLACEHOLDER  ||  *dest == obj);
    return obj;
}
```

Swift中的`@inlinable`和C中的inline基本相同，它也在标准库的定义中广泛出现，可用于方法，计算属性，下标，便利构造方法或者deinit方法中。

例如Swift对Array中map函数的定义：

```swift
@inlinable public func map<T>(_ transform: (Element) throws -> T) rethrows -> [T]
```

其实Array中声明的大部分函数前面都加了`@inlinable`，它的意思是当应用某一处调用该方法时，编译器会将调用处进行展开，`map`函数的实现直接挪过来。

**需要注意内联声明不能用于标记为`private`或者`fileprivate`的地方。**

内联的好处是运行时更快，因为它省略了从标准库调用`map`实现的步骤。但这也是有弊端的，因为由编译器做展开，势必增加了编译的开销，所以当进行编译时间优化时就会发现，那些系统高阶函数的链式调用往往是最耗时的。

###  @warn_unqualified_access

给顶级函数、实例方法或者类和静态方法应用这个特性来在函数或者方法不带前置修饰使用时触发警告，比如模块名、类型名或者实例变量和常量。使用这个特性来降低同一生效范围内相同函数名造成的歧义。

比如说，Swift 标准库包含了顶级函数[min(_:_:)](https://developer.apple.com/documentation/swift/1538339-min/)和包含可比元素的序列的方法[min()](https://developer.apple.com/documentation/swift/sequence/1641174-min)。序列方法使用 warn_unqualified_access 特性声明以便于在 Sequence 扩展中避免同时使用两者出现困惑。

```swift
@warn_unqualified_access
@inlinable public func min() -> Element?

@warn_unqualified_access
@inlinable public func max() -> Element?
```



###  @objc

把这个特性用到任何可以在 Objective-C 中表示的声明上——例如，非内嵌类，协议，非泛型枚举（原始值类型只能是整数），类和协议的属性、方法（包括 setter 和 getter ），初始化器，反初始化器，下标。 objc 特性告诉编译器，这个声明在 Objective-C 代码中是可用的。



用 objc 特性标记的类必须继承自一个 Objective-C 中定义的类。如果你把 objc 用到类或协议中，它会隐式地应用于该类或协议中 Objective-C 兼容的成员上。如果一个类继承自另一个带 objc 特性标记或 Objective-C 中定义的类，编译器也会隐式地给这个类添加 objc 特性。标记为 objc 特性的协议不能继承自非 objc 特性的协议。

@objc还有一个用处是当你想在OC的代码中暴露一个不同的名字时，可以用这个特性，它可以用于类，函数，枚举，枚举成员，协议，getter，setter等。



```swift
// 当在OC代码中访问enabled的getter方法时，是通过isEnabled
class ExampleClass: NSObject {
    @objc var enabled: Bool {
        @objc(isEnabled) get {
            // Return the appropriate value
        }
    }
}
```

这一特性还可以用于解决潜在的命名冲突问题，因为Swift有命名空间，常常不带前缀声明，而OC没有命名空间是需要带的，当在OC代码中引用Swift库，为了防止潜在的命名冲突，可以选择一个带前缀的名字供OC代码使用。

[Charts](https://github.com/danielgindi/Charts)作为一个在OC和Swift中都很常用的图标库，是需要较好的同时兼容两种语言的使用的，所以也可以看到里面有大量通过`@objc`标记对OC调用时的重命名代码：

```swift
@objc(ChartAnimator)
open class Animator: NSObject { }

@objc(ChartComponentBase)
open class ComponentBase: NSObject { }
```

### @objcMembers

因为Swift中定义的方法默认是不能被OC调用的，除非我们手动添加@objc标识。但如果一个类的方法属性较多，这样会很麻烦，于是有了这样一个标识符`@objcMembers`，它可以让整个类的属性方法都隐式添加`@objc`，不光如此对于类的子类、扩展、子类的扩展都也隐式的添加@objc，当然对于OC不支持的类型，仍然无法被OC调用：

```swift
@objcMembers
class MyClass : NSObject {
  func foo() { }             // implicitly @objc

  func bar() -> (Int, Int)   // not @objc, because tuple returns
      // aren't representable in Objective-C
}

extension MyClass {
  func baz() { }   // implicitly @objc
}

class MySubClass : MyClass {
  func wibble() { }   // implicitly @objc
}

extension MySubClass {
  func wobble() { }   // implicitly @objc
}
```

参考：[Swift3、4中的@objc、@objcMembers和dynamic](https://juejin.im/post/5cb6dccde51d456e5a0728db)

### @testable

`@testable`是用于测试模块访问主target的一个关键词。

因为测试模块和主工程是两个不同的target，在swift中，每个target代表着不同的module，不同module之间访问代码需要public和open级别的关键词支撑。但是主工程并不是对外模块，为了测试修改访问权限是不应该的，所以有了`@testable`关键词。使用如下：

```swift
import XCTest
@testable import Project

class ProjectTests: XCTestCase {
  /* code */
}
```

这时测试模块就可以访问那些标记为internal或者public级别的类和成员了。

## 声明修饰符

### final static



### lazy

lazy是懒加载的关键词，当我们仅需要在使用时进行初始化操作就可以选用该关键词。举个例子：

```swift
class Avatar {
  lazy var smallImage: UIImage = self.largeImage.resizedTo(Avatar.defaultSmallSize)
  var largeImage: UIImage

  init(largeImage: UIImage) {
    self.largeImage = largeImage
  }
}
```

对于smallImage，我们声明了lazy，如果我们不去调用它是不会走后面的图片缩放计算的。但是如果没有lazy，因为是初始化方法，它会直接计算出smallImage的值。所以lazy很好的避免的不必要的计算。

另一个常用lazy的地方是对于UI属性的定义：

```swift
lazy var dayLabel: UILabel = {
    let label = UILabel()
  	label.text = self.todayText()
    return label
}()
```

这里使用的是一个闭包，当调用该属性时，执行闭包里面的内容，返回具体的label，完成初始化。

使用lazy你可能会发现它只能通过var初始而不能通过let，这是由 `lazy` 的具体实现细节决定的：它在没有值的情况下以某种方式被初始化，然后在被访问时改变自己的值，这就要求该属性是可变的。

另外我们可以在Sequences中使用lazy，在讲解它之前我们先看一个例子：

```swift
func increment(x: Int) -> Int {
  print("Computing next value of \(x)")
  return x+1
}

let array = Array(0..<1000)
let incArray = array.map(increment)
print("Result:")
print(incArray[0], incArray[4])
```

在执行`print("Result:")`之前，`Computing next value of ...`会被执行1000次，但实际上我们只需要0和4这两个index对应的值。

上面说了序列也可以使用lazy，使用的方式是：

```swift
let array = Array(0..<1000)
let incArray = array.lazy.map(increment)
print("Result:")
print(incArray[0], incArray[4])

// Result:
// 1 5
```

在执行`print("Result:")`之前，并不会打印任何东西，只打印了我们用到的1和5。就是说这里的lazy可以延迟到我们取值时才去计算map里的结果。

我们看下这个lazy的定义：

```swift
@inlinable public var lazy: LazySequence<Array<Element>> { get }
```

它返回一个`LazySequence`的结构体，这个结构体里面包含了`Array<Element>`，而`map`的计算在`LazySequence`里又重新定义了一下：

```swift
/// Returns a `LazyMapSequence` over this `Sequence`.  The elements of
/// the result are computed lazily, each time they are read, by
/// calling `transform` function on a base element.
@inlinable public func map<U>(_ transform: @escaping (Base.Element) -> U) -> LazyMapSequence<Base, U>
```

这里完成了lazy序列的实现。`LazySequence`类型的lazy只能被用于map、flatMap、compactMap这样的高阶函数中。

**参考**： [“懒”点儿好](https://swift.gg/2016/03/25/being-lazy/)

**纠错**：参考文章中说："这些类型（LazySequence）只能被用在 `map`，`flatMap`，`filter`这样的高阶函数中" 其实是没有filter的，因为filter是过滤函数，它需要完整遍历一遍序列才能完成过滤操作，是无法懒加载的，而且我查了`LazySequence`的定义，确实是没有`filter`函数的。

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



SwiftUI中的关键词