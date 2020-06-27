#【译】更好的了解Xcode构建系统

原文：https://medium.com/flawless-app-stories/xcode-build-system-know-it-better-96936e7f52a

作者：Varun Tomar



一个程序在运行到一台设备之前是经历了很多转换的步骤的。和其它的编程语言处理系统一样，Xcode构建系统为了确保执行顺序和各种依赖库，需要运行很多命令行指令，传递各种各样的参数。整个构建过程分为以下五个阶段：

1、预处理

2、编译

3、汇编

4、链接

5、加载



## 预处理

预处理的目的是把我们的程序转换成能被编译器识别的形式。它用宏的定义替换宏，发现依赖关系并解析预处理器指令。如果Swift编译器没有预处理器，我们则不能在Swift项目中定义宏命令。在Xcode8中是允许我们通过`Build Setting`中的`SWIFT_ACTIVE_COMPILATION_CONDITIONS`定义预处理器标记位的。它跟OC中的预处理器宏定义是一致的。

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200624180433.png)



## 编译器

编译是整个过程中非常重要的一环。编译器是一个程序，它把高级语言像是Swift&Objective C转换成低级语言，像是object文件。在iOS中有两类编译器『Clange & swiftc』。该过程的抽象描述为：

![img](https://miro.medium.com/max/450/1*sryDiLA0zu5EhrPCUjZBew.png)

**注意**：编译器包含两个主要部分：前端和后端。Clange是C/C++/Objective-C的编译前端，swiftc是Swift的编译前端，LLVM是后端。这些乱七八糟的东西是什么，LLVM是从哪里来的?

![img](https://miro.medium.com/max/500/1*cb10Q6m1P8eHbU74uqyocQ.gif)

不要担心😉，我将会对它进行简明扼要的介绍，尽管这可能要单独出一篇文章说明其中细节（之后的文章，我会就这方面写一篇文章）。

> LLVM(*Low Level Virtual Machine*)是一个后端编译器，用于在其上构建编译器。它处理优化和生产适应目标架构(ARM、x86)的代码。CLang/Swiftc是一个前端编译器，可以解析C、c++、Objective C和Swift代码，并将其转换为适合LLVM的中间表示(IR)。在本文后面，您将更好地理解它。



让我们更细粒度的理解下Swift语言的编译过程，参考下面的图示：

![img](https://miro.medium.com/max/2058/1*wzgnQ32uL0GGi3XrX4rlTA.png)



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

我们可以将这段代码转换成抽象语法树格式内容，这需要运行：

```shell
xcrun swiftc -dump-ast <filename>.Swift
```

注：最后的参数是swift文件的路径，这里是定位到了编译文件所在目录，填入内容为`Test.swift`。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200627183014.png)

抽象语法树会输出很多有趣的结果，这里只展示了部分。如果你去阅读这段输出内容，会发现一些AST背后发生的事情。下面这段代码展示了几点有意思的东西：

```
(func_decl range=[Test.swift:12:3 - line:12:16] "fly()" interface type='(Bird) -> () -> ()' access=internal
(parameter "self")
```

注意：如上面所示，当我们创建一个函数时，swift会传递一个参数self，这就是为什么我们可以访问其他函数。

```
(class_decl range=[Test.swift:17:1 - line:20:1] "Sparrow" interface type='Sparrow.Type' access=internal non-resilient inherits: Bird
```

上面的语法树代码展示了如何实现继承关系。



2、现在我们来看看在构建AST时可以执行的语义分析。语义分析负责将AST转换为格式良好、类型完全检查的AST形式，删除源代码中语义问题的警告或错误。



3、下一步是SIL的生成和优化。为了在这个阶段之后得到SIL，我们可以运行：

```
xcrun swiftc -emit-silgen <filename>.swift
```

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200627184524.png)

看到这些终端输出的SIL生成内容，你可能会发出这种惊讶：“OMG😮,**@$s4Test4BirdC3flyyyF**是个什么鬼东西？” 别担心，这没你想的那么可怕，它就是一种**名称压缩**，把一些有用的信息合并（编码）成一个特定的字符串。这个编码的结果包含了类型（class/struct/enum）、module、上下文环境等等。举个例子，在`@$s4Test4BirdC3flyyyF`中，*Bird*后面的字母`C`代表着`Bird`是一个class。它其实还有很多特殊的表达方式，但这里我们不会对这一技术介绍太深入。如果你对它感兴趣，可以给在下方的评论区写出你的理解。此外我们还可以将这一类型的字符串变的更加易读