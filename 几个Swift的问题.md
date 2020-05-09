希望看完此文后，你对自己Swift继续保持信心

## 一、 协议 Protocol

### `ExpressibleByDictionaryLiteral`

字典的字面量协议，该协议的完整写法为。

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

首先字面量（Literal）的意思是：**字面量（literal）是用于表达源代码中一个固定值的表示法（notation）**。

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

其实基础类型基本都是通过字面量就行构造的，像

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



###  `Sequence`

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



###  `Collection`

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

是一个元素可以反复遍历并且可以通过索引的下标访问的有限集合，注意Sequence可以是无限的，Collection必须是有限的。

Collection在Sequence的基础上扩展了下标访问、元素个数能特性。我们常用的集合类型`Array`，`Dictionary`，`Set`都遵循该协议。



- `CustomStringConvertible`

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
print(p) //>> (21, 30)
//String(describing: <#T##CustomStringConvertible#>)
let s = String(describing: p)
print(s) //>> (21, 30)
```

如果不实现`CustomStringConvertible`，直接打印对象，系统会根据默认设置进行输出。我们可以通过`CustomStringConvertible`对这一输出行为进行设置，还有一个协议是`CustomDebugStringConvertible`：

```swift
public protocol CustomDebugStringConvertible {
    var debugDescription: String { get }
}
```

跟`CustomStringConvertible`用法一样，对应`debugPrint`方法。

###  `Hashable` `Codable`

我们常用的`Dictionary`，`Set`均实现了`Hashable`协议。Hash的目的是为了将查找集合某一元素的时间复杂度降低到O(1)，所以需要将集合元素与存储地址之间建议一种尽可能一一对应的关系。

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

关于`Hashable`协议的进一步讨论可以移步Mattt的[Hashable / Hasher](https://nshipster.cn/hashable/)这篇文章。



###  `Codable`

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
let mike = try JSONDecoder().decode(Person.self, from: json)
print(mike)
//输出：Student(id: "1283984", name: "Mike", age: 18)
```

是不是非常简单，Codable还支持各种自定义解编码过程，完全可以取代`SwiftyJSON`，`HandyJSON`等编解码库。

- `Comparable`

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

###  `RangeReplaceableCollection`

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

关于`@PropertyWrapper`特性的进一步介绍可以参考这里：[Swift Property Wrappers](https://nshipster.com/propertywrapper/)

## 三、关键字

> 以下关键词的使用场景是什么？

### `public` `open` 

`public` `open`为权限关键词。对于一个严格的项目来说，精确的最小化访问控制级别对于代码的维护来说相当重要的。完整的权限关键词，按权限大小排序如下：

`open > public > internal > fileprivate > private`

* `open`权限最大，允许外部module访问，继承，重写。
* `public`允许外部module访问，但不允许继承，重写。
* `internal`为默认关键词，在同一个module内可以共用。
* `fileprivate`表示代码可以在当前文件中被访问，而不做类型限定。
* `private`表示代码只能在当前作用域或者同一文件中同一类型的作用域中被使用。

这些权限关键词可以修饰，属性，方法和类型。需要注意：当一个类型的某一属性要用public修饰时，该类型至少要用public（或者open）权限的关键词修复。可以理解为数据访问是分层的，我们为了获取某一属性或方法需要先获取该类型，所以外层（类型）的访问权限要满足大于等于内层（类型、方法、属性）权限。

参考：[Swift AccessControl](https://docs.swift.org/swift-book/LanguageGuide/AccessControl.html)

### `static` `class` `final`

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

### `mutating`  `inout`

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

### `infix operator`

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

关于操作符的更多内容可以查看这里：[Swift Operators](https://nshipster.cn/swift-operators/)。注意，因为该文章较早，其中对于操作符的一些定义已经改变。

### `@dynamicMemberLookup`，`@dynamicCallable`

这两个关键词我确实没有用过，看到`dynamic`可以知道这两个特性是为了让Swift具有动态性。

这个特性中文叫动态查找成员。在使用`@dynamicMemberLookup`标记了对象后（对象、结构体、枚举、protocol），实现了`subscript(dynamicMember member: String)`方法后我们就可以访问到对象不存在的属性。如果访问到的属性不存在，就会调用到实现的 `subscript(dynamicMember member: String)`方法，key 作为 member 传入这个方法。 举个例子：

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

更多关于这两个动态标记的讨论可以看卓同学的这篇：[细说 Swift 4.2 新特性：Dynamic Member Lookup](https://juejin.im/post/5b24c9896fb9a00e69608a71)


- `where`
- `@autoclosure`
- `@escaping`

## 四、高阶函数

- Filter, Map, Reduce, flatmap, compactMap

> 有何异同？

## 五、其他

- `柯里化` 什么意思
- `POP` 与 `OOP`的区别
- `Any` 与`AnyObject` 区别
- `rethrows` 和 `throws` 有什么区别呢？
- `break` `return` `continue` `fallthough` 在语句中的含义（switch、while、for）



pod repo push SealRepo MeeviiAds.podspec --sources=git@bitbucket.org:sealcn/sealrepo.git,https://github.com/CocoaPods/Specs --allow-warnings --skip-import-validation --skip-tests --no-private --verbose