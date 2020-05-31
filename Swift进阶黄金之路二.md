![image-20200511230812677](https://cdn.jsdelivr.net/gh/zhangferry/Images/blog/image-20200511230812677.png)

上期遗留一个问题：为什么 `rethrows` 一般用在参数中含有可以 `throws` 的方法的高阶函数中。

在Swift的官方文档中对`rethrows`有以下说明：

> A function or method can be declared with the `rethrows` keyword to indicate that it throws an error only if one of its function parameters throws an error. These functions and methods are known as *rethrowing functions* and *rethrowing methods*. Rethrowing functions and methods must have at least one throwing function parameter.

返回`rethrows`的函数要求至少有一个可抛出异常的函数式参数，而有以函数作为参数的函数就叫做高阶函数。

## 特性修饰词

在Swift语法中有很多`@`符号，这些`@`符号在Swift4之前的版本大多是兼容OC的特性，Swift4及之后则出现越来越多搭配`@`符号的新特性。以`@`开头的修饰词，在官网中叫`Attributes`，在SwiftGG的翻译中叫[特性](https://www.cnswift.org/attributes)，我感觉这一类以`@`开头修饰的词叫`特性修饰词`比较合理一点。从Swift5的发布来看（`@dynamicCallable`,`@State`），之后将会有更多的特性修饰词出现，在他们出来之前，我们有必要先了解下现有的一些特性修饰词以及它们的作用。

参考：[Swift Attributes](https://docs.swift.org/swift-book/ReferenceManual/Attributes.html)

### @available

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

这个关键词用的较少，通过其命名我们可以推断出其大概含义：对“不合规”的访问进行警告。这是为了解决对于相同名称的函数，不同访问对象可能产生歧义的问题。

比如说，Swift 标准库包含了`Array`的`min()`和包含`Sequence`的`min()`。在他们之前加上 `@warn_unqualified_access`特性声明以便在调用时告诉使用者加上限定对象。

```swift
extension Array where Self.Element : Comparable {
  @warn_unqualified_access
	@inlinable public func min() -> Element?
}
extension Sequence where Self.Element : Comparable {
  @warn_unqualified_access
  @inlinable public func min() -> Self.Element?
}
```

这里有一个场景可以便于理解它的含义，我们自定义一个求`Array`中最小值的函数：

```swift
extension Array where Element: Comparable {
    func minValue() -> Element? {
        return min()
    }
}
```

我们会收到编译器的警告：`Use of 'min' treated as a reference to instance method in protocol 'Sequence', Use 'self.' to silence this warning`。它告诉我们编译器推断我们当前使用的是Sequence中的`min()`，这与我们的想法是违背的。因为有这个`@warn_unqualified_access`限定，我们能及时的发现问题，并解决问题：`self.min()`。

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

###@frozen 和@unknown default

frozen意为冻结，是为Swift5的ABI稳定准备的一个字段，意味向编译器保证之后不会做出改变。为什么需要这么做以及这么做有什么好处，他们和ABI稳定是息息相关的，内容有点多就不放这里了，之后会单独出一篇文章介绍，这里只介绍这两个字段的含义。

```swift
@frozen public enum ComparisonResult : Int {
    case orderedAscending = -1
    case orderedSame = 0
    case orderedDescending = 1
}

@frozen public struct String {}

extension AVPlayerItem {
  	public enum Status : Int {
        case unknown = 0
        case readyToPlay = 1
        case failed = 2
    }
}
```

`ComparisonResult`这个枚举值被标记为`@frozen`即使保证之后该枚举值不会再变。注意到`String`作为结构体也被标记为`@frozen`，意为String结构体的属性及属性顺序将不再变化。其实我们常用的类型像`Int`、`Float`、`Array`、`Dictionary`、`Set`等都已被“冻结”。需要说明的是冻结仅针对`struct`和`enum`这种值类型，因为他们在编译器就确定好了内存布局。对于class类型，不存在是否冻结的概念，可以想下为什么。

对于没有标记为frozen的枚举`AVPlayerItem.Status`，则认为该枚举值在之后的系统版本中可能变化。

对于可能变化的枚举，我们在列出所有case的时候还需要加上对`@unknown default`的判断，这一步会有编译器检查：

```swift
switch currentItem.status {
    case .readyToPlay:
        /* code */
    case .failed:
        /* code */
    case .unknown:
        /* code */
    @unknown default:
        fatalError("not supported")
}
```

### @State、@Binding、@ObservedObject、@EnvironmentObject

这几个是SwiftUI中出现的特性修饰词，因为我对SwiftUI的了解不多，这里就不做解释了。附一篇文章供大家了解。

[[译]理解 SwiftUI 里的属性装饰器@State, @Binding, @ObservedObject, @EnvironmentObject](https://juejin.im/post/5d625c01f265da03cd0a8a58)

## Swift中的一些特性关键词

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

Swift开发过程中我们会经常跟闭包打交道，而用到闭包就不可避免的遇到循环引用问题。在Swift处理循环引用可以使用`unowned`和`weak`这两个关键词。看下面两个例子：

```swift
class Dog {
    var name: String
    init (name: String ) {
        self.name = name
    }
    deinit {
        print("\(name) is deinitialized")
    }
}

class Bone {
  	// weak 修饰词
    weak var owner: Dog?
    init(owner: Dog?) {
        self.owner = owner
    }
    deinit {
        print("bone is deinitialized" )
    }
}

var lucky: Dog? = Dog(name: "Lucky")
var bone: Bone? = Bone(owner: lucky!)
lucky =  nil
// Lucky is deinitialized
```

这里Dog和Bone是相互引用的关系，如果没有`weak var owner: Dog?`这里的weak声明，将不会打印`Lucky is deinitialized`。还有一种解决循环应用的方式是把`weak`替换为`unowned`关键词。

* weak相当于oc里面的weak，弱引用，不会增加循环计数。主体对象释放时被weak修饰的属性也会被释放，所以weak修饰对象就是optional。
* unowned相当于oc里面的`unsafe_unretained`，它不会增加引用计数，即使它的引用对象释放了，它仍然会保持对被已经释放了的对象的一个 "无效的" 引用，它不能是 Optional 值，也不会被指向 `nil`。如果此时为无效引用，再去尝试访问它就会crash。

这两者还有一个更常用的地方是在闭包里面：

```swift
lazy var someClosure: () -> Void = { [weak self] in
    // 被weak修饰后self为optional，这里是判断self非空的操作                                
    guard let self = self else { retrun }
    self.doSomethings()
}
```

这里如果是`unowned`修饰self的话，就不需要用guard做解包操作了。但是我们不能为了省略解包的操作就用`unowned`，也不能为了安全起见全部`weak`，弄清楚两者的适用场景非常重要。

根据苹果的建议：

> Define a capture in a closure as an unowned reference when the closure and the instance it captures will always refer to each other, and will always be deallocated at the same time.

当闭包和它捕获的实例总是相互引用，并且总是同时释放时，即相同的生命周期，我们应该用unowned，除此之外的场景就用weak。

![img](https://www.uraimo.com/imgs/unownedbig.png)

参考：[内存管理，WEAK 和 UNOWNED](https://swifter.tips/retain-cycle/)

[Unowned 还是 Weak？生命周期和性能对比](https://swift.gg/2017/05/16/unowned-or-weak-lifetime-and-performance/)

### KeyPath

KeyPath是键值路径，最开始是用于处理KVC和KVO问题，后来又做了更广泛的扩展。

```swift
// KVC问题，支持struct、class
struct User {
    let name: String
    var age: Int
}

var user1 = User()
user1.name = "ferry"
user1.age = 18
 
//使用KVC取值
let path: KeyPath = \User.name
user1[keyPath: path] = "zhang"
let name = user1[keyPath: path]
print(name) //zhang

// KVO的实现还是仅限于继承自NSObject的类型
// playItem为AVPlayerItem对象
playItem.observe(\.status, changeHandler: { (_, change) in
    /* code */    
})
```

这个KeyPath的定义是这样的：

```swift
public class AnyKeyPath : Hashable, _AppendKeyPath {}

/// A partially type-erased key path, from a concrete root type to any
/// resulting value type.
public class PartialKeyPath<Root> : AnyKeyPath {}

/// A key path from a specific root type to a specific resulting value type.
public class KeyPath<Root, Value> : PartialKeyPath<Root> {}
```

定义一个`KeyPath`需要指定两个类型，根类型和对应的结果类型。对应上面示例中的path：

```swift
let path: KeyPath<User, String> = \User.name
```

根类型就是User，结果类型就是String。也可以不指定，因为编译器可以从`\User.name`推断出来。那为什么叫根类型的？可以注意到KeyPath遵循一个协议`_AppendKeyPath`，它里面定义了很多`append`的方法，KeyPath是多层可以追加的，就是如果属性是自定义的Address类型，形如：

```swift
struct Address {
    var country: String = ""
}
let path: KeyPath<User, String> = \User.address.country
```

这里根类型为`User`，次级类型是`Address`，结果类型是`String`。所以`path`的类型依然是`KeyPath<User, String>`。

明白了这些我们可以用KeyPath做一些扩展：

```swift
extension Sequence {
    func sorted<T: Comparable>(by keyPath: KeyPath<Element, T>) -> [Element] {
        return sorted { a, b in
            return a[keyPath: keyPath] < b[keyPath: keyPath]
        }
    }
}
// users is Array<User>
let newUsers = users.sorted(by: \.age)
```

这个自定义`sorted`函数实现了通过传入keyPath进行升序排列功能。

参考：[The power of key paths in Swift](https://www.swiftbysundell.com/articles/the-power-of-key-paths-in-swift/)

### some

`some`是Swift5.1新增的特性。它的用法就是修饰在一个 protocol 前面，默认场景下 protocol 是没有具体类型信息的，但是用 `some` 修饰后，编译器会让 protocol 的实例类型对外透明。

可以通过一个例子理解这段话的含义，当我们尝试定义一个遵循`Equatable`协议的value时：

```swift
// Protocol 'Equatable' can only be used as a generic constraint because it has Self or associated type requirements
var value: Equatable {
    return 1
}

var value: Int {
    return 1
}
```

编译器提示我们`Equatable`只能被用来做泛型的约束，它不是一个具体的类型，这里我们需要使用一个遵循`Equatable`的具体类型（Int）进行定义。但有时我们并不想指定具体的类型，这时就可以在协议名前加上`some`，让编译器自己去推断value的类型：

```swift
var value: some Equatable {
    return 1
}
```

在SwiftUI里some随处可见：

```swift
struct ContentView: View {
    var body: some View {
        Text("Hello World")
    }
}
```

这里使用`some`就是因为`View`是一个协议，而不是具体类型。

当我们尝试欺骗编译器，每次随机返回不同的`Equatable`类型：

```swift
var value: some Equatable {
    if Bool.random() {
        return 1
    } else {
        return "1"
    }
}
```

聪明的编译器是会发现的，并警告我们`Function declares an opaque return type, but the return statements in its body do not have matching underlying types`。

参考：[SwiftUI 的一些初步探索 (一)](https://onevcat.com/2019/06/swift-ui-firstlook/)