随着项目的扩大，依靠人工codereview来保证项目的质量，越来越不现实，这时就有必要借助于一种自动化的代码审查工具：程序静态分析。

程序静态分析（Program Static Analysis）是指在不运行代码的方式下，通过词法分析、语法分析、控制流、数据流分析等技术对程序代码进行扫描，验证代码是否满足规范性、安全性、可靠性、可维护性等指标的一种代码分析技术。（来自百度百科）

词法分析，语法分析等工作是由编译器进行的，所以对iOS项目为了完成静态分析，我们需要借助于编译器。对于OC语言的静态分析可以完全通过[Clang](http://clang.llvm.org/)，对于Swift的静态分析除了Clange还需要借助于[SourceKit](http://www.jpsim.com/uncovering-sourcekit)。

Swift语言对应的静态分析工具是SwiftLint，OC语言对应的静态分析工具有Infer和OCLitn。以下会是对各个静态分析工具的安装和使用做一个介绍。

## SwiftLint
对于Swift项目的静态分析可以使用[SwiftLint](https://github.com/realm/SwiftLint)。SwiftLint 是一个用于强制检查 Swift 代码风格和规定的一个工具。它的实现是 Hook 了 Clang 和 SourceKit 从而能够使用 [AST](http://clang.llvm.org/docs/IntroductionToTheClangAST.html) 来表示源代码文件的更多精确结果。Clange我们了解了，那SourceKit是干什么用的？

SourceKit包含在[Swift](https://github.com/apple/swift/tree/master/tools/SourceKit)项目的主仓库，它是一套工具集，支持Swift的大多数源代码操作特性：源代码解析、语法突出显示、排版、自动完成、跨语言头生成等工作。

### 安装

安装有两种方式，任选其一：
**方式一：通过Homebrew**

```
brew install swiftlint
```
这种是全局安装，各个应用都可以使用。
**方式二：通过CocoaPods**
```
pod 'SwiftLint', :configurations => ['Debug']
```
这种方式相当于把SwiftLint作为一个三方库集成进了项目，因为它只是调试工具，所以我们应该将其指定为仅Debug环境下生效。

### 继承进Xcode
我们需要在项目中的`Build Phases`，添加一个`Run Script Phase`。如果是通过homebrew安装的，你的脚本应该是这样的。

```shell
if which swiftlint >/dev/null; then
  swiftlint
else
  echo "warning: SwiftLint not installed, download from https://github.com/realm/SwiftLint"
fi
```

如果是通过cocoapods安装的，你得脚本应该是这样的：

```shell
"${PODS_ROOT}/SwiftLint/swiftlint"
```

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200520101250.png)

### 运行SwiftLint

键入`CMD + B`编译项目，在编译完后会运行我们刚才加入的脚本，之后我们就能看到项目中大片的警告信息。有时候build信息并不能填入项目代码中，我们可以在编译的log日志里查看。

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200520101753.png)

### 定制

SwiftLint规则太多了，如果我们不想执行某一规则，或者想要滤掉对Pods库的分析，我们可以对SwfitLint进行配置。

在项目根目录新建一个`.swiftlint.yml`文件，然后填入如下内容：

```yaml
disabled_rules: # rule identifiers to exclude from running
  - colon
  - trailing_whitespace
  - vertical_whitespace
  - function_body_length
opt_in_rules: # some rules are only opt-in
  - empty_count
  # Find all the available rules by running:
  # swiftlint rules
included: # paths to include during linting. `--path` is ignored if present.
  - Source
excluded: # paths to ignore during linting. Takes precedence over `included`.
  - Carthage
  - Pods
  - Source/ExcludedFolder
  - Source/ExcludedFile.swift
  - Source/*/ExcludedFile.swift # Exclude files with a wildcard
analyzer_rules: # Rules run by `swiftlint analyze` (experimental)
  - explicit_self

# configurable rules can be customized from this configuration file
# binary rules can set their severity level
force_cast: warning # implicitly
force_try:
  severity: warning # explicitly
# rules that have both warning and error levels, can set just the warning level
# implicitly
line_length: 110
# they can set both implicitly with an array
type_body_length:
  - 300 # warning
  - 400 # error
# or they can set both explicitly
file_length:
  warning: 500
  error: 1200
# naming rules can set warnings/errors for min_length and max_length
# additionally they can set excluded names
type_name:
  min_length: 4 # only warning
  max_length: # warning and error
    warning: 40
    error: 50
  excluded: iPhone # excluded via string
  allowed_symbols: ["_"] # these are allowed in type names
identifier_name:
  min_length: # only min_length
    error: 4 # only error
  excluded: # excluded via string array
    - id
    - URL
    - GlobalAPIKey
reporter: "xcode" # reporter type (xcode, json, csv, checkstyle, junit, html, emoji, sonarqube, markdown)
```

一条rules提示如下，其对应的rules名就是`function_body_length`。

```
Function Body Length Violation: Function body should span 40 lines or less excluding comments and whitespace: currently spans 43 lines (function_body_length)
```

`disabled_rules`下填入我们不想遵循的规则。

`excluded`设置我们想跳过检查的目录，Carthage、Pod、SubModule这些一般可以过滤掉。

其他的一些像是文件长度（file_length），类型名长度（type_name），我们可以通过设置具体的数值来调节。

另外SwiftLint也支持自定义规则，我们可以根据自己的需求，定义自己的`rule`。

### 生成报告

如果我们想将此次分析生成一份报告，也是可以的（该命令是通过homebrew安装的swiftlint）：

```bash
# reporter type (xcode, json, csv, checkstyle, junit, html, emoji, sonarqube, markdown)
$ swiftlint lint --reporter html > swiftlint.html
```

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200520141314.png)

## xcodebuild

xcodebuild是xcode内置的编译命令，我们可以用它来编译打包我们的iOS项目，接下来介绍的Infer和OCLint都是基于xcodebuild的编译产物进行分析的，所以有必要简单介绍一下它。

一般编译一个项目，我们需要指定项目名，configuration，scheme，sdk等信息以下是几个简单的命令及说明。

```shell
# 不带pod的项目，target名为TargetName，在Debug下，指定模拟器sdk环境进行编译
xcodebuild -target TargetName -configuration Debug -sdk iphonesimulator
# 带pod的项目，workspace名为TargetName.xcworkspace，在Release下，scheme为TargetName，指定真机环境进行编译。不指定模拟器环境会验证证书
xcodebuild -workspace WorkspaceName.xcworkspace -scheme SchemeName Release
# 清楚项目的编译产物
xcodebuild -workspace WorkspaceName.xcworkspace -scheme SchemeName Release clean
```

**之后对xcodebuild命令的使用都需要将这些参数替换为自己项目的参数。**

## Infer

[Infer](https://infer.liaohuqiu.net/)是Facebook开发的针对C、OC、Java语言的静态分析工具，它同时支持对iOS和Android应用的分析。对于Facebook内部的应用像是 Messenger、Instagram 和其他一些应用均是有它进行静态分析的。它主要检测隐含的问题，主要包括以下几条：

* 资源泄露，内存泄露
* 变量和参数的非空检测
* 循环引用
* 过早的nil操作

暂不支持自定义规则。

### 安装及使用

```bash
$ brew install infer
```


运行infer
```shell
$ cd projectDir
# 跳过对Pods的分析
$ infer run --skip-analysis-in-path Pods -- xcodebuild -workspace "Project.xcworkspace" -scheme "Scheme" -configuration Debug -sdk iphonesimulator
```

我们会得到一个`infer-out`的文件夹，里面是各种代码分析的文件，有txt，json等文件格式，当这样不方便查看，我们可以将其转成html格式：

```bash
$ infer explore --html
```

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200520110053.png)

点击trace，我们会看到该问题代码的上下文。



因为Infer默认是增量编译，只会分析变动的代码，如果我们想整体编译的话，需要clean一下项目：

```shell
$ xcodebuild -workspace "Project.xcworkspace" -scheme "Scheme" -configuration Debug -sdk iphonesimulator clean
```

再次运行Infer去编译。

```bash
$ infer run --skip-analysis-in-path Pods -- xcodebuild -workspace "Project.xcworkspace" -scheme "Scheme" -configuration Debug -sdk iphonesimulator
```

### Infer的大致原理

Infer的静态分析主要分两个阶段：

**1、捕获阶段**

Infer 捕获编译命令，将文件翻译成 Infer 内部的中间语言。

这种翻译和编译类似，Infer 从编译过程获取信息，并进行翻译。这就是我们调用 Infer 时带上一个编译命令的原因了，比如: `infer -- clang -c file.c`, `infer -- javac File.java`。结果就是文件照常编译，同时被 Infer 翻译成中间语言，留作第二阶段处理。特别注意的就是，如果没有文件被编译，那么也没有任何文件会被分析。

Infer 把中间文件存储在结果文件夹中，一般来说，这个文件夹会在运行 `infer` 的目录下创建，命名是 `infer-out/`。

**2、分析阶段**

在分析阶段，Infer 分析 `infer-out/` 下的所有文件。分析时，会单独分析每个方法和函数。

在分析一个函数的时候，如果发现错误，将会停止分析，但这不影响其他函数的继续分析。

所以你在检查问题的时候，修复输出的错误之后，需要继续运行 Infer 进行检查，知道确认所有问题都已经修复。

错误除了会显示在标准输出之外，还会输出到文件 `infer-out/bug.txt` 中，我们过滤这些问题，仅显示最有可能存在的。

在结果文件夹中（`infer-out`），同时还有一个 csv 文件 `report.csv`，这里包含了所有 Infer 产生的信息，包括：错误，警告和信息。



## OCLint

[OCLint](http://oclint.org/)是基于[Clange Tooling](http://clang.llvm.org/docs/LibTooling.html)编写的库，它支持扩展，检测的范围比Infer要大。不光是隐藏bug，一些代码规范性的问题，例如命名和函数复杂度也均在检测范围之内。

#### 安装OCLint

OCLint一般通过Homebrew安装

```bash
$ brew tap oclint/formulae   
$ brew install oclint
```

通过Hombrew安装的版本为0.13。

```bash
$ oclint --version
LLVM (http://llvm.org/):
  LLVM version 5.0.0svn-r313528
  Optimized build.
  Default target: x86_64-apple-darwin19.0.0
  Host CPU: skylake

OCLint (http://oclint.org/):
  OCLint version 0.13.
  Built Sep 18 2017 (08:58:40).
```

我分别用Xcode11在两个项目上运行过OCLint，一个实例项目可以正常运行，另一个复杂的项目却运行失败，报如下错误：

```bash
1 error generated
1 error generated
...
oclint: error: cannot open report output file ..../onlintReport.html
```

我并不清楚原因，如果你想试试0.13能否使用的话，直接跳到安装xcpretty。如果你也遇到了这个问题，可以回来安装oclint0.15版本。

#### OCLint0.15

我在[oclint issuse #547](https://github.com/oclint/oclint/issues/547)这里找到了这个问题和对应的解决方案。

我们需要更新oclint至0.15版本。brew上的最新版本是0.13，github上的最新版本是0.15。我下载github上的release0.15版本，但是这个包并不是编译过的，不清楚是不是官方自己搞错了，只能手动编译了。因为编译要下载llvm和clange，这两个包较大，所以我将编译过后的包直接传到了这里[CodeChecker](https://github.com/zhangferry/CodeChecker)。

如果不关心编译过程，可以下载编译好的包，跳到设置环境变量那一步。

**编译OCLint**

1、安装[CMake](https://cmake.org/)和[Ninja](https://ninja-build.org/)这两个编译工具

```bash
$ brew install cmake ninja
```

2、clone OCLint项目

```bash
$ git clone https://github.com/oclint/oclint
```

3、进入oclint-scripts目录，执行make命令

```bash
$ ./make
```

成功之后会出现build文件夹，里面有个oclint-release就是编译成功的oclint工具。

**设置oclint工具的环境变量**

设置环境变量的目的是为了我们能够快捷访问。然后我们需要配置PATH环境变量，注意OCLint_PATH的路径为你存放oclint-release的路径。将其添加到`.zshrc`，或者`.bash_profile`文件末尾:

```bash
OCLint_PATH=/Users/zhangferry/oclint/build/oclint-release
export PATH=$OCLint_PATH/bin:$PATH
```

执行`source .zshrc`，刷新环境变量，然后验证oclint是否安装成功：

```bash
$ oclint --version
OCLint (http://oclint.org/):
OCLint version 0.15.
Built May 19 2020 (11:48:49).
```

出现这个介绍就说明我们已经完成了安装。

### 安装xcpretty

xcpretty是一个格式化xcodebuild输出内容的脚本工具，oclint的解析依赖于它的输出。它的安装方式为：

```shell
$ gem install xcpretty
```

### OCLint的使用

在使用OCLint之前还需要一些准备工作，需要将编译项`COMPILER_INDEX_STORE_ENABLE`设置为NO。

* 将 Project 和 Targets 中 Building Settings 下的 `COMPILER_INDEX_STORE_ENABLE` 设置为 **NO**
* 在 podfile 中 **target 'target' do 前面**添加下面的脚本，将各个pod的编译配置也改为此选项

```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
      target.build_configurations.each do |config|
          config.build_settings['COMPILER_INDEX_STORE_ENABLE'] = "NO"
      end
  end
end
```

#### 使用方式

1、进入项目根目录，运行如下脚本：

```shell
$ xcodebuild -workspace ProjectName.xcworkspace -scheme ProjectScheme -configuration Debug -sdk iphonesimulator | xcpretty -r json-compilation-database -o compile_commands.json
```

会将xcodebuild编译过程中的一些信息记录成一个文件`compile_commands.json`，如果我们在项目根目录看到了该文件，且里面是有内容的，证明我们完成了第一步。



2、我们将这个json文件转成方便查看的html，过滤掉对Pods文件的分析，为了防止行数上限，我们加上行数的限制：

```shell
$ oclint-json-compilation-database -e Pods -- -report-type html -o oclintReport.html -rc LONG_LINE=9999 -max-priority-1=9999 -max-priority-2=9999 -max-priority-3=9999
```

最终会产生一个`oclintReport.html`文件。

![image-20200519153146276](/Users/zhangferry/Library/Application Support/typora-user-images/image-20200519153146276.png)



OCLint支持自定义规则，因为其本身规则已经很丰富了，自定义规则的需求应该很小，也就没有尝试。



**封装脚本**

OCLint跟Infer一样都是通过运行几个脚本语言进行执行的，我们可以将这几个命令封装成一个脚本文件，以OCLint为例，Infer也类似：

```shell
#!/bin/bash
# mark sure you had install the oclint and xcpretty

# You need to replace these values with your own project configuration
workspace_name="WorkSpaceName.xcworkspace"
scheme_name="SchemeName"

# remove history
rm compile_commands.json
rm oclint_result.xml
# clean project
# -sdk iphonesimulator means run simulator
xcodebuild -workspace $workspace_name -scheme $scheme_name -configuration Debug -sdk iphonesimulator clean || (echo "command failed"; exit 1);

# export compile_commands.json
xcodebuild -workspace $workspace_name -scheme $scheme_name -configuration Debug -sdk iphonesimulator \
| xcpretty -r json-compilation-database -o compile_commands.json \
|| (echo "command failed"; exit 1);

# export report html
# you can run `oclint -help` to see all USAGE
oclint-json-compilation-database -e Pods -- -report-type html -o oclintReport.html \
-disable-rule ShortVariableName \
-rc LONG_LINE=1000 \
-max-priority-1=9999 \
-max-priority-2=9999 \
-max-priority-3=9999 || (echo "command failed"; exit 1);

open -a "/Applications/Safari.app" oclintReport.html
```



`oclint-json-compilation-database`命令的几个参数说明：

-e
 需要忽略分析的文件，这些文件的警告不会出现在报告中

-rc
 需要覆盖的规则的阀值，这里可以自定义项目的阀值，[默认阀值](]http://docs.oclint.org/en/stable/howto/thresholds.html#mccabe76)

-enable-rule
 支持的规则，默认是oclint提供的都支持，可以组合-disable-rule来过滤掉一些规则
 [规则列表](http://docs.oclint.org/en/stable/rules/index.html)

-disable-rule
 需要忽略的规则，根据项目需求设置

#### 在Xcode中使用OCLint

因为OCLint提供了xcode格式的输出样式，所以我们可以将它作为一个脚本放在Xcode中。

1、在项目的 TARGETS 下面，点击下方的 "+" ，选择 cross-platform 下面的 Aggregate。输入名字，这里命名为 OCLint

![../_images/xcode_screenshot_1.png](http://docs.oclint.org/en/stable/_images/xcode_screenshot_1.png)



2、选中该Target，进入Build Phases，添加Run Script，写入下面脚本：

```shell
# Type a script or drag a script file from your workspace to insert its path.
# 内置变量
cd ${SRCROOT}
xcodebuild clean 
xcodebuild | xcpretty -r json-compilation-database
oclint-json-compilation-database -e Pods -- -report-type xcode
```

可以看出该脚本跟上面的脚本一样，只不过 将`oclint-json-compilation-database`命令的`-report-type`由`html`改为了`xcode`。而OCLint作为一个target本身就运行在特定的环境下，所以xcodebuild可以省去配置参数。



3、通过`CMD + B`我们编译一下项目，执行脚本任务，会得到能够定位到代码的warning信息：

![../_images/xcode_screenshot_8.png](http://docs.oclint.org/en/stable/_images/xcode_screenshot_8.png)



## 总结

以下是对这几种静态分析方案的对比，我们可以根据需求选择适合自己的静态分析方案。

|                 | 支持语言           | Infer                      | OCLint               |
| --------------- | ------------------ | -------------------------- | -------------------- |
| 支持语言        | Swift              | C、C++、OC、Java           | C、C++、OC           |
| 易用性          | 简单               | 较简单                     | 带编译过程的话较复杂 |
| 能否集成进Xcode | 可以               | 不能集成进xcode            | 可以                 |
| 自带规则多少    | 较多，包含代码规范 | 相对较少，主要检测潜在问题 | 较多，包含代码规范   |
| 能够扩展规则    | 可以               | 不可以                     | 可以                 |



## 参考

[OCLint 实现 Code Review - 给你的代码提提质量](https://juejin.im/post/5ce9f477f265da1b7c60f4fe#heading-1)

[Using OCLint in Xcode](http://docs.oclint.org/en/stable/guide/xcode.html)

[Infer 的工作机制](https://infer.liaohuqiu.net/docs/infer-workflow.html)

[LLVM & Clang 入门]([https://github.com/CYBoys/Blogs/blob/master/LLVM_Clang/LLVM%20%26%20Clang%20%E5%85%A5%E9%97%A8.md](https://github.com/CYBoys/Blogs/blob/master/LLVM_Clang/LLVM %26 Clang 入门.md))

