

## 打包形式差别

常见的两种静态库打包形式：.framework和.a，它们两者有如下区别：

|          | framework                                   | .a                               |
| -------- | ------------------------------------------- | -------------------------------- |
| 资源文件 | 包含在framework里                           | .a只包含编译后的代码，不包含资源 |
| 代码文件 | 包含在framework里，为编译后的代码           | .a包含编译后的代码               |
| 动态性   | 系统framework均为动态，个人创建的均为静态库 | 静态库                           |

通常说我们个人创建的framework也是静态的，这个是相对系统framework来说的。个人创建的framework是分动态和静态两种的，这个可以在Build Setting里的Mach-O Type进行设置。

这里的动态是一种伪动态，它不可以在不同应用中共享，仅可以在应用和附属扩展（Extension）之间共享。

如果设置为静态，二进制包也会包含到主项目的执行文件中，这时主应用和Extension是不可以共享的。


我们可以通过Podfile里的`use_frameworks!`控制pod使用framework形式打包还是使用.a形式打包。

如果是使用framework形式打包，可以在pod的project里控制各个库是使用静态framework还是动态framework。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20201111225120.png)



## CocoaPods工作流程

有了以上知识