随着项目的扩大，依靠人工codereview来保证项目的质量，越来越不现实，这时就有必要借助于一种自动化的代码审查工具：程序静态分析。

程序静态分析（Program Static Analysis）是指在不运行代码的方式下，通过词法分析、语法分析、控制流、数据流分析等技术对程序代码进行扫描，验证代码是否满足规范性、安全性、可靠性、可维护性等指标的一种代码分析技术。（来自百度百科）

词法分析，语法分析等工作是由编译器进行的，所以对iOS项目为了完成静态分析，我们需要借助于编译器。

大致思路：。。。。

以下会对各个静态分析工具的使用和安装做一个介绍。

## SwiftLint
对于Swift项目的静态分析可以使用[SwiftLint](https://github.com/realm/SwiftLint)。SwiftLint 是一个用于强制检查 Swift 代码风格和规定的一个工具。它的实现是 Hook 了 [Clang](http://clang.llvm.org/) 和 [SourceKit](http://www.jpsim.com/uncovering-sourcekit) 从而能够使用 [AST](http://clang.llvm.org/docs/IntroductionToTheClangAST.html) 来表示源代码文件的更多精确结果。Clange我们了解了，那SourceKit是干什么用的？

SourceKit是一套工具，它支持Swift的大多数源代码操作特性：源代码解析、语法突出显示、排版、自动完成、跨语言头生成，等等。

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

### 使用
我们需要在项目中的`Build Phases`，添加一个`Run Script Phase`：

如果是通过homebrew安装的，你的脚本应该是这样的。

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

最终效果如下，注意SwiftLint的脚本要放到编译步骤的下面，因为它是基于编译结果进行分析的。

![image-20200519205356286](/Users/zhangferry/Library/Application Support/typora-user-images/image-20200519205356286.png)

### 测试

键入`CMD + B`编译项目，会运行我们刚才加入的脚本，之后我们就能看到项目中大片的警告信息。



### 定制

有时候我们会需要自己指定一些规则，可以在项目根目录新建一个`.swiftlint.yml`文件，自定义lint行为，然后填入如下内容：

```yaml
disabled_rules: # rule identifiers to exclude from running
  - colon
  - comma
  - control_statement
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

我们可以通过`disabled_rules`设置不想执行的规则，SwiftLint规则太多了，这个真的好用。可以用`excluded`设置我们想跳过检查的目录，Carthage、Pod、SubModule这些可以过滤掉。其他的一些像是文件长度，函数体的长度，我们可以通过设置具体的数值来调节。

### xcodebuild

xcodebuild是xcode内置的编译命令，我们可以用它来编译我们的iOS项目，接下来介绍的Infer和OCLint也都是基于xcodebuild的编译产物进行分析的。

它的基本用法如下：

```shell
# 不带pod的项目
xcodebuild -target <target name> -configuration <build configuration> -sdk iphonesimulator
# 带pod的项目
xcodebuild -workspace <xcworkspace name> -scheme <scheme>
```

一般编译时我们还会指定编译环境，这个参数时：`-configuration <build configuration>`，configuration一般有Debug和Release两个系统提供的值。还有参数是指定模拟器环境：`-sdk iphonesimulator`，如果不加就默认指定到真机。

这样下来一个完整的build命令就是这个样子：

```shell
xcodebuild -workspace "Project.xcworkspace" -scheme "Scheme" -configuration Debug -sdk iphonesimulator
```

## Infer

[Infer](https://infer.liaohuqiu.net/)是针对C、OC、Java语言的静态分析工具，意味着它可以同时支持对iOS和Android应用的分析。

### 安装

```shell
brew install infer
```


运行infer
```shell
$ cd projectDir
$ infer run --skip-analysis-in-path Pods -- xcodebuild -workspace "Project.xcworkspace" -scheme "Scheme" -configuration Debug -sdk iphonesimulator
```

我们会得到一个`infer-out`的文件夹，里面是各种代码分析的文件，这样不方便查看，我们可以将其转成html格式：

```
$ infer explore --html
```

因为Infer默认是增量编译，只会分析变动的代码，如果我们想整体编译的话，需要clean一下项目：

```shell
$ xcodebuild -workspace "Project.xcworkspace" -scheme "Scheme" -configuration Debug -sdk iphonesimulator clean
```

再次运行Infer去编译。





1、捕获阶段

Infer 捕获编译命令，将文件翻译成 Infer 内部的中间语言。

这种翻译和编译类似，Infer 从编译过程获取信息，并进行翻译。这就是我们调用 Infer 时带上一个编译命令的原因了，比如: `infer -- clang -c file.c`, `infer -- javac File.java`。结果就是文件照常编译，同时被 Infer 翻译成中间语言，留作第二阶段处理。特别注意的就是，如果没有文件被编译，那么也没有任何文件会被分析。

Infer 把中间文件存储在结果文件夹中，一般来说，这个文件夹会在运行 `infer` 的目录下创建，命名是 `infer-out/`。

2、分析阶段

在分析阶段，Infer 分析 `infer-out/` 下的所有文件。分析时，会单独分析每个方法和函数。

在分析一个函数的时候，如果发现错误，将会停止分析，但这不影响其他函数的继续分析。

所以你在检查问题的时候，修复输出的错误之后，需要继续运行 Infer 进行检查，知道确认所有问题都已经修复。

错误除了会显示在标准输出之外，还会输出到文件 `infer-out/bug.txt` 中，我们过滤这些问题，仅显示最有可能存在的。

在结果文件夹中（`infer-out`），同时还有一个 csv 文件 `report.csv`，这里包含了所有 Infer 产生的信息，包括：错误，警告和信息。



## OCLint

http://oclint.org/



```
➜  pbn_cn_ios git:(ferry) ✗ oclint --version
LLVM (http://llvm.org/):
  LLVM version 5.0.0svn-r313528
  Optimized build.
  Default target: x86_64-apple-darwin19.0.0
  Host CPU: skylake

OCLint (http://oclint.org/):
  OCLint version 0.13.
  Built Sep 18 2017 (08:58:40).
```

![image-20200519103647312](/Users/zhangferry/Library/Application Support/typora-user-images/image-20200519103647312.png)



从brew安装OCLint是0.13版本的，在Xcode11中运行会报错。但是源码版本已经是0.15了，解决了这个问题，对应的[issuse#547](https://github.com/oclint/oclint/issues/547)。



所以我们需要手动编译OCLint 0.15版本：

编译OCLint

安装CMake和Ninja两个编译工具

```
brew install cmake ninja
```

clone OCLint项目

```
git clone https://github.com/oclint/oclint
```

进入oclint-scripts目录，执行make命令

```
./make
```

成功之后会出现build文件夹：

然后我们需要配置PATH环境变量，将其添加到`.zshrc`文件，或者`.bash_profile`中:

```
OCLint_PATH=/Users/zhangferry/oclint/build/oclint-release
export PATH=$OCLint_PATH/bin:$PATH
```

注意OCLint_PATH的路径为你存放oclint-release的路径。

执行`source .zshrc`，刷新环境变量，然后验证oclint是否安装成功：

```
➜  ~ oclint --version
OCLint (http://oclint.org/):
OCLint version 0.15.
Built May 19 2020 (11:48:49).
```

0.15版本证明我们已经完成了安装。

**安装xcpretty**

用于对xcodebuild的输出进行格式化

```shell
$ gem install xcpretty
```

xcpretty是一个格式化xcodebuild输出内容的脚本工具，oclint的解析依赖于它的输出。

**OCLint的使用**

在使用OCLint之前还需要一些准备工作

* 将 Project 和 Targets 中 Building Settings 下的 COMPILER_INDEX_STORE_ENABLE 设置为 **NO**
* 在 podfile 中 target 'xx' do 前面添加下面的脚本，将各个pod的编译配置也改为此选项

```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
      target.build_configurations.each do |config|
          config.build_settings['COMPILER_INDEX_STORE_ENABLE'] = "NO"
      end
  end
end
```



进入项目根目录，运行如下脚本：

​```shell
$ xcodebuild -workspace ProjectName.xcworkspace -scheme ProjectScheme -configuration Debug -sdk iphonesimulator | xcpretty -r json-compilation-database -o compile_commands.json
```

会将xcodebuild编译过程中的一些信息记录成一个文件`compile_commands.json`。

然后我们将这个json文件转成方便查看的html，为了防止警告的限制，我们加上行数的限制，最终如下：

```shell
$ oclint-json-compilation-database -e Pods -- -report-type html -o oclintReport.html -rc LONG_LINE=9999 -max-priority-1=9999 -max-priority-2=9999 -max-priority-3=9999
```

最终会产生一个`oclintReport.html`文件。

![image-20200519153146276](/Users/zhangferry/Library/Application Support/typora-user-images/image-20200519153146276.png)



这几个命令可以封装成一个脚本文件，我做了些精简：

```shell
#!/bin/bash
# mark sure you had install the oclint and xcpretty

# You need to replace these values with your own project configuration
workspace_name="WorkSpaceName.xcworkspace"
scheme_name="SchemeName"

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



在Xcode中使用OCLint

* 在项目的 TARGETS 下面，点击下方的 "+" ，选择 cross-platform 下面的 Aggregate。输入名字，这里命名为 OCLint

  ![../_images/xcode_screenshot_1.png](http://docs.oclint.org/en/stable/_images/xcode_screenshot_1.png)

* 选中该Target，进入Build Phases，添加Run Script，写入下面脚本：

  ```shell
  # Type a script or drag a script file from your workspace to insert its path.
  # 内置变量
  cd ${SRCROOT}
  xcodebuild clean 
  xcodebuild | xcpretty -r json-compilation-database
  oclint-json-compilation-database -e Pods -- -report-type xcode
  ```

可以看出该脚本跟上面的脚本一样，只不过 将oclint-json-compilation-database命令的-report-type由html改为了xcode。而OCLint运行在特定的环境下，xcodebuild不需要指定具体target即可。

其实我们还可以将刚才写的脚本直接拖到这里来。

通过`CMD + B`我们编译一下项目，执行脚本任务，会得到能够定位到代码的warning信息：

![../_images/xcode_screenshot_8.png](http://docs.oclint.org/en/stable/_images/xcode_screenshot_8.png)



## 参考

[OCLint 实现 Code Review - 给你的代码提提质量](https://juejin.im/post/5ce9f477f265da1b7c60f4fe#heading-1)

[SwiftSyntax详解](https://juejin.im/post/5dac6d3ef265da5b741514b0)

