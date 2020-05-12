![image-20200511230812677](https://cdn.jsdelivr.net/gh/zhangferry/Images/blog/image-20200511230812677.png)

这篇是对[一文鉴定是Swift的王者，还是青铜](https://juejin.im/post/5e96f898e51d4546c27bcf81)文章中问题的解答。这些问题仅仅是表层概念，属于知识点，在我看来即使都很清楚也并不能代表上了王者，如果非要用段位类比的话，黄金还是合理的😄。

Swift是一门上手容易，但是精通较难的语言。即使下面这些内容都不清楚也不妨碍你开发业务需求，但是了解之后它能够帮助我们写出更加Swifty的代码。

## 一、 协议 Protocol

### ExpressibleByDictionaryLiteral

`ExpressibleByDictionaryLiteral`是字典的字面量协议，该协议的完整写法为：

```swift
public protocol ExpressibleByDictionaryLiteral {
    /// The key type of a dictionary literal.
    associatedtype Key
    /// The value type of a dictionary literal.
    associatedtype Value
    /// Creates an instance initialized with the given key-value pairs.
    init(dictionaryLiteral elements: (Self.Key, Self.Value)...)
}
```

首先字面量（Literal）的意思是：**用于表达源代码中一个固定值的表示法（notation）**。

举个例子，构造字典我们可以通过以下两种方式进行：

```swift
// 方法一：
var countryCodes = Dictionary<String, Any>()
countryCodes["BR"] = "Brazil"
countryCodes["GH"] = "Ghana"
// 方法二：
let countryCodes = ["BR": "Brazil", "GH": "Ghana"]
```

第二种构造方式就是通过字面量方式进行构造的。

其实基础类型基本都是通过字面量进行构造的：

```swift
let num: Int = 10
let flag: Bool = true
let str: String = "Brazil"
let array: [String] = ["Brazil", "Ghana"]
```

而这些都有对应的字面量协议：

```swift
ExpressibleByNilLiteral // nil字面量协议
ExpressibleByIntegerLiteral // 整数字面量协议
ExpressibleByFloatLiteral // 浮点数字面量协议
ExpressibleByBooleanLiteral // 布尔值字面量协议
ExpressibleByStringLiteral // 字符串字面量协议
ExpressibleByArrayLiteral // 数组字面量协议
```

###  Sequence

Sequence翻译过来就是序列，该协议的目的是一系列相同类型的值的集合，并且提供对这些值的迭代能力，这里的迭代可以理解为遍历，也即`for-in`的能力。可以看下该协议的定义：

```swift
protocol Sequence {
    associatedtype Iterator: IteratorProtocol
    func makeIterator() -> Iterator
}
```

`Sequence`又引入了另一个协议`IteratorProtocol`，该协议就是为了提供序列的迭代能力。

```swift
public protocol IteratorProtocol {
    associatedtype Element
    public mutating func next() -> Self.Element?
}
```

我们通常用`for-in`实现数组的迭代：

```swift
let animals = ["Antelope", "Butterfly", "Camel", "Dolphin"]
for animal in animals {
    print(animal)
}
```

这里的`for-in`会被编译器翻译成：

```swift
var animalIterator = animals.makeIterator()
while let animal = animalIterator.next() {
    print(animal)
}
```

###  Collection

Collection译为集合，其继承于Sequence。

```swift
public protocol Collection : Sequence {
  associatedtype Index : Comparable
  var startIndex: Index { get }
  var endIndex: Index { get }
  var isEmpty: Bool { get }
  var count: Int { get }
  
  subscript(position: Index) -> Element { get }
  subscript(bounds: Range<Index>) -> SubSequence { get }
}
```

是一个元素可以反复遍历并且可以通过索引的下标访问的有限集合，注意`Sequence`可以是无限的，`Collection`必须是有限的。

`Collection`在`Sequence`的基础上扩展了下标访问、元素个数能特性。我们常用的集合类型`Array`，`Dictionary`，`Set`都遵循该协议。


### CustomStringConvertible

这个协议表示自定义类型输出的样式。先来看下它的定义：

```swift
public protocol CustomStringConvertible {
    var description: String { get }
}
```

只有一个`description`的属性。它的使用很简单：

```swift
struct Point: CustomStringConvertible {
    let x: Int, y: Int
    var description: String {
        return "(\(x), \(y))"
    }
}

let p = Point(x: 21, y: 30)
print(p) // (21, 30)
//String(describing: <#T##CustomStringConvertible#>)
let s = String(describing: p)
print(s) // (21, 30)
```

如果不实现`CustomStringConvertible`，直接打印对象，系统会根据默认设置进行输出。我们可以通过`CustomStringConvertible`对这一输出行为进行设置，还有一个协议是`CustomDebugStringConvertible`：

```swift
public protocol CustomDebugStringConvertible {
    var debugDescription: String { get }
}
```

跟`CustomStringConvertible`用法一样，对应`debugPrint`的输出。

### Hashable

我们常用的`Dictionary`，`Set`均实现了`Hashable`协议。Hash的目的是为了将查找集合某一元素的时间复杂度降低到O(1)，为了实现这一目的需要将集合元素与存储地址之间建议一种尽可能一一对应的关系。

我们再看Hashable`协议的定义：

```swift
public protocol Hashable : Equatable {
		var hashValue: Int { get }
    func hash(into hasher: inout Hasher)
}

public protocol Equatable {
    static func == (lhs: Self, rhs: Self) -> Bool
}
```

注意到`func hash(into hasher: inout Hasher)`，Swift 4.2 通过引入 `Hasher` 类型并采用新的通用哈希函数进一步优化 `Hashable`。

如果你要自定义类型实现 `Hashable` 的方式，可以重写 `hash(into:)` 方法而不是 `hashValue`。`hash(into:)` 通过传递了一个 `Hasher` 引用对象，然后通过这个对象调用 `combine(_:)` 来添加类型的必要状态信息。

```swift
// Swift >= 4.2
struct Color: Hashable {
    let red: UInt8
    let green: UInt8
    let blue: UInt8

    // Synthesized by compiler
    func hash(into hasher: inout Hasher) {
        hasher.combine(self.red)
        hasher.combine(self.green)
        hasher.combine(self.blue)
    }

    // Default implementation from protocol extension
    var hashValue: Int {
        var hasher = Hasher()
        self.hash(into: &hasher)
        return hasher.finalize()
    }
}
```

参考：[Hashable / Hasher](https://nshipster.cn/hashable/)

### Codable

`Codable`是可`Decodable`和`Encodable`的类型别名。它能够将程序内部的数据结构序列化成可交换数据，也能够将通用数据格式反序列化为内部使用的数据结构，大大提升对象和其表示之间互相转换的体验。处理的问题就是我们经常遇到的JSON转模型，和模型转JSON。

```swift
public typealias Codable = Decodable & Encodable

public protocol Decodable {
    init(from decoder: Decoder) throws
}
public protocol Encodable {
    func encode(to encoder: Encoder) throws
}
```

这里只举一个简单的解码过程：

```swift
//json数据
{
    "id": "1283984",
    "name": "Mike",
  	"age": 18
}
// 定义对象
struct Person: Codable{
    var id: String
    var name: String
  	var age: Int
}
// json为网络接口返回的Data类型数据
let mike = try! JSONDecoder().decode(Person.self, from: json)
print(mike)
//输出：Student(id: "1283984", name: "Mike", age: 18)
```

是不是非常简单，Codable还支持各种自定义解编码过程，完全可以取代`SwiftyJSON`，`HandyJSON`等编解码库。

### Comparable

这个是用于实现比较功能的协议，它的定义如下：

```swift
public protocol Comparable : Equatable {
  
    static func < (lhs: Self, rhs: Self) -> Bool

    static func <= (lhs: Self, rhs: Self) -> Bool

    static func >= (lhs: Self, rhs: Self) -> Bool

    static func > (lhs: Self, rhs: Self) -> Bool
}
```

其继承于`Equatable`，即判等的协议。可以很清楚的理解实现了各种比较的定义就具有了比较的功能。这个不做比较。

### RangeReplaceableCollection

`RangeReplaceableCollection`支持用另一个集合的元素替换元素的任意子范围的集合。

看下它的定义：

```swift
public protocol RangeReplaceableCollection : Collection where Self.SubSequence : RangeReplaceableCollection {

    associatedtype SubSequence
  
  	mutating func append(_ newElement: Self.Element)
  	mutating func insert<S>(contentsOf newElements: S, at i: Self.Index) where S : Collection, Self.Element == S.Element
  	/* 拼接、插入、删除、替换的方法，他们都具有对组元素的操作能力 */
  
  	override subscript(bounds: Self.Index) -> Self.Element { get }
    override subscript(bounds: Range<Self.Index>) -> Self.SubSequence { get }
}
```

举个例子，Array支持该协议，我们可以进行如下操作：

```swift
var bugs = ["Aphid", "Damselfly"]
bugs.append("Earwig")
bugs.insert(contentsOf: ["Bumblebee", "Cicada"], at: 1)
print(bugs)
// Prints "["Aphid", "Bumblebee", "Cicada", "Damselfly", "Earwig"]"
```

这里附一张Swift中Array遵循的协议关系图，有助于大家理解上面讲解的几个协议之间的关系：



![img](https://user-gold-cdn.xitu.io/2020/5/10/171ff1f43fb51bfc?w=1075&h=369&f=svg&s=21578)


图像来源：https://swiftdoc.org/v3.1/type/array/hierarchy/

## 二、@propertyWrapper

> 阅读以下代码，print 输出什么

```swift
@propertyWrapper
struct Wrapper<T> {
    var wrappedValue: T

    var projectedValue: Wrapper<T> { return self }

    func foo() { print("Foo") }
}
struct HasWrapper {
    @Wrapper var x = 0
    
    func foo() {
        print(x) // 0
        print(_x) // Wrapper<Int>(wrappedValue: 0)
        print($x) // Wrapper<Int>(wrappedValue: 0)
     }
}
```

这段代码看似要考察对`@propertyWrapper`的理解，但是有很多无用内容，导致代码很奇怪。

`@propertyWrapper`的意思就是属性包装，它可以将一系列相似的属性方法进行统一处理。举个例子，如果我们需要在`UserDefaults`中加一个是否首次启动的值，正常可以这样处理：

```swift
extension UserDefaults {
  	enum Keys {
      static let isFirstLaunch = "isFirstLaunch"
    }
    var isFirstLaunch: Bool {
        get {
            return bool(forKey: Keys.isFirstLaunch)
        }
        set {
            set(newValue, forKey: Keys.isFirstLaunch)
        }
    }
}
```

如果我们需要加入很多这样属性的话，就需要写大量的`get` 、`set`方法。而`@propertyWrapper`的作用就是为属性的这种设置提供一个模板写法，以下是使用属性包装的写法。

```swift
@propertyWrapper
struct UserDefaultWrapper<T> {
    private let key: String
    private let defaultValue: T
    init(key: String, defaultValue: T) {
        self.key = key
        self.defaultValue = defaultValue
    }
  
    var wrappedValue: T {
        get {
            UserDefaults.standard.object(forKey: key) as? T ?? defaultValue
        }
        set {
            UserDefaults.standard.set(newValue, forKey: key)
        }
    }
}

extension UserDefaults {
  	@UserDefaultWrapper(key: Keys.isFirstLaunch, defaultValue: false)
    var isFirstLaunch: Bool
}

```

`@propertyWrapper`约束的对象必须要定义`wrappedValue`属性，因为该对象包裹的属性会走到`wrappedValue`的实现。

回到实例代码，定义了`wrappedValue`却并没有添加任何实现，这是允许的。所以访问x的时候其实是访问`Wrapper`的`wrappedValue`，因为没有给出任何实现所以直接打印出`0`。而`_x`和`$x`对应的就是`Wrapper`自身。

参考：[Swift Property Wrappers](https://nshipster.com/propertywrapper/)

## 三、关键字

### public open 

`public` `open`为权限关键词。对于一个严格的项目来说，精确的最小化访问控制级别对于代码的维护来说相当重要的。完整的权限关键词，按权限大小排序如下：

`open > public > internal > fileprivate > private`

* `open`权限最大，允许外部module访问，继承，重写。
* `public`允许外部module访问，但不允许继承，重写。
* `internal`为默认关键词，在同一个module内可以共用。
* `fileprivate`表示代码可以在当前文件中被访问，而不做类型限定。
* `private`表示代码只能在当前作用域或者同一文件中同一类型的作用域中被使用。

这些权限关键词可以修饰，属性，方法和类型。需要注意：当一个类型的某一属性要用public修饰时，该类型至少要用public（或者open）权限的关键词修复。可以理解为数据访问是分层的，我们为了获取某一属性或方法需要先获取该类型，所以外层（类型）的访问权限要满足大于等于内层（类型、方法、属性）权限。

参考：[Swift AccessControl](https://docs.swift.org/swift-book/LanguageGuide/AccessControl.html)

### static class final

原文中`final`跟权限关键词放在一起了，其实是不合理的，就将其放到这里来讨论。

`static`静态变量关键词，来源于C语言。

在Swift中常用语以下场景：

```swift
// 仅用于类名前，表示该类不能被继承。仅支持class类型
final class Manager {
  	// 单例的声明
		static let shared = Manager()
  	// 实例属性，可被重写
  	var name: String = "Ferry"
  	// 实例属性，不可被重写
  	final var lastName: String = "Zhang"
  	// 类属性，不可被重写
  	static var address: String = "Beijing"
  	// 类属性，可被重写。注意只能作为计算属性，而不能作为存储属性
  	class var code: String {
    		return "0122"
    }
  
  	// 实例函数，可被重写
  	func download() {
      /* code... */
    }
  	// 实例函数，不可被重写
  	final func download() {
      /* code... */
    }
  	// 类函数，可被重写
  	class func removeCache() {
     	/* code... */ 
    }
  	// 类函数，不可被重写
  	static func download() {
      /* code... */
    }
}

struct Manager {
  	// 单例的声明
		static let shared = Manager()
  	// 类属性
  	static var name: String = "Ferry"
  	// 类函数
  	static func download() {
      /* code... */
    }
}
```

`struct`和`enum`因为不能被继承，所以也就无法使用`class`和`final`关键词，仅能通过`static`关键词进行限定

### mutating inout

mutating用于修饰会改变该类型的函数之前，基本都用于`struct`对象的修改。看下面例子：

```swift
struct Point {
    var x: CGFloat
    var y: CGFloat
		// 因为该方法改变了struct的属性值（x），所以必须要加上mutating
    mutating func moveRight(offset: CGFloat) {
        x += offset
    }

  	func normalSwap(a: CGFloat, b: CGFloat) {
        let temp = a
        a = b
        b = temp
    }
		// 将两个值交换，需传入对象地址。注意inout需要加载类型名前
    func inoutSwap(a: inout CGFloat, b: inout CGFloat) {
        let temp = a
        a = b
        b = temp
    }
}

var location1: CGFloat = 10
var location2: CGFloat = -10

var point = Point.init(x: 0, y: 0)
point.moveRight(offset: location1)
print(point)	//Point(x: 10.0, y: 0.0)

point.normalSwap(a: location1, b: location2)
print(location1)	//10
print(location2)	//-10
// 注意需带取址符&
point.inoutSwap(a: &location1, b: &location2)
print(location1)	//-10
print(location2)	//10
```

`inout`需要传入取值符，所以它的改变会导致该对象跟着变动。可以再回看上面说的`Hashable`的一个协议实现：

```swift
func hash(into hasher: inout Hasher) {
    hasher.combine(self.red)
    hasher.combine(self.green)
    hasher.combine(self.blue)
}
```

只有使用`inout`才能修改传入的hasher的值。

### infix operator

`infix operator`即为中缀操作符，还有prefix、postfix后缀操作符。

它的作用是自定义操作符。比如Python里可以用`**`进行幂运算，但是Swift里面，我们就可以利用自定义操作符来定义一个用`**`实现的幂运算。

```swift
// 定义中缀操作符
infix operator **
// 实现该操作符的逻辑，中缀需要两个参数
func ** (left: Double, right: Double) -> Double {
    return pow(left, right)
}
let number = 2 ** 3
print(value) //8
```

同理我们还可以定义前缀和后缀操作符：

```swift
//定义阶乘操作，后缀操作符
postfix operator ~!
postfix func ~! (value: Int) -> Int {

    func factorial(_ value: Int) -> Int {
        if value <= 1 {
            return 1
        }
        return value * factorial(value - 1)
    }
    return factorial(value)
}
//定义输出操作，前缀操作符
prefix operator <<
prefix func << (value: Any) {
    print(value)
}

let number1 = 4~!
print(number1) // 24

<<number1 // 24
<<"zhangferry" // zhangferry
```

前缀和后缀仅需要一个操作数，所以只有一个参数即可。

关于操作符的更多内容可以查看这里：[Swift Operators](https://nshipster.cn/swift-operators/)。

注意，因为该文章较早，其中对于操作符的一些定义已经改变。

### @dynamicMemberLookup，@dynamicCallable

这两个关键词我确实没有用过，看到`dynamic`可以知道这两个特性是为了让Swift具有动态性。

`@dynamicMemberLookup`中文叫动态查找成员。在使用`@dynamicMemberLookup`标记了对象后（对象、结构体、枚举、protocol），实现了`subscript(dynamicMember member: String)`方法后我们就可以访问到对象不存在的属性。如果访问到的属性不存在，就会调用到实现的 `subscript(dynamicMember member: String)`方法，key 作为 member 传入这个方法。 举个例子：

```swift
@dynamicMemberLookup
struct Person {
    subscript(dynamicMember member: String) -> String {
        let properties = ["nickname": "Zhuo", "city": "Hangzhou"]
        return properties[member, default: "undefined"]
    }
}
//执行以下代码
let p = Person()
print(p.city)	//Hangzhou
print(p.nickname)	//Zhuo
print(p.name)	//undefined
```

我们没有定义Person的`city`、`nickname`，`name`属性，却可以用点语法去尝试访问它。如果没有`@dynamicMemberLookup`这种写法会被编译器检查出来并报错，但是加了该关键词编译器就不会管它是不是存在都予以通过。

```swift
@dynamicCallable
struct Person {
    // 实现方法一
    func dynamicallyCall(withArguments: [String]) {
        for item in withArguments {
            print(item)
        }
    }
    // 实现方法二
    func dynamicallyCall(withKeywordArguments: KeyValuePairs<String, String>){
        for (key, value) in withKeywordArguments {
            print("\(key) --- \(value)")
        }
    }
}
let p = Person()
p("zhangsan")
// 等于 p.dynamicallyCall(withArguments: ["zhangsan"])
p("zhangsan", "20", "男")
// 等于 p.dynamicallyCall(withArguments: ["zhangsan", "20", "男"])
p(name: "zhangsan")
// 等于 p.dynamicallyCall(withKeywordArguments: ["name": "zhangsan"])
p(name: "zhangsan", age:"20", sex: "男")
// 等于 p.dynamicallyCall(withKeywordArguments: ["name": "zhangsan", "age": "20", "sex": "男"])
```

`@dynamicCallable`可以理解成动态调用，当为某一类型做此声明时，需要实现`dynamicallyCall(withArguments:)`或者`dynamicallyCall(withKeywordArguments:)`。编译器将允许你调用并为定义的方法。

一个动态查找成员变量，一个动态方法调用，带上这两个特性Swift就可以变成彻头彻尾的动态语言了。所以作为静态语言的Swift也是可以具有动态特性的。

更多关于这两个动态标记的讨论可以看卓同学的这篇：[细说 Swift 4.2 新特性：Dynamic Member Lookup](https://juejin.im/post/5b24c9896fb9a00e69608a71)

### where

where一般用作条件限定。它可以用在for-in、swith、do-catch中：

```swift
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9]
for item in numbers where item % 2 == 1 {
    print("odd: \(item)")	// 将输出1，3，5，7，9等数
}

numbers.forEach { (item) in
    switch item {
    case let x where x % 2 == 0:
        print("even: \(x)") // 将输出2，4，6，8等数
    default:
        break
    }
}
```

`where`也可以用于类型限定。

我们可以扩展一个字典的merge函数，它可以将两个字典进行合并，对于相同的`Key`值以要合并的字典为准。并且该方法我只想针对`Key`和`Value`都是`String`类型的字典使用，就可以这么做：

```swift
// 这里的Key Value来自于Dictionary中定义的泛型
extension Dictionary where Key == String, Value == String {
    //同一个key操作覆盖旧值
    func merge(other: Dictionary) -> Dictionary {
        return self.merging(other) { _, new in new }
    }
}
```

### @autoclosure

`@autoclosure` 是使用在闭包类型之前，做的事情就是把一句表达式自动地**封装**成一个闭包 (closure)。

比如我们有一个方法接受一个闭包，当闭包执行的结果为 `true` 的时候进行打印，分别使用普通闭包和加上`autoclosure`的闭包实现：

```swift
func logIfTrueNormal(predicate: () -> Bool) {
    if predicate() {
        print("True")
    }
}
// 注意@autoclosure加到闭包的前面
func logIfTrueAutoclosure(predicate: @autoclosure () -> Bool) {
    if predicate() {
        print("True")
    }
}
// 调用方式
logIfTrueNormal(predicate: {3 > 1})
logIfTrueAutoclosure(predicate: 3 > 1)
```

编译器会将`logIfTrueAutoclosure`函数参数中的`3 > 1`这个表达式转成`{3 > 1}`这种尾随闭包样式。

那这种写法有什么用处呢？我们可以从一个示例中体会一下，在Swift系统提供的几个短路运算符（即表达式左边如果已经确定结果，右边将不再运算）中均采用了`@autoclosure`标记的闭包。那`??`运算符举例，它的实现是这样的：

```swift
public func ?? <T>(optional: T?, defaultValue: @autoclosure () throws -> T)
    rethrows -> T {
  switch optional {
  case .some(let value):
    return value
  case .none:
    return try defaultValue()
  }
}
// 使用
var name: String? = "ferry"
let currentName = name ?? getDefaultName()
```

因为使用了`@autoclosure`标记闭包，所以`??`的`defaultValue`参数我们可以使用表达式，又因为是闭包，所以当`name`非空时，直接返回了该值，不会调用`getDefaultName()`函数，减少计算。

参考：[@AUTOCLOSURE 和 ??](https://swifter.tips/autoclosure/)，注意因为Swift版本问题，实例代码无法运行。

### @escaping

`@escaping`也是闭包修饰词，用它标记的闭包被称为逃逸闭包，还有一个关键词是`@noescape`，用它修饰的闭包叫做非逃逸闭包。在Swift3及之后的版本，闭包默认为非逃逸闭包，在这之前默认闭包为逃逸闭包。

这两者的区别主要在于声明周期的不同，当闭包作为参数时，如果其声明周期与函数一致就是非逃逸闭包，如果声明周期大于函数就是逃逸闭包。结合示例来理解：

```swift
// 非逃逸闭包
func logIfTrueNormal(predicate: () -> Bool) {
    if predicate() {
        print("True")
    }
}
// 逃逸闭包
func logIfTrueEscaping(predicate: @escaping () -> Bool) {
    DispatchQueue.main.async {
        if predicate() {
            print("True")
        }
    }
}
```

第二个函数的闭包为逃逸闭包是因为其是异步调用，在函数退出时，该闭包还存在，声明周期长于函数。

如果你无法判断出应该使用逃逸还是非逃逸闭包，也无需担心，因为编译器会帮你做出判断。第二个函数，如果我们不声明逃逸闭包编译器会报错，警告我们：`Escaping closure captures non-escaping parameter 'predicate'`。当然我们还是应该理解两者的区别。

## 四、高阶函数

### Filter, Map, Reduce, flatmap, compactMap

这几个高阶函数都是对数组对象使用的，我们通过示例去了解他们吧：

```swift
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9]
// filter 过滤
let odd = numbers.filter { (number) -> Bool in
    return number % 2 == 1
}
print(odd) // [1, 3, 5, 7, 9]

//map 转换
let maps = odd.map { (number) -> String in
    return "\(number)"
}
print(maps) // ["1", "3", "5", "7", "9"]

// reduce 累计运算
let result = odd.reduce(0, +)
print(result) // 25

// flatMap 1.数组展开
let numberList = [[1, 2, 3], [4, 5]， [[6]]]
let flatMapNumber = numberList.flatMap { (value) in
    return value
}
print(flatMapNumber) // [1, 2, 3, 4, 5, [6]]

// flatMap 2.过滤数组中的nil
let country = ["cn", "us", nil, "en"]
let flatMap = country.flatMap { (value) in
    return value
}
print(flatMap) //["cn", "us", "en"]

// compactMap 过滤数组中的nil
let compactMap = country.compactMap { (value) in
    return value
}
print(compactMap) // ["cn", "us", "en"]
```

filter，reduce其实很好理解，map、flatMap、compactMap刚开始接触时确实容易搞混，这个需要多加使用和练习。

注意到`flatMap`有两种用法，一种是展开数组，将二维数组降为一维数组，一种是过滤数组中的`nil`。在Swift4.1版本已经将`flatMap`过滤数组中nil的函数标位`deprecated`，所以我们过滤数组中nil的操作应该使用`compactMap`函数。

参考：[Swift 烧脑体操（四） - map 和 flatMap](https://blog.devtang.com/2016/03/05/swift-gym-4-map-and-flatmap/#)


## 五、几个Swift中的概念

### 柯里化什么意思

柯里化指的是从一个多参数函数变成一连串单参数函数的变换，这是实现函数式编程的重要手段，举个例子：

```swift
// 该函数返回类型为（Int） -> Bool
func greaterThan(_ comparer: Int) -> (Int) -> Bool {
    return { number in
        return number > comparer
    }
}
// 定义一个greaterThan10的函数
let greaterThan10 = greaterThan(10)
greaterThan10(13)    // => true
greaterThan10(9)     // => false
```

所以柯里化也可以理解为批量生成一系列相似的函数。

参考：[柯里化 (CURRYING)](https://swifter.tips/currying/)

### `POP` 与 `OOP`的区别

OOP(object-oriented programming)面向对象编程：

在面向对象编程世界里，一切皆为对象，它的核心思想是继承、封装、多态。

POP(protocol-oriented programming)面向协议编程：

面向协议编程则主要通过协议，又或叫做接口对一系列操作进行定义。面向协议也有继承封装多态，只不过这些不是针对对象建立的。


为什么Swift演变成了一门面向协议的编程语言。这是因为面向对象存在以下几个问题：

1、动态派发的安全性（这应该是OC的困境，在Swift中Xcode是不可能让这种问题编译通过的）

2、横切关注点（Cross-Cutting Concerns）问题。面向对象无法描述两个不同事物具有某个相同特性这一点。

3、菱形问题（比如C++中）。C++可以多继承，在多继承中，两个父类实现了相同的方法，子类无法确定继承哪个父类的此方法，由于多继承的拓扑结构是一个菱形，所以这个问题有被叫做菱形缺陷（Diamond Problem）。

参考文章：

[Swift 中的面向协议编程：是否优于面向对象编程？](https://swift.gg/2018/12/03/pop-vs-oop/)

[面向协议编程与 Cocoa 的邂逅 (上)](https://onevcat.com/2016/11/pop-cocoa-1/)

### `Any` 与`AnyObject` 区别

**AnyObject**： 是一个协议，所有class都遵守该协议，常用语跟OC对象的数据转换。

**Any**：它可以代表任何型別的类(class)、结构体 (struct)、枚举 (enum)，包括函式和可选型，基本上可以说是任何东西。

### `rethrows` 和 `throws` 有什么区别呢？

throws是处理错误用的，可以看一个往沙盒写入文件的例子：

```swift
// 写入的方法定义
public func write(to url: URL, options: Data.WritingOptions = []) throws
// 调用
do {
    let data = Data()
    try data.write(to: localUrl)
} catch let error {
    print(error.localizedDescription)
}
```

将一个会有错误抛出的函数末尾加上`throws`，则该方法调用时需要使用`try`语句进行调用，用于提示当前函数是有抛错风险的，其中`catch`句柄是可以忽略的。

`rethrows`与`throws`并没有太多不同，它们都是标记了一个方法应该抛出错误。但是 `rethrows` 一般用在参数中含有可以 `throws` 的方法的高阶函数中（想一下为什么是高阶函数？下期给出答案）。

查看`map`的方法声明，我们能同时看到 `throws`,`rethrows`：

```swift
@inlinable public func map<T>(_ transform: (Element) throws -> T) rethrows -> [T]
```

不知道你们第一次见到`map`函数本体的时候会不会疑惑，为什么`map`里的闭包需要抛出错误？为什么我们调用的时候并没有用`try`语法也可以正常通过？

其实是这样的，`transform`是需要我们定义的闭包，它有可能抛出异常，也可能不抛出异常。Swift作为类型安全的语言就需要保证在有异常的时候需要使用try去调用，在没有异常的时候要正常调用，那怎么兼容这两种情况呢，这就是`rethrows`的作用了。

```swift
func squareOf(x: Int) -> Int {return x * x}

func divideTenBy(x: Int) throws -> Double {
    guard x != 0 else {
        throw CalculationError.DivideByZero
    }
    return 10.0 / Double(x)
}

let theNumbers = [10, 20, 30]
let squareResult = theNumbers.map(squareOf(x:)) // [100, 400, 9000]

do {
    let divideResult = try theNumbers.map(divideTenBy(x:))
} catch let error {
    print(error)
}
```

当我们直接写`let divideResult = theNumbers.map(divideTenBy(x:))`时，编译器会报错：`Call can throw but is not marked with 'try'`。这样就实现了根据情况去决定是否需要用`try-catch`去捕获map里的异常了。

参考：[错误和异常处理](https://swifter.tips/error-handle/)

### break return continue fallthough 在语句中的含义（switch、while、for）

这个比较简单，只说相对特别的示例吧，在Swift的switch语句，会在每个case结束的时候自动退出该switch判断，如果我们想不退出，继续进行下一个case的判断，可以加上`fallthough`。