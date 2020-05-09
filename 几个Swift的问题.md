希望看完此文后，你对自己Swift继续保持信心

#### 一、 协议 Protocol

- `ExpressibleByDictionaryLiteral`

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



- `Sequence`

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



- `Collection`

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

- `Hashable` `Codable`

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



`Codable`

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

其继承于`Equatable`，即判等的协议。可以很清楚的理解实现了各种比较的定义就具有了比较的功能。

- `RangeReplaceableCollection`



> 以上协议常见应用场景是什么，有什么作用？

#### 二、@propertyWrapper

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
            print(x) // `wrappedValue`
            print(_x) // wrapper type itself
            print($x) // `projectedValue`
        }
    }
```

#### 三、关键字

- `public` `open` `final`
- `static` `class`
- `mutating`  `inout`
- `infix operator`
- `dynamicMemberLookup`
- `where`
- `@dynamicCallable`
- `@autoclosure`
- `@escaping`

> 以上关键字使用场景是什么？

#### 四、高阶函数

- Filter, Map, Reduce, flatmap, compactMap

> 有何异同？

#### 五、其他

- `柯里化` 什么意思
- `POP` 与 `OOP`的区别
- `Any` 与`AnyObject` 区别
- `rethrows` 和 `throws` 有什么区别呢？
- `break` `return` `continue` `fallthough` 在语句中的含义（switch、while、for）



pod repo push SealRepo MeeviiAds.podspec --sources=git@bitbucket.org:sealcn/sealrepo.git,https://github.com/CocoaPods/Specs --allow-warnings --skip-import-validation --skip-tests --no-private --verbose