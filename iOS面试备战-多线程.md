![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200719183858.png)

iOS面试中多线程绝对是最重要的知识点之一，它在日常开发中会被广泛使用，而且多线程是有很多区分度很高的题目可供考察的。这篇文章会梳理下多线程和GCD相关的概念和几个典型问题。因为GCD相关的API用OC看着更直管一些，所以这期实例就都用OC语言书写。

## 概念篇

在面对一些我们常见的概念时，我们常有种这个东西我熟的感觉，但是如果没有深入研究它们的概念和区别，还是很容易弄混或者讲不清楚的。所以这里单独抽一节讲下多线程中的概念。

### 进程，线程，任务，队列

进程：资源分配的最小单位。在iOS中一个应用的启动就是开启了一个进程。

线程：CPU调度的最小单位。一个进程里会有多个线程。

大家可以思考下，进程和线程为什么是从资源分配和CPU调度层面进行定义的。

任务：每次执行的一段代码，比如下载一张图片，触发一个网络请求。

队列：队列是用来组织任务的，一个队列包含多个任务。

### GCD

GCD（Grand Central Dispatch）是异步执行任务的技术之一。开发者只需要定义想执行的任务并追加到适当的Dispatch Queue中，GCD就能生成必要的线程执行该任务。这里的线程管理是由系统处理的，我们不必关心线程的创建销毁，这大大方便了我们的开发效率。也可以说GCD是一种简化线程操作的多线程使用技术方案。

安卓没有跟GCD完全相同的一套技术方案的，虽然它可以处理GCD实现的一系列效果。

### 串行，并行，并发

GCD的使用都是通过调度队列（Dispatch Queue）的形式进行的，调度队列有以下 几种形式：

**串行（serial）**：多任务中某时刻只能有一个任务被运行；

**并行（parallel）**：相对于串行，某时刻有多个任务同时被执行，需要多核能力；

**并发（concurrent）**：引入时间片和抢占之后才有了并发的说法，某个时间片只有一个任务在执行，执行完时间片后进行资源抢占，到下一个任务去执行，即“微观串行，宏观并发”，所以这种情况下只有一个空闲的某核，多核空闲就又可以实现并行运行了；

我们常用的调度队列有以下几种：

```objective-c
// 串行队列
dispatch_queue_t serialQueue = dispatch_queue_create("com.gcd.serialQueue", DISPATCH_QUEUE_SERIAL);
// 并发队列
dispatch_queue_t concurrentQueue = dispatch_queue_create("com.gcd.concurrentQueue", DISPATCH_QUEUE_CONCURRENT);
// 全局并发队列
dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
// 主队列
let mainQueue = DispatchQueue.main
```
注意GCD创建的是并发队列而不是并行队列。但这里的并发队列是一个相对宽泛的定义，它包含并行的概念，GCD作为一个智能的中心调度系统会根据系统情况判断当前能否使用多核能力分摊多个任务，如果满足的话此时就是在并行的执行队列中的任务。

### 同步，异步

 **同步**：函数会阻塞当前线程直到任务完成返回才能进行其它操作；

 **异步**：在任务执行完成之前先将函数值返回，不会阻塞当前线程；



### 串行、并发和同步、异步相互结合能否开启新线程

|      | 串行队列     | 并发队列     | 主队列       |
| ---- | ------------ | ------------ | ------------ |
| 同步 | 不开启新线程 | 不开启新线程 | 不开启新线程 |
| 异步 | 开启新线程   | 开启新线程   | 不开启新线程 |



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

**队列与线程的关系**

队列是对任务的描述，它可以包含多个任务，这是应用层的一种描述。线程是系统级的调度单位，它是更底层的描述。一个队列（并行队列）的多个任务可能会被分配到多个线程执行。



## 问题

### 代码分析

**1、分析下面代码的执行逻辑**

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    [self syncMainTask];
}

- (void)syncMainTask {
    dispatch_queue_main_t mainQueue = dispatch_get_main_queue();
    dispatch_sync(mainQueue, ^{
        NSLog(@"main queue task");
    });
}
```

这段代码会输出`task1`，然后发生死锁，导致crash。

**追加问题一：为什么会死锁？死锁就会导致crash？**

我们先分析crash的情况，正常死锁应该就是卡死的情况，不应该导致carsh。那为什么会carsh呢，看崩溃信息：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200706154838.png)

是一个`EXC_BAD_INSTRUCTION`类型的crash，执行了一个出错的命令。

然后看`__DISPATCH_WAIT_FOR_QUEUE__`的调用栈信息：

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200706155152.png)

右侧汇编代码给出了更详细的crash信息：`BUG IN CLIENT OF LIBDISPATCH: dispatch_sync called on queue already owned by current thread`。

在当前线程已经拥有的队列中执行`dispatch_sync`同步操作会导致crash。

在`libdispatch`的源码中我们可以找到该函数的定义：

```c
DISPATCH_NOINLINE
static void
__DISPATCH_WAIT_FOR_QUEUE__(dispatch_sync_context_t dsc, dispatch_queue_t dq)
{
	uint64_t dq_state = _dispatch_wait_prepare(dq);
	if (unlikely(_dq_state_drain_locked_by(dq_state, dsc->dsc_waiter))) {
		DISPATCH_CLIENT_CRASH((uintptr_t)dq_state,
				"dispatch_sync called on queue "
				"already owned by current thread");
	}
	/*...*/
}
```

所以我们知道了，这个carsh是`libdispatch`内部抛出的，当它检测到可能发生死锁时，就直接触发崩溃，事实上它不能完全判断出所有死锁的情况。

我们分析这里为什么会发生死锁。首先`syncMainTask`就是在主队列中的，我们在主队列先添加`dispatch_sync`然后再添加其内部的block。主队列FIFO，只有sync执行完了才会执行内部的block，而此时是一个同步队列，block执行完才会退出sync，所以导致了死锁。

对于死锁的解释我也查了好几篇文章，有些说法其实是经不起推敲的，这个解释是我认为相对合理的。

附一篇参考文章：[GCD死锁](https://juejin.im/post/5b4d945ef265da0f9c6797f0)

**引出问题二：什么情况下会发生死锁？**

GCD中发生死锁需要满足两个条件：

* 同步执行串行队列
* 执行sync的队列和block所在队列为同一个队列

**引出问题三：如何避免死锁？这段代码应该如何修改？**

根据上面提到的条件，我们可以将任务异步执行，或者换成一个并发队列。另外将block放到一个非主队列里执行也是可以的。



**2、分析一下代码执行结果**

```objective-c
int a = 0;
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
while (a < 2) {
    dispatch_async(queue, ^{
        a++;
    });
}
NSLog(@"a = %d", a);
```

首先该段代码会编译不过，编译器检测到变量`a`被block截获，并尝试修改就报以下错误：

`Variable is not assignable (missing __block type specifier)`。如果我们要在block里对外界变量重新复制，需要添加`__block`的声明：`__block int a = 0;`

我们分析这段代码，在开始while之后加入一个异步任务，再之后呢，这个是不确定了，可能是执行`a++`也可能是因不满足退出条件再次执行加入异步任务，直到满足`a<2`才会退出while循环。那输出结果也就是不确定了，因为可能在判断跳出循环和输出结果的时候另外的线程又执行了一次`a++`。

再扩展下，如果将那个并发队列改成主队列，执行逻辑还是一样的吗？

首先主队列是不会开启新线程的，主队列上的异步操作执行时机是等别的任务都执行完了，再来执行添加的`a++`。显然在while循环里，主队列既有任务还未执行完毕，所以就不会执行`a++`，也就导致while循环不会退出，形成死循环。

## 其它问题

### 什么是线程安全，为什么UI操作必须在主线程执行

线程安全：当多个线程访问某个方法时，不管你通过怎样的调用方式或者说这些线程如何交替的执行，我们在主程序中不需要去做任何的同步，这个类的结果行为都是我们设想的正确行为，那么我们就可以说这个类时线程安全的。

为什么UI操作必须放到主线程：首先UIKit不是线程安全的，多线程访问会导致UI效果不可预期，所以我们不能使用多个线程去处理UI。那既然要单线程处理UI为什么是在主线程呢，这是因为UIApplication作为程序的起点是在主线程初始化的，所以我们后续的UI操作也都要放到主线程处理。

关于这个问题展开讨论可以参阅这篇文章：[iOS拾遗——为什么必须在主线程操作UI](https://juejin.im/post/5c406d97e51d4552475fe178)

###开启新的线程有哪些方法 

1、NSThread

2、NSOperationQueue

3、GCD

4、NSObject的`performSelectorInBackground`方法

5、pthread

### 多线程任务要实现顺序执行有哪些方法

1、dispatch_group

2、dispatch_barrier

3、dispatch_semaphore_t

4、NSOperation的addDependency方法

### 如何实现一个多读单写的功能？

多读单写的意思就是可以有多个线程同时参与读取数据，但是写数据时不能有读操作的参与切只有一个线程在写数据。

我们写一个示例程序，看下在不做限制的多读多写程序中会发生什么。

```objective-c
// 计数器
self.count = 0;
// 并发队列
self.concurrentQueue = dispatch_get_global_queue(0, 0);
for (int i = 0; i< 10; i++) {
    dispatch_async(self.concurrentQueue, ^{
        [self read];
    });
    dispatch_async(self.concurrentQueue, ^{
        [self write];
    });
}
// 读写操作
- (void)read {
    NSLog(@"read---- %d", self.count);
}

- (void)write {
    self.count += 1;
    NSLog(@"write---- %d", self.count);
}

// 输出内容
2020-07-18 11:47:03.612175+0800 GCD_OC[76121:1709312] read---- 0
2020-07-18 11:47:03.612273+0800 GCD_OC[76121:1709311] read---- 1
2020-07-18 11:47:03.612230+0800 GCD_OC[76121:1709314] write---- 1
2020-07-18 11:47:03.612866+0800 GCD_OC[76121:1709312] write---- 2
2020-07-18 11:47:03.612986+0800 GCD_OC[76121:1709311] write---- 3
2020-07-18 11:47:03.612919+0800 GCD_OC[76121:1709314] read---- 2
2020-07-18 11:47:03.613252+0800 GCD_OC[76121:1709312] read---- 3
2020-07-18 11:47:03.613346+0800 GCD_OC[76121:1709314] write---- 4
2020-07-18 11:47:03.613423+0800 GCD_OC[76121:1709311] read---- 4
```

每次运行的输出结果都会不一样，根据这个输出内容，我们可以看到在还没有执行到输出write----1的时候，就已经执行了read----1，在write---- 3之后 read的结果却是2。这绝对是我们所不期望的。其实在程序设计中我们是不应该设计出多读多写这种行为，因为这个结果是不可控。

解决方案之一是对读写操作都加上锁做成单独单写，这样是没问题但有些浪费性能，正常写操作确定之后结果就确定了，读的操作可以多线程同时进行，而不需要等别的线程读完它才能读，所以有了多读单写的需求。

解决多读单写常见有两种方案，第一种是使用读写锁`pthread_rwlock_t`。

读写锁具有一些几个特性：

* 同一时间，只能有一个线程进行写的操作
* 同一时间，允许有多个线程进行读的操作。
* 同一时间，不允许既有写的操作，又有读的操作。

这跟我们的多读单写需求完美吻合，也可以说读写锁的设计就是为了实现这一需求的。它的实现方式如下：

```objective-c
// 执行读写操作之前需要定义一个读写锁
@property (nonatomic,assign) pthread_rwlock_t lock;
pthread_rwlock_init(&_lock,NULL);
// 读写操作
- (void)read {
    pthread_rwlock_rdlock(&_lock);
    NSLog(@"read---- %d", self.count);
    pthread_rwlock_unlock(&_lock);
}

- (void)write {
    pthread_rwlock_wrlock(&_lock);
    _count += 1;
    NSLog(@"write---- %d", self.count);
    pthread_rwlock_unlock(&_lock);
}
// 输出内容
2020-07-18 12:00:29.363875+0800 GCD_OC[77172:1722472] read---- 0
2020-07-18 12:00:29.363875+0800 GCD_OC[77172:1722471] read---- 0
2020-07-18 12:00:29.364195+0800 GCD_OC[77172:1722469] write---- 1
2020-07-18 12:00:29.364325+0800 GCD_OC[77172:1722472] write---- 2
2020-07-18 12:00:29.364450+0800 GCD_OC[77172:1722470] read---- 2
2020-07-18 12:00:29.364597+0800 GCD_OC[77172:1722471] write---- 3
2020-07-18 12:00:29.366490+0800 GCD_OC[77172:1722469] read---- 3
2020-07-18 12:00:29.366703+0800 GCD_OC[77172:1722472] write---- 4
2020-07-18 12:00:29.366892+0800 GCD_OC[77172:1722489] read---- 4
```

我们查看输出日志，所以的读操作结果都是最近一次写操作所赋的值，这是符合我们预期的。



还有一种实现多读单写的方案是使用GCD中的栅栏函数`dispatch_barrier`。栅栏函数的目的就是保证在同一队列中它之前的操作全部执行完毕再执行后面的操作。为了保证写操作的互斥行，我们要对写操作执行「栅栏」：

```objective-c
// 我们定义一个用于读写的并发对列
self.rwQueue = dispatch_queue_create("com.rw.queue", DISPATCH_QUEUE_CONCURRENT);

- (void)read {
    dispatch_sync(self.rwQueue, ^{
        NSLog(@"read---- %d", self.count);
    });
}

- (void)write {
    dispatch_barrier_async(self.rwQueue, ^{
        self.count += 1;
        NSLog(@"write---- %d", self.count);
    });
}
```

这个输出结果跟读写锁实现是一样的，也是符合预期的。

这里多说几句，这里的读和写分别使用`sync`和`async`。读操作要用同步是为了阻塞线程尽快返回结果，不用担心无法实现多读，因为我们使用了并发队列，是可以实现多读的。至于写操作使用异步的栅栏函数，是为了写时不阻塞线程，通过栅栏函数实现单写。如果我们将读写都改成sync或者async，由于栅栏函数的机制是会顺序先读后写。如果反过来，读操作异步，写操作同步也是可以达到多读单写的目的的，但读的时候不立即返回结果，网上有人说只能使用异步方式，防止发生死锁，这个说法其实不对，因为同步队列是不会发生死锁的。



### 用GCD如何实现一个控制最大并发数且执行任务FIFO的功能？

这个相对简单，通过信号量实现并发数的控制，通过并发队列实现任务的FIFO的执行

```objective-c
int maxConcurrent = 3;
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
dispatch_semaphore_t semaphore = dispatch_semaphore_create(maxConcurrent);
dispatch_async(queue, ^{
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    // task
    dispatch_semaphore_signal(semaphore);
});
```

![](https://gitee.com/zhangferry/Images/raw/master/gitee/20200719185749.png)