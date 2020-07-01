iOS面试中对于多线程的考察是必须的。多线程在开发中会被频繁的用到。这篇文章会梳理下多线程和GCD相关的概念和问题。

我认为一套合适的iOS面试题，多线程相关内容是必须要问的。



## 概念篇

在面对一些我们常见的概念时，我们常有种这个东西我熟的感觉，但是如果没有深入研究它们的概念和区别，还是很容易弄混或者讲不清楚的。所以这里单独抽一节讲下多线程中的概念。

### 进程，线程，任务，队列

进程：资源分配的最小单位。在iOS中一个应用的启动就是开启了一个进程。
线程：CPU调度的最小单位。一个进程里会有多个线程。

任务：每次执行的一段代码，比如下载一张图片，触发一个网络请求。

队列：队列是任务一个个任务的合集。

### 同步，异步

 **同步**：函数会阻塞直到任务完成返回才能进行其它操作；

```swift
let serialQueue = DispatchQueue.init(label: "SERIAL")
main.sync {
    print("serial sync")
}
print("end")
//> serial sync
//> end
```

 **异步**：在任务执行完成之前先将函数值返回，不会阻塞当前线程；

```swift
let serialQueue = DispatchQueue.init(label: "SERIAL")
main.async {
    print("serial async")
}
print("end")
//> end
//> serial async
```



### 串行，并行，并发

**串行（serial）**：多任务中某时刻只能有一个任务被运行；

**并行（parallel）**：相对于串行，某时刻有多个任务同时被执行，需要多核能力；

**并发（concurrent）**：引入时间片和抢占之后才有了并发的说法，某个时间片只有一个任务在执行，执行完时间片后进行资源抢占，到下一个任务去执行，即“微观串行，宏观并发”，所以这种情况下只有一个空闲的某核，多核空闲就又可以实现并行运行了；

我们常用的创建队列的方法有以下两种，分别对应串行队列和并发队列：

```swift
// 串行队列
let serialQueue = DispatchQueue.init(label: "SERIAL")
// 并发队列
let concurrentQueue = DispatchQueue.global()
// 主队列
let mainQueue = DispatchQueue.main
```

注意没有直接创建并行队列的方法，那GCD有没有运用多核的能力呢？是有的，只不是这个处理能力是隐式的，当我们使用并发队列时，注意这里不是严格的并发队列，GCD内部会根据CPU当前的状态自动选择时单核执行还是多核执行。



### 串行并发和同步异步相互结合

|      | 串行         | 并发         |
| ---- | ------------ | ------------ |
| 同步 | 不开启新线程 | 不开启新线程 |
| 异步 | 开启新线程   | 开启新线程   |



## 问题

### 代码分析

**1、该段代码有什么问题**

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    dispatch_queue_t mainQueue = dispatch_get_main_queue();
    dispatch_sync(mainQueue, ^{
        NSLog(@"task1");
    });
    NSLog(@"finished");
}
```

大部分人能答出来会carsh，因为阻塞了主线程。那为什么会阻塞呢？





在对GCD相关代码执行顺序分析时我们需要明白：

1、主线程有一个特点：主线程会先执行主线程上的代码片段，然后才会去执行放在主队列中的任务。

2、同步执行  dispatch_sync函数的特点：该函数只有在该函数中被添加到某队列的某方法执行完毕之后才会返回。即 方法会等待 task 执行完再返回

```swift
func taks2() {
    concurrentQueue.async {
        self.main.sync {
            print("task2>1")
        }
        print("task2>2")
    }
    sleep(1)
    print("task2>3")
    // 3 > 1 > 2
}

func taks3() {
    concurrentQueue.async {
        self.mainQueue.async {
            print("task3>1")
        }
        print("task3>2")
    }
    sleep(1)
    print("task3>3")
    // 2 > 3 > 1
}

func task4() {
    var a = 0
    while a < 5 {
        DispatchQueue.main.async {
            a += 1
        }
    }
    print(a)
}
```



## 其它问题

###开启新的线程有哪些方法 



#### 用GCD如何实现一个多读单写的功能？



#### 用GCD如何实现一个一定并发数且按顺序执行的功能？