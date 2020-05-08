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

Collection译为集合协议，其继承于Sequence。

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

是一个元素可以反复遍历并且可以通过索引的下标访问的有限集合，注意Sequence可以是无限的，Collection必须是有限的。我们常用的集合类型`Array`，`Dictionary`，`Set`都遵循该协议。



- `CustomStringConvertible`
- `Hashable` `Codable`
- `Comparable`
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