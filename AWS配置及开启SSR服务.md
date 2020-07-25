# AWS实例配置及开启SSR服务

这个教程的大致步骤为：注册亚马逊服务（首年免费） > 配置实例 > 启动实例 > 配置服务器的SSR服务。

## 注册亚马逊

1、前往亚马逊 http://aws.amazon.com/cn/，注册面向国外的服务。

2、填写个人资料，绑定信用卡，接收验证码。

## 创建EC2实例

1、进入控制中心，选择机房地点为亚太地区（香港）：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200706115902.png)

选亚太区别的地方也行，因为香港最近，网速会好一些。如果遇到香港地区默认关闭的情况，则需要先打开该地区的服务，然后等5分钟左右会生效，再去选择香港。

2、启动一个EC2实例

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200706120349.png)

3、选择一个Ubuntu镜像服务

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200706120614.png)

4、选择实例类型

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200706120817.png)

仅有`t3.micro`这个是符合条件的免费套餐，然后点击下一步。

5、一直点下一步，直到添加标签选项。

设置秘钥和其对应的值，我这边设置的是：username：zhangferry

6、配置安全组可以先跳过，后面再进行设置。

7、启动审核，设置秘钥。

* 创建新的秘钥对
* 设置秘钥对名称，就是上一步设置过的
* 下载秘钥对，记得保存好该秘钥
* 点击启动实例

## 启动实例

1、点击实例，可以看到我们刚才创建的实例

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200706122436.png)

2、点击连接，会弹出连接实例的方法介绍

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200706122622.png)

通过终端输入上述ssh命令时，会有一个确认的提示，我们输入yes即可建立与远程服务器的连接。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200706123105.png)

3、设置安全组，用于建立访问的规则

点击此处进入安全组的查看界面：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200706123342.png)

点这里，进入入站规则的配置：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200706123525.png)

按照下面设置配置入站规则：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200706123259.png)

4、设置动态IP

点击动态ip选项，然后点击分配弹性ip地址，即可分配给我们一个共有IPv4地址。

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200706130411.png)

## 开启SSR服务

1、一般服务器会自动安装python环境，查看当前服务器python版本

```shell
ubuntu@ip-172-31-15-183:~$ python --version

Command 'python' not found, did you mean:

  command 'python3' from deb python3
  command 'python' from deb python-is-python3
  
ubuntu@ip-172-31-15-183:~$ python3 --version
Python 3.8.2
```

服务器python版本为3.8.2。

2、安装python-pip

因为服务器python版本为3.8.2，所以我们要安装的其实是python3-pip。

```shell
ubuntu@ip-172-31-15-183:~$ sudo apt-get update && sudo apt-get install python3-pip
```

查看pip是否安装成功：

```shell
ubuntu@ip-172-31-15-183:~$ pip3 --version
pip 20.0.2 from /usr/lib/python3/dist-packages/pip (python 3.8)
```

出现对应的版本号，说明我们安装成功了。

3、通过pip3安装shadowsocks

```shell
ubuntu@ip-172-31-15-183:~$ sudo pip3 install shadowsocks
```

4、配置shadowsocks服务

我们需要先生成一个shadowsocks的配置文件：

```shell
ubuntu@ip-172-31-15-183:~$ sudo touch /etc/shadowsocks.json
```

然后编辑该文件：

```shell
ubuntu@ip-172-31-15-183:~$ sudo vi /etc/shadowsocks.json
```

在vim中录入如下内容：

```
{
  "server":"0.0.0.0",
  "local_address":"127.0.0.1",
  "local_port":1080,
  "port_password":{
     "8000":"password_0",
     "8001":"password_1",
     "8002":"password_2",
     "8003":"password_3",
     "8004":"password_4"
  },
  "timeout":300,
  "method":"aes-256-cfb",
  "fast_open": false
}
```

其中`port_pasword`为我们需要访问的端口号和密码，可以配置多个。注意我们前面配置的端口号范围（8000-10000），要那个范围之内。

5、开启shadowsocks服务

```shell
ubuntu@ip-172-31-15-183:~$ sudo ssserver -c /etc/shadowsocks.json -d start
```

可能我们会遇到如下问题：

```
INFO: loading config from /etc/shadowsocks.json
2020-07-05 15:36:28 INFO     loading libcrypto from libcrypto.so.1.1
Traceback (most recent call last):
  File "/usr/local/bin/ssserver", line 8, in <module>
    sys.exit(main())
  File "/usr/local/lib/python3.8/dist-packages/shadowsocks/server.py", line 34, in main
    config = shell.get_config(False)
  File "/usr/local/lib/python3.8/dist-packages/shadowsocks/shell.py", line 262, in get_config
    check_config(config, is_local)
  File "/usr/local/lib/python3.8/dist-packages/shadowsocks/shell.py", line 124, in check_config
    encrypt.try_cipher(config['password'], config['method'])
  File "/usr/local/lib/python3.8/dist-packages/shadowsocks/encrypt.py", line 44, in try_cipher
    Encryptor(key, method)
  File "/usr/local/lib/python3.8/dist-packages/shadowsocks/encrypt.py", line 82, in __init__
    self.cipher = self.get_cipher(key, method, 1,
  File "/usr/local/lib/python3.8/dist-packages/shadowsocks/encrypt.py", line 109, in get_cipher
    return m[2](method, key, iv, op)
  File "/usr/local/lib/python3.8/dist-packages/shadowsocks/crypto/openssl.py", line 76, in __init__
    load_openssl()
  File "/usr/local/lib/python3.8/dist-packages/shadowsocks/crypto/openssl.py", line 52, in load_openssl
    libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,)
  File "/usr/lib/python3.8/ctypes/__init__.py", line 386, in __getattr__
    func = self.__getitem__(name)
  File "/usr/lib/python3.8/ctypes/__init__.py", line 391, in __getitem__
    func = self._FuncPtr((name_or_ordinal, self))
AttributeError: /lib/x86_64-linux-gnu/libcrypto.so.1.1: undefined symbol: EVP_CIPHER_CTX_cleanup
```

这是因为在openssl1.1.0版本中，废弃了`EVP_CIPHER_CTX_cleanup`函数，新版中使用的函数叫`EVP_CIPHER_CTX_reset`。我们需要将openssl中的旧函数进行一个替换。

首先用vim打开对应的文件：

```shell
ubuntu@ip-172-31-15-183:~$ vim /usr/local/lib/python3.8/dist-packages/shadowsocks/crypto/openssl.py
```

注意这里python的版本号是3.8，这里需要跟服务器的python版本号一致。

如果能打开说明我们找对了地方，在编辑之前我们还需要设置下该文件的可写权限：

```shell
ubuntu@ip-172-31-15-183:~$ chmod 666 /usr/local/lib/python3.8/dist-packages/shadowsocks/crypto/openssl.py
```

这下我们可以进行编辑了，为了方便定位，我们可以打开vim的行号显示：

```
:set nu
```

然后把52行和111行的的`EVP_CIPHER_CTX_cleanup`函数替换成`EVP_CIPHER_CTX_reset`。保存并退出。

再次尝试开启shadowsocks服务。

```shell
ubuntu@ip-172-31-15-183:~$ sudo ssserver -c /etc/shadowsocks.json -d start
```

没有错误信息就说明我们开启成功了。

如果需要关闭的话执行下面的命令进行关闭：

```shell
ubuntu@ip-172-31-15-183:~$ sudo ssserver -c /etc/shadowsocks.json -d stop
```



5、配置shadowsocks客户端

打开客户端，填入我们服务器的ip地址（共有IPv4地址），端口号，加密方式和密码（这几项为shadowsocks.json内容）即可。