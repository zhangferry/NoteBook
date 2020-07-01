#【译】更好的了解Xcode构建系统

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200628234308.png)

> 原文：https://medium.com/flawless-app-stories/xcode-build-system-know-it-better-96936e7f52a
>
> 作者：Varun Tomar

一个程序在运行到一台设备之前经历了很多转换的步骤。和其它的编程语言处理系统一样，Xcode构建系统为了确保执行顺序和各种依赖库，需要运行很多命令行指令，传递各种各样的参数。整个构建过程分为以下五个阶段：

1、预处理

2、编译

3、汇编

4、链接

5、加载



## 预处理

预处理的目的是把我们的程序转换成能被编译器识别的形式。它用宏的定义替换宏，发现依赖关系并解析预处理器指令。如果Swift编译器没有预处理器，我们则不能在Swift项目中定义宏命令。在Xcode8中是允许我们通过`Build Setting`中的`SWIFT_ACTIVE_COMPILATION_CONDITIONS`定义预处理器标记位的。它跟OC中的预处理器宏定义是一致的。

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200624180433.png)



## 编译器

编译是整个过程中非常重要的一环。编译器是一个程序，它把高级语言像是Swift&Objective C转换成低级语言，像是目标文件。在iOS中有两类编译器『Clange & swiftc』。该过程可以描述为：

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200628234338.png)

**注意**：编译器包含两个主要部分：前端和后端。Clange是C/C++/Objective-C的编译前端，swiftc是Swift的编译前端，LLVM是后端。这些乱七八糟的东西是什么，LLVM是从哪里来的?

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200628234359.png)

不要担心😉，我将会对它进行简明扼要的介绍，尽管其中细节说明可能要单独出一篇文章才能说明白（之后的文章，我会就这方面写一篇文章）。

> LLVM(*Low Level Virtual Machine*)是一个后端编译器，用于在其上构建编译器。它处理优化和生产适应目标架构(ARM、x86)的代码。Clang/swiftc是一个前端编译器，可以解析C、C++、Objective C和Swift代码，并将其转换为适合LLVM的中间表示(IR)。在本文后面，您将更好地理解它。



让我们更细粒度的理解下Swift语言的编译过程，参考下面的图示：

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200628234416.png)



**接下来介绍Swift编译器示如何一步一步工作的：**

1、Swift代码解析成AST **(Abstract Syntax Tree抽象语法树)**。AST是一个代表源码结构的抽象语法树，树的每个节点代表一个结构。它是什么样的呢，让我们通过一个例子来理解它。下面是我们将要进行分析的Swift代码：

```swift
//
//  Test.swift
//
//  Created by Varun Tomar
//  Copyright © 2020 Varun Tomar. All rights reserved.
//

import Foundation

class Bird {
  func fly() { }
}

func isFlyHigh(bird: Bird) -> Bool { return false }

class Sparrow: Bird {
  override func fly() { }
  func add(x: Int, y: Int) -> Int { return x + y }
}
```

我们可以将这段代码转换成抽象语法树格式内容，这需要运行

```shell
xcrun swiftc -dump-ast <filename>.swift
```

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200628235348.png)

抽象语法树输出的结果很有趣，这里只展示了部分。如果你去阅读这段输出内容，会理解一些AST背后发生的事情。下面这段代码展示了一些有意思的东西：

```
(func_decl range=[Test.swift:12:3 - line:12:16] "fly()" interface type='(Bird) -> () -> ()' access=internal
(parameter "self")
```

注意：如上面所示，当我们创建一个函数时，swift会传递一个参数`self`，这就是为什么我们可以不传入self直接访问这些函数的原因。

```
(class_decl range=[Test.swift:17:1 - line:20:1] "Sparrow" interface type='Sparrow.Type' access=internal non-resilient inherits: Bird
```

上面这段代码展示了语法树是如何描述继承关系的。

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200628234432.png)

2、现在我们来到了AST构建完成之后的语义分析。语义分析负责将AST转换为格式漂亮，类型完全检查的AST格式。它移除了一些源码语义问题上的警告和错误信息。



3、下一步是SIL的生成和优化。为了在这个阶段之后得到SIL，我们可以运行：

```
xcrun swiftc -emit-silgen <filename>.swift
```

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200628235415.png)

看到这些终端输出的SIL生成内容，你可能会发出这样的惊讶："OMG😮。`@$s4Test4BirdC3flyyyF`是个什么鬼东西？"。别担心，这没你想的那么可怕，它只是一种名称混淆，用于将实体的附加信息压缩到单个字符中。这个处理过的名称包含了一些类型（class/struct/enum）、module、上下文等信息。例如，在`@$s4Test4BirdC3flyyyF`中，*Bird*后面的字母`C`表示`Bird`是一个class。它还可以表达很多东西，但这不是本文的重点。如果你对它有兴趣，可以在反馈部分添加评论。此外我们可以使用`swift-demangle`追溯一个混淆的字符串直到它最初可读的文本样式。



可读的SIL可以通过如下命令实现：

```
xcrun swiftc -emit-silgen <filename>.swift | xcrun swift-demangle
```

我们可以看下它的结果：

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200628235432.png)

这次它变得更易读了。那就跟着我🚶‍♂️一起探索SIL吧：

* 函数是以关键词`nil`开始的。
* 关键词`hidden`跟Swift代码中的`internal`是对应的。
* `@main.Bird.fly() -> ()`是从混淆的文本`@$s4Test4BirdC3flyyyF`中解析出来的，代表着函数名。
* `$@convention(method)`意味着调用该函数需要一个上下文（context）。例如在`self.fly()`中，`self`就是函数调用的上下文。
* `$@convention(thin)`代表着这是一个自由函数，它的调用不需要上下文。
* 如果参数指定了类型，就需要`owned`的标记。

有没有感觉到很有趣🧐？我其实有更有趣的发现。。。

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200628234457.png)

4、在SIL的生成和优化之后，就来到了IR（中间件）阶段。IR是LLVM的输入内容。运行：

```
xcrun swiftc -emit-ir <filename>.swift
```

看到输出的机器语言时，你可能会一脸懵逼🤯🤯

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200628235451.png)



## 汇编

在这里，控制由汇编程序来将输入转换为可重定位的机器码。它生成Mach-O文件。

>Mach-O文件用于对象文件、可执行文件和库。它是以一些有意义的字节码组成的集合，将在iOS设备的ARM处理器或Mac设备的英特尔处理器上运行。



## 链接器和加载程序

链接器是一个将多个目标文件和库合并为一个Mach-O执行文件的程序。最后，操作系统中的加载器将程序装载入内存并执行。