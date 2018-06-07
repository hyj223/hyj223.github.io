---
layout:     post
title:      "dispatch_group_t VS nsoperationqueue"
description:   "2种实现方式"
date:       2015-12-07
tags:
    - iOS
    - 多线程

---

### 这是一个需求
> 假设有3个操作，需要其中一个操作需要在另外两个操作之后执行~  

**第一种方式**  

```
    dispatch_group_t group = dispatch_group_create();
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    
    dispatch_group_async(group, queue, ^{
        //block 1
        NSLog(@"block 1 start~");
        [NSThread sleepForTimeInterval:1.0];
        NSLog(@"block 1 end~");
    });
    
    dispatch_group_async(group, queue, ^{
        //block 2
        NSLog(@"block 2 start~");
        [NSThread sleepForTimeInterval:2.0];
        NSLog(@"block 2 end~");
    });
    
    
    dispatch_group_notify(group, queue, ^{
        //block 3
        NSLog(@"block 3~");
        self.view.backgroundColor = [UIColor whiteColor];
    });
```  

**第二种方式**  

```
    NSOperationQueue *operationQueue = [[NSOperationQueue alloc] init];
    NSBlockOperation *blockOperation1 = [NSBlockOperation blockOperationWithBlock:^{
        //code~
        NSLog(@"block 1 start~");
        sleep(2);
        NSLog(@"block 1 end~");
    }];
    NSBlockOperation *blockOperation2 = [NSBlockOperation blockOperationWithBlock:^{
        //code~
        NSLog(@"block 2 start~");
        sleep(1);
        NSLog(@"block 2 end~");
    }];
    
    NSBlockOperation *blockOperation3 = [NSBlockOperation blockOperationWithBlock:^{
        //code~
        NSLog(@"block 3");
    }];

    [blockOperation3 addDependency:blockOperation1];
    [blockOperation3 addDependency:blockOperation2];
    
    [operationQueue addOperation:blockOperation1];
    [operationQueue addOperation:blockOperation2];
    [operationQueue addOperation:blockOperation3];

```  

**thoughts**  

    * 如果需要对线程进行细颗粒度的处理，那么还是NSOperation &NSOperationQueue更具有优势~  
        像优先级，并行操作数，取消操作等~
    * 简单操作，GCD的方式更加简洁优雅~    


