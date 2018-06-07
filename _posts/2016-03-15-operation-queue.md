---
layout:     post
title:      "operation queue 知识梳理"
subtitle:   ""
date:       2016-03-15
author:     "huyujin"
tags:
    - 多线程

---   


## 写在开头  
> 关于多线程这块，很重要~  
> 先理理关于NSOperation & NSOperationQueue相关的知识~


## Operation Queues  

用到NSOperation的例子有很多。网络请求，图像压缩，etc。  

### 基础  

### NSOperation  
![](https://raw.githubusercontent.com/hyj223/hyj223.github.io/master/_posts/images/20160315/1.png)  

在 iOS 开发中，我们可以使用 `NSOperation` 类来封装需要执行的任务。`NSOperation` 本身是一个抽象类，不能直接实例化，因此，如果我们想要使用它来执行具体任务的话，就必须创建自己的子类或者使用系统预定义的两个子类，`NSInvocationOperation` 和 `NSBlockOperation` 。  
	
#### NSInvocationOperation  
	
可以通过一个 object 和 selector 非常方便地创建一个 NSInvocationOperation ，这是一种非常动态和灵活的方式。  
	
Apple提供的关于`NSInvocationOperation `的API如下：  
![](https://raw.githubusercontent.com/hyj223/hyj223.github.io/master/_posts/images/20160315/3.png)   
	
创建一个简单的NSInvocationOperation对象： 

```  
	@implementation MyCustomClass
	- (NSOperation*)taskWithData:(id)data {
	    NSInvocationOperation* theOp = [[NSInvocationOperation alloc] initWithTarget:self
	                    selector:@selector(myTaskMethod:) object:data];
	 
	   return theOp;
	}
	 
	// This is the method that does the actual work of the task.
	- (void)myTaskMethod:(id)data {
	    // Perform the task.
	}
	@end
	
```

#### NSBlockOperation  
	
使用`NSBlockOperation `来并发执行一个或多个 `block` ，只有当一个 `NSBlockOperation` 所关联的所有 `block` 都执行完毕时，这个 `NSBlockOperation` 才算执行完成。
	
Apple提供的关于`NSBlockOperation`的API:  
![](https://raw.githubusercontent.com/hyj223/hyj223.github.io/master/_posts/images/20160315/2.png)  
	
我们可以这样使用：  
	
```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
        sleep(2); 
        NSLog(@"currentThread = %@", [NSThread currentThread]);
    }];
    NSLog(@"operation  start before");
    [operation start];
    NSLog(@"operation  start after");
}

```   
	
> IMPORTANT  
> Block 中的操作与start方法在同一个线程执行，并且是同步执行的。  
	
继续扩展
	
```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"BlockOperation 1 begin");
        sleep(10);  // 加个睡眠模仿耗时操作
        NSLog(@"BlockOperation 1 currentThread = %@", [NSThread currentThread]);
        NSLog(@"BlockOperation 1 mainThread    = %@", [NSThread mainThread]);
        NSLog(@"BlockOperation 1 end");
    }];
    [op addExecutionBlock:^{
        NSLog(@"BlockOperation 2 begin");
        sleep(10);
        NSLog(@"BlockOperation 2 currentThread = %@", [NSThread currentThread]);
        NSLog(@"BlockOperation 2 mainThread    = %@", [NSThread mainThread]);
        NSLog(@"BlockOperation 2 end");
    }];
    [op addExecutionBlock:^{
        NSLog(@"BlockOperation 3 begin");
        sleep(10);
        NSLog(@"BlockOperation 3 currentThread = %@", [NSThread currentThread]);
        NSLog(@"BlockOperation 3 mainThread    = %@", [NSThread mainThread]);
        NSLog(@"BlockOperation 3 end");
    }];
    
    NSLog(@"start before");
    [op start];
    NSLog(@"start after");
}

```  

> IMPORTANT  
> * Block2 和 Block3 中的 currentThread 并不是主线程，而且其中的操作也是异步执行的。  
> * 通过addExecutionBlock添加的操作则是多线程异步操作。
	
#### Custom NSOperation  

我们可以自定义非并发和并发两种不同类型的 NSOperation 子类。  
	
对于一个非并发的 operation ，我们需要做的就只是执行 main 方法中的任务以及能够正常响应取消事件就可以了，其它的复杂工作比如依赖配置、KVO 通知等 NSOperation 类都已经帮我们处理好了。而对于一个并发的 operation ，我们还需要重写 NSOperation 类中的一些现有方法。
	
每一个 operation 都应该至少实现以下两个方法：  
	
	* 一个自定义的初始化方法,
	* main 方法。 

一个自定义的初始化方法来将创建的 operation 置于一个已知的状态，并且重写 main 方法来执行我们的任务。  	
自定义一个简单的 operation:  
	
```
@interface MyNonConcurrentOperation : NSOperation
@property id (strong) myData;
-(id)initWithData:(id)data;
@end
 
@implementation MyNonConcurrentOperation
- (id)initWithData:(id)data {
   if (self = [super init])
      myData = data;
   return self;
}
 
-(void)main {
   @try {
      // Do some work on myData and report the results.
   }
   @catch(...) {
      // Do not rethrow exceptions.
   }
}
@end

```  
	
**Responding to Cancellation Events**    
当一个 `operation` 开始执行后，它会一直执行它的任务直到完成或被取消为止。我们可以在任意时间点取消一个 `operation` ，甚至是在它还未开始执行之前。为了让我们自定义的 `operation` 能够支持取消事件，我们需要在代码中定期地检查 `isCancelled` 方法的返回值，一旦检查到这个方法返回 YES ，我们就需要立即停止执行接下来的任务。  
	
通常来说，当我们自定义一个 operation 类时，我们需要考虑在以下几个关键点检查 isCancelled 方法的返回值：  
		
	* 在真正开始执行任务之前；
	* 至少在每次循环中检查一次，而如果一次循环的时间本身就比较长的话，则需要检查得更加频繁；
	* 在任何相对来说比较容易中止 operation 的地方。  

尽管 `operation` 是支持取消操作的，但却并不是立即取消的，而是在你调用了 `operation` 的 `cancel` 方法之后的下一个 `isCancelled` 的检查点取消的。  
	
```
- (void)main {
   @try {
      BOOL isDone = NO;
 
      while (![self isCancelled] && !isDone) {
          // Do some work and set isDone to YES when finished
      }
   }
   @catch(...) {
      // Do not rethrow exceptions.
   }
}

```  
	
**Configuring Operations for Concurrent Execution**    
如果你想要手动地执行一个 operation ，又想这个 operation 能够异步执行的话，你需要做一些额外的配置来让你的 operation 支持并发执行。   
![](https://raw.githubusercontent.com/hyj223/hyj223.github.io/master/_posts/images/20160315/4.png) 
	
	* start ：必须的，所有并发执行的 operation 都必须要重写这个方法，替换掉 NSOperation 类中的默认实现。start 方法是一个 operation 的起点，我们可以在这里配置任务执行的线程或者一些其它的执行环境。另外，需要特别注意的是，在我们重写的 start 方法中一定不要调用父类的实现；  
	* main ：可选的，通常这个方法就是专门用来实现与该 operation 相关联的任务的。尽管我们可以直接在 start 方法中执行我们的任务，但是用 main 方法来实现我们的任务可以使设置代码和任务代码得到分离，从而使 operation 的结构更清晰；
	* isExecuting & isFinished ：必须的，并发执行的 operation 需要负责配置它们的执行环境，并且向外界客户报告执行环境的状态。因此，一个并发执行的 operation 必须要维护一些状态信息，用来记录它的任务是否正在执行，是否已经完成执行等。此外，当这两个方法所代表的值发生变化时，我们需要生成相应的 KVO 通知，以便外界能够观察到这些状态的变化； 
	* isConcurrent ：必须的，这个方法的返回值用来标识一个 operation 是否是并发的 operation ，我们需要重写这个方法并返回 YES 。  


下面分三部分来描述如何定义一个并发的operation。
	
* Defining a concurrent operation	  

the implementations of the isConcurrent, isExecuting, and isFinished methods for the MyOperation class are relatively straightforward. The isConcurrent method should simply return YES to indicate that this is a concurrent operation. The isExecuting and isFinished methods simply return values stored in instance variables of the class itself.
	
	
```
@interface MyOperation : NSOperation {
    BOOL        executing;
    BOOL        finished;
}
- (void)completeOperation;
@end
 
@implementation MyOperation
- (id)init {
    self = [super init];
    if (self) {
        executing = NO;
        finished = NO;
    }
    return self;
}
 
- (BOOL)isConcurrent {
    return YES;
}
 
- (BOOL)isExecuting {
    return executing;
}
 
- (BOOL)isFinished {
    return finished;
}
@end

``` 
	
* the `start` method of `MyOperaton`  
	
In this case, the method simply starts up a new thread and configures it to call the main method. The method also updates the executing member variable and generates KVO notifications for the isExecuting key path to reflect the change in that value. With its work done, this method then simply returns, leaving the newly detached thread to perform the actual task. 
	
	
```
- (void)start {
	if (self.isCancelled) {
	    [self willChangeValueForKey:@"isFinished"];
	    _finished = YES;
	    [self didChangeValueForKey:@"isFinished"];
	
	    return;
	}
	
	[self willChangeValueForKey:@"isExecuting"];
	
	[NSThread detachNewThreadSelector:@selector(main) toTarget:self withObject:nil];
	_executing = YES;
	
	[self didChangeValueForKey:@"isExecuting"];
}
```  
		
* Updating an operation at completion time  

the main method is the entry point for a new thread. It performs the work associated with the operation object and calls the custom completeOperation method when that work is finally done. The completeOperation method then generates the needed KVO notifications for both the isExecuting and isFinished key paths to reflect the change in state of the operation.  
	  
	
```
- (void)main {
   @try {
 
       // Do the main work of the operation here.
 
       [self completeOperation];
   }
   @catch(...) {
      // Do not rethrow exceptions.
   }
}
 
- (void)completeOperation {
    [self willChangeValueForKey:@"isFinished"];
    [self willChangeValueForKey:@"isExecuting"];
 
    executing = NO;
    finished = YES;
 
    [self didChangeValueForKey:@"isExecuting"];
    [self didChangeValueForKey:@"isFinished"];
}

```  
	
> IMPORTANT   
> Even if an operation is canceled, you should always notify KVO observers that your operation is now finished with its work. When an operation object is dependent on the completion of other operation objects, it monitors the isFinished key path for those objects. Only when all objects report that they are finished does the dependent operation signal that it is ready to run. Failing to generate a finish notification can therefore prevent the execution of other operations in your application.  
	
* Maintaining KVO Compliance  

NSOperation 类的以下 key paths 支持 KVO 通知，我们可以通过观察这些 key paths 非常方便地监听到一个 operation 内部状态的变化：  
	
	* isCancelled
	* isConcurrent
	* isExecuting
	* isFinished
	* isReady
	* dependencies
	* queuePriority
	* completionBlock


### NSOperationQueue  
	
NSOperationQueue的队列类型：  
	
	* 主队列
		* [NSOperationQueue mainQueue]  
		* 凡是添加到主队列中的任务（NSOperation），都会放到主线程中执行
	* 非主队列
		* [[NSOperationQueue alloc] init]
		* 添加到这种队列中的任务（NSOperation），就会自动放到子线程中执行  
		
简单使用NSOperationQueue:  

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // 创建3个 NSInvocationOperation 操作
    NSOperationQueue *opQueue = [NSOperationQueue new];
    for (NSUInteger i = 0; i < 3; i++) {
        // 可以传递一个 NSObject 给operation的操作方法
        NSDictionary *dict = [NSDictionary dictionaryWithObject:[NSString stringWithFormat:@"Operation_%lu", i] forKey:@"key"];
        NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(operationSelector:) object:dict];
        [opQueue addOperation:op];
    }
}
    
// NSInvocationOperation 操作执行的方法
- (void)operationSelector:(NSDictionary *)dict
{
    // 接收传进来的dict
    NSLog(@"dictValue = %@", [dict valueForKey:@"key"]);
    sleep(10);  // 加个睡眠模仿耗时操作
    NSLog(@"currentThread = %@", [NSThread currentThread]);
    NSLog(@"mainThread    = %@", [NSThread mainThread]);
}

```  
	
NSOperationQueue的其他属性：  
	
```
// 直接添加一个 NSOperation 操作，并且加入并发队列，只要当前队列允许，就会立刻执行。
- (void)addOperation:(NSOperation *)op;
// 添加一组操作，如果 waitUntilFinished 为 NO，则必须在当前队列中的所有操作都执行完了，才会执行这组操作，否则立刻执行。
- (void)addOperations:(NSArray<NSOperation *> *)ops waitUntilFinished:(BOOL)wait NS_AVAILABLE(10_6, 4_0);
// 直接在这里写一个block，block中的操作加入并发队列，并且只要当前队列允许执行，就会立刻执行。
- (void)addOperationWithBlock:(void (^)(void))block NS_AVAILABLE(10_6, 4_0);
```

```
// 返回当前队列中的所有操作NSOperation
@property (readonly, copy) NSArray<__kindof NSOperation *> *operations;

// 返回当前队列中的操作数量，对应 operations.count
@property (readonly) NSUInteger operationCount NS_AVAILABLE(10_6, 4_0);

// 可读写的属性，当设备性能不足或根据需求要限制并行的操作数量时，可以设置这个值。
// 设置了这个值之后，队列中并发执行的操作数量不会大于这个值。超出这个值在排队中的操作会处于休眠状态。
// 默认值为 NSOperationQueueDefaultMaxConcurrentOperationCount = -1
@property NSInteger maxConcurrentOperationCount;  

// 可以给队列指定一个名字用来做标识
@property (nullable, copy) NSString *name NS_AVAILABLE(10_6, 4_0);


// 给队列指定一个优先级，默认为 NSQualityOfServiceDefault = -1
@property NSQualityOfService qualityOfService NS_AVAILABLE(10_10, 8_0); 

// 取消队列中的所有操作。其实就是调用 operations 中每个操作的`cancel`方法才取消操作。
// 但是，在前面的文章中说过，调用`cancel`方法并不会终止操作，而是设置`cancelled`属性为 YES，
// 这就需要自己在操作中分节点去判断`cancelled`属性了，在适当的时机结束操作。
- (void)cancelAllOperations;  

	
// 调用这个方法时，会判断 NSOperationQueue 中的操作是否全部执行完，如果没有，则调用者所在的线程会在调用处等待。
// 直到 NSOperationQueue 中的所有操作执行完成，当前线程才继续执行。如果 NSOperationQueue 为空，则该方法立刻返回。
- (void)waitUntilAllOperationsAreFinished;


// 取得调用者的当前线程中的 NSOperationQueue 操作队列
+ (nullable NSOperationQueue *)currentQueue NS_AVAILABLE(10_6, 4_0);


// 取得主线程中的 
+ (NSOperationQueue *)mainQueue NS_AVAILABLE(10_6, 4_0);

```
  

### 深入  

* NSOperation包含了一个十分优雅的状态机来描述每一个操作的执行。    
	> isReady -> isExecuting ->isFinished  
	> * isReady: 返回 YES 表示操作已经准备好被执行, 如果返回NO则说明还有其他没有先前的相关步骤没有完成。  
	> * isExecuting: 返回YES表示操作正在执行，反之则没在执行。  
	> * isFinished : 返回YES表示操作执行成功或者被取消了。  
	

* **并发数**  
即同时执行的任务数。  

```
@property NSInteger maxConcurrentOperationCount;
```

* **队列的取消、暂停、恢复**   
取消队列的所有操作:  
	
```
- (void)cancelAllOperations;
```  
> 提示：也可以调用NSOperation的- (void)cancel方法取消单个操作  
	
暂停和恢复队列:  
	
```
@property (getter=isSuspended) BOOL suspended;
```  
> 提示：暂停一个queue不会导致正在执行的operation在任务中途暂停,只是简单地阻止调度新Operation执行。


* **优先级**  
在operation queue中，系统提供以下优先级。  
	
```
typedef NS_ENUM(NSInteger, NSOperationQueuePriority) {
	NSOperationQueuePriorityVeryLow = -8L,
	NSOperationQueuePriorityLow = -4L,
	NSOperationQueuePriorityNormal = 0,
	NSOperationQueuePriorityHigh = 4,
	NSOperationQueuePriorityVeryHigh = 8
};

```  
* **依赖关系**  
  
根据任务的复杂度，可以把任务拆分成一系列子任务。而在某些情况下，子任务的执行是顺序性的。这部分可以通过给NSOperation添加依赖关系来解决。  
	
比如说，对于服务器下载并压缩一张图片的整个过程，你可能会将这个整个过程分为两个操作（可能你还会用到这个网络子过程再去下载另一张图片，然后用压缩子过程去压缩磁盘上的图片）。显然图片需要等到下载完成之后才能被调整尺寸，所以我们定义网络子操作是压缩子操作的依赖。  
	
```
[resizingOperation addDependency:networkingOperation];
[operationQueue addOperation:networkingOperation];
[operationQueue addOperation:resizingOperation];

```  

### 总结  

尽管GCD对于内嵌异步操作十分理想，NSOperation依旧提供更复杂、面向对象的计算模型，它对于涉及到各种类型数据、需要重复处理的任务又是更加理想的。  

## 最后  

### 参考文章    
<https://developer.apple.com/library/ios/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008091-CH1-SW1>  
<http://nshipster.cn/nsoperation/>  
<http://blog.leichunfeng.com/blog/2015/07/29/ios-concurrency-programming-operation-queues/>  
<http://vinnyxiong.gitcafe.io/blog/nsoperation-and-nsoperationqueue-1.html>  
<https://www.zybuluo.com/chenbinghua/note/181977>  
<http://vinnyxiong.cn/blog/nsoperation-and-nsoperationqueue-3.html>  
<http://www.jianshu.com/p/0b0d9b1f1f19>
