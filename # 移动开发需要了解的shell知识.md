# 移动开发需要了解的shell知识

## 什么是shell

我们常说的shell有两层含义：
* Shell 是一个应用程序，它连接了用户和 Linux 内核，让用户能够更加高效、安全、低成本地使用 Linux 内核，这就是 Shell 的本质。

* Shell是一种解释型语言，我们可以通过它编写脚本。

### 解释型语言和编译型语言
像C/C++，Swift，java，Go等，通过他们编写的程序需要经过编译生成可执行文件，才能运行，我们看不到源码。这类语言就叫编译型语言。
像Shell，Python，JavaScript等，是可以一边执行一边翻译的，不会生成可执行文件，必须拿到源码才能运行程序。这类语言就叫解释型语言。

### Shell环境
既然shell是一种解释型语言，那么是由谁来解释它呢？Mac OS中安装了多种shell解释器：bash，zsh，C shell。在Catalina之前的版本默认为bash，之后的默认解释器就换成了zsh。

### 终端
我们经常输入并运行脚本的地方就是终端，常见的有Terminal，[iTerm2](https://www.iterm2.com/)。shell解释器是运行在终端里面的，我们可以输入：
```shell
$ echo $SHELL
```
我的电脑显示为
```
/bin/zsh
```
说明我当前的shell环境是zsh。很多人的 mac 中使用 zsh 而不是 bash，一大半是因为[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) 这个配置集，它兼容 bash，还有自动补全提示，主题配置等好用的功能。

## shell能做什么
我们开发时虽然直接接触shell编程的机会不多，但是如果我们了解并熟悉shell之后，会很大程度上提升工作效率。shell在iOS开发过程中也是有多种用途的。

## 运行脚本
比如我们要创建一个脚本文件：test.sh。有两种方式可以运行（解释）它：
1、作为可执行程序：
cd到脚本所在目录，执行：
```shell
# 增加test.sh可执行权限
$ chmod +x ./test.sh
$ ./test.sh
```
在修改脚本权限之前可以通过`ls -l`命令查看文件状态变化：
```
# 修改前
$ -rw-r--r--@  1 username  staff    4 Mar 21 12:49 test.sh
# 修改后
$ -rwxr-xr-x@  1 username  staff    4 Mar 21 12:49 test.sh
```
2、作为解释器参数：
```
$ sh test.sh
```

## Shell语法
### 定义变量
```shell
var_name="zhangferry"
```
* 命名规范跟Swift，OC差不多。
* 注意等号左右不能带空格。
* 不需声明变量类型。

### 使用变量
```shell
# 花括号可加可不加
echo $var_name
echo ${var_name}
```

### 字符串
```shell
var_str_0='Hello, I am \"$var_name\"! \n'
var_str_1="Hello, I am \"$var_name\"! \n"
```
输出这两个字符串：
```shell
echo $var_str_0
echo $var_str_1
```
得到结果为：
```
Hello, I am \"$var_name\"!

Hello, I am "zhangferry"!

```
可以看出单引号不会解析变量，而是原封不动打印展示出来，但是会解析`\n`换行符。
双引号会解析变量和换行符。

### 字符串运算符

下表列出了常用的字符串运算符，假定变量 a 为 "abc"，变量 b 为 "efg"：

| 运算符 | 说明                                         | 举例                     |
| ------ | -------------------------------------------- | ------------------------ |
| =      | 检测两个字符串是否相等，相等返回 true。      | [ $a = $b ] 返回 false。 |
| !=     | 检测两个字符串是否相等，不相等返回 true。    | [ $a != $b ] 返回 true。 |
| -z     | 检测字符串长度是否为0，为0返回 true。        | [ -z $a ] 返回 false。   |
| -n     | 检测字符串长度是否不为 0，不为 0 返回 true。 | [ -n "$a" ] 返回 true。  |
| $      | 检测字符串是否为空，不为空返回 true。        | [ $a ] 返回 true。       |

**注意：**条件表达式要放在方括号之间，并且要有空格，例如: `[$a==$b]` 是错误的，必须写成 `[ $a = $b ]`，方括号里共有四个空格。

### test命令
Shell中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。

数值测试：

| 参数 | 说明           |
| ---- | -------------- |
| -eq  | 等于则为真     |
| -ne  | 不等于则为真   |
| -gt  | 大于则为真     |
| -ge  | 大于等于则为真 |
| -lt  | 小于等于则为真 |
| -le  | 小于则为真     |

**注意**：为什么数值比较通过字符串而不是`<`，`>`符号进行比较是？这是因为，shell中这两个符号是用于重定向的。

使用方式是：
```shell
num1=100
num2=100
if test $[num1] -eq $[num2]
then
    echo '两个数相等！'
else
    echo '两个数不相等！'
fi
```
**注意**：shell的控制流程没有大括号，if条件语句之后是`then`，条件结尾处是`fi`。
运行上面的脚本，输出结果为：两个数不相等！

test命令还可以跟字符串运算符进行搭配使用，判断字符串是否为空，是否相等等操作。

### 传递参数
我们可以在执行 Shell 脚本时，向脚本传递参数，脚本内获取参数的格式为：$n。n 代表一个数字，1 为执行脚本的第一个参数，2 为执行脚本的第二个参数，以此类推……
我们将tesh.sh脚本改为：

```shell
echo "\$0：$0"
echo "\$1：$1"
echo "\$2：$2"
```
执行脚本：
```
sh test.sh 1 2 3
```
可以得到：
```
$0：test.sh
$1：1
$2：2
```
注意多个参数通过空格分开，`$0`为当前脚本文件，`$1`开始为第一个参数，`$2`为第二个参数，以此类推。

如果我们要实现这样一个需求，当有参数传入时，使用该参数，当无参数时，使用默认参数。

## shell中的操作符

### find



### 

## iOS打包脚本

我们可以用学到的shell知识，完善一下iOS打包脚本，我们需要的功能如下。

* 可以选择打包的版本是Debug还是Release
* 显示打包时长
```shell
## 将脚本放到跟项目同级的目录中
## 跳转到脚本父级目录
## shell允许多条命令通过`;`隔开写在一行执行
PROJECT_PATH=$(cd `dirname $0`; pwd)
## 项目和Scheme名
PROJECT_NAME=ProjectName
SCHEME_NAME=SchemeName
## Archive路径
BUILD_PATH=${PROJECT_PATH}/Archive
## exportOptions.plist路径
EXPORTLIST_PATH=${PROJECT_PATH}/exportOptions.plist

## 如果没有传入参数，默认Debug
if test -z $1
then
    BUILD_MODE="Debug"
else
    BUILD_MODE="Release"
fi
echo ">>>current build mode: ${BUILD_MODE}"

start_time=`date +%s`

# 将打出的包根据Mode放入到不同的文件中
EXPORT_IPA_PATH=${PROJECT_PATH}/IAP/{BUILD_MODE}

echo ">>>>clean project"
xcodebuild clean -configuration ${BUILD_MODE} -quiet || exit

echo ">>>build project: ${BUILD_MODE}"
xcodebuild \
archive -workspace ${PROJECT_PATH}/${PROJECT_NAME}.xcworkspace \
-scheme ${SCHEME_NAME} \
-configuration ${BUILD_MODE} \
-archivePath ${BUILD_PATH}/${PROJECT_NAME}_${BUILD_MODE}.xcarchive -quiet || exit

echo ">>>export archive"
xcodebuild -exportArchive -archivePath ${BUILD_PATH}/${PROJECT_NAME}_${BUILD_MODE}.xcarchive \
-configuration ${BUILD_MODE} \
-exportPath ${EXPORT_IPA_PATH} \
-exportOptionsPlist ${EXPORTLIST_PATH} \
-quiet || exit

end_time=`date +%s`
## 计算打包时间
spent_time=`expr ${end_time} - ${start_time}`
spent_time=`expr ${spent_time} / 60`
echo ">>>export ${PROJECT_NAME} iap success!"
echo ">>>spent ${spent_time} mins"
```