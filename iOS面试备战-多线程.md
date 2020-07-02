iOS面试中对于多线程的考察是必须的。多线程在开发中会被频繁的用到。这篇文章会梳理下多线程和GCD相关的概念和问题。

我认为一套合适的iOS面试题，多线程相关内容是必须要问的。因为有些GCD的API用OC看着更舒服一些，所以这期实例就都用OC语言了。



## 概念篇

在面对一些我们常见的概念时，我们常有种这个东西我熟的感觉，但是如果没有深入研究它们的概念和区别，还是很容易弄混或者讲不清楚的。所以这里单独抽一节讲下多线程中的概念。

### 进程，线程，任务，队列

进程：资源分配的最小单位。在iOS中一个应用的启动就是开启了一个进程。
线程：CPU调度的最小单位。一个进程里会有多个线程。

任务：每次执行的一段代码，比如下载一张图片，触发一个网络请求。

队列：队列时用来组织任务的，我们将任务添加到队列中，系统会根据资源决定是否创建新的线程去处理队列中的任务。

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

### GCD



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



### 主线程和主队列

主线程是一个线程，主队列是指主线程上的任务组织形式。

主队列只会在主线程执行，但主线程上执行的不一定就是主队列，还有可能是别的同步队列。因为前说过，同步操作不会开辟新的线程，所以当你自定义一个同步的串行或者并行队列时都是还在主线程执行。

判断当前是否是主线程：

```objective-c
BOOL isMainThread = [NSThread isMainThread];
```

判断当前是否在主队列上：

```objective-c
static void *mainQueueKey = "mainQueueKey";
dispatch_queue_set_specific(dispatch_get_main_queue(), mainQueueKey, &mainQueueKey, NULL);
BOOL isMainQueue = dispatch_get_specific(mainQueueKey));
```

## 问题

### 代码分析

**1、该段代码会输出什么**

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
  	NSLog(@"task1");
    dispatch_queue_t mainQueue = dispatch_get_main_queue();
    dispatch_sync(mainQueue, ^{
        NSLog(@"task2");
    });
    NSLog(@"task3");
}
```

这段代码会输出`task1`，然后发生死锁，导致crash。

追加问题一：为什么会死锁？

分析崩溃原因还能看出来，是`EXC_BAD_INSTRUCTION`类型的crash。跳到`__DISPATCH_WAIT_FOR_QUEUE__ ()`函数的汇编界面，我们还能看到出错信息：`BUG IN CLIENT OF LIBDISPATCH: dispatch_sync called on queue already owned by current thread`。

在当前线程的队列中执行同步操作会引起死锁。我们知道死锁是两个线程或者两个任务之间相互等待引起的，那这个案例中是谁跟谁相互等待了呢？网上关于这个问题确有很多误导人的答案：task2和task3相互等待，sync和task2相互等待，都是不对的分析。

正确的理解应该是执行到dispatch_sync时，同步操作阻塞当前线程，需要执行block中的内容，然后才能执行后面的操作。又因为这时在主队列上

引出问题二：什么情况下会发生死锁？

死锁发生的原因是两个任务或者两个线程之间相互等待。那就代表上面的代码发生了两个任务相互等待的情况。

* 这是一个同步队列，同步会阻塞当前线程，直到完成里面的任务`task1`才会返回，然后执行后面的代码。任务`finished`等待任务`task1`的完成。
* 这是在主队列插入代码，队列遵循FIFO。

引出问题三：如何避免死锁？这段代码应该如何修改？



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