# 《Objective-C 高级编程》
## Grand Central Dispatch （GCD）
#### 开发者要做的只是定义想执行的任务并追加到适当的 Dispatch Queue 中。
#### dispatch_retain and dispatch_release is no longer needed in iOS6 with ARC.
### QOS 优先级 (priority)
1. QOS_CLASS_USER_INTERACTIVE
2. QOS_CLASS_USER_INITIATED
3. QOS_CLASS_UTILITY
4. QOS_CLASS_BACKGROUND

都是后台并行队列，不需要销毁，类型为qos_class_t

### 使用多线程编程，在执行长时间的处理时仍可保证用户界面的响应性能。

### Dispatch Queue
Dispatch Queue 按照追加的顺序（FIFO）执行处理  
串行(Serial)队列：等待现在队列中执行处理结束  
并行(Concurrent)队列：不等待当前队列中执行处理结束，将队列中的任务分配到多个线程中  
  
同步(Synchronized)：阻塞当前线程，等待 Block 中任务执行完毕后继续执行当前线程  
异步(Asynchronized)：不会阻塞当前线程，将 Block 中任务分配到其他线程中  
  
    let blk0 = { print(0) }
    let blk1 = { print(1) }
    let blk2 = { print(2) }
    
    dispatch_async(queue, blk0)
    dispatch_async(queue, blk1)
    dispatch_async(queue, blk2)
    
    dispatch_sync(queue, blk0)
    dispatch_sync(queue, blk1)
    dispatch_sync(queue, blk2)
    
|               | 同步           | 异步  |
|:-------------: |:-------------:| :-----:|
| 串行队列       | 顺序执行        | 其他线程中顺序执行 |
| 并行队列       | 顺序执行       |   多线程随机执行 |

#### 为什么同步执行并行队列会顺序执行？
主线程是一个串行队列，在主线程中将任务同步分配到并行队列中时，每分配一个都会阻塞主线程，任务完成后，主线程恢复，继续分配任务，所以会顺序执行。即使是分配到不同的并行队列中结果也相同。  

### dispatch_set_target_queue(dispatch_object_t!, dispatch_queue_t!)
指定$0的优先级与$1相同。  
Tips: 将任务分配到多个不同的串行队列中，同一队列中的任务顺序执行，而多个队列之间是并行的。但若把多个不同的串行队列设定同一个**串行队列**作为目标时，则这些串行队列间也变成串行执行，如同一个串行队列。  

### dispatch_after

    let time = dispatch_time(DISPATCH_TIME_NOW, Int64(10 * NSEC_PER_SEC))
    
    dispatch_after(time, queue, blk0)
    
在 time 时间后将指定的 block 追加到指定的队列中(async)  
并不是在指定时间后处理 block，而是在指定时间后将 block 追加到指定的队列中，若队列为串行队列且其中仍有未执行的任务，则 block 需要排队等待执行。  


### Dispatch Group
    let group = dispatch_group_create()
    
    dispatch_group_async(group, queue1, blk0)
    dispatch_group_async(group, queue2, blk1)
    dispatch_group_async(group, queue3, blk2)
    
    dispatch_group_notify(group, anyQueue, finishBlock)
    
将会等待追加到`group`中所有`queue`中任务执行结束后执行`finishBlock`。  
    
    let timeout = dispatch_time(DISPATCH_TIME_NOW, Int64(10 * NSEC_PER_SEC))
    
    dispatch_group_wait(group, timeout)
    
`dispatch_group_wait`的第二个参数为等待时间，等待时间后判断`group`中队列是否执行完毕，执行完毕返回0，否则返回1。

### dispatch_barrier_async
    dispatch_async(queue, blk0)
    dispatch_async(queue, blk1)
    dispatch_async(queue, blk2)
    dispatch_async(queue, blk3)
    dispatch_async(queue, blk4)
    dispatch_barrier_async(queue, blk5)
    dispatch_async(queue, blk6)
    dispatch_async(queue, blk7)
    dispatch_async(queue, blk8)
    dispatch_async(queue, blk9)
    
在一个并行队列中，使用`dispatch_barrier_async`追加的任务一定在队列中所有已有任务执行完毕后执行，一定在后追加的任务前执行。  
使用`dispatch_barrier_async`追加到并行队列中的所有任务**串行执行**。  
Example: 在数据库的读写中，需要串行执行写操作，同时并行执行读操作，且写操作不能与任何其他操作同时进行。即可使用`dispatch_async`追加读操作，使用`dispatch_barrier_async`追加写操作。 

### dispatch_apply
    dispatch_apply(iterations: Int, queue, block: (Int) -> Void)
    
同步的(sync)将`block`追加到`queue`中`iterations`次。`block.$0`代表block的序列。  
 但是`iterations`次追加是一次性追加到`queue`中的，如果`queue`是一个并行队列，则这些`block`将并行执行。当这些`block`执行完毕后恢复当前线程。

    dispatch_apply(array.count, queue) { (index) in print(a[index]) }
    
这样可以在`queue`中处理`array`中所有元素。

### dispatch_suspend & dispatch_resume
挂起/恢复指定的队列。  
对任何队列挂起后如果没有恢复将导致运行错误。  
向挂起的队列中追加任务有效，将在队列恢复后执行。