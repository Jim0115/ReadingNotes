### 第29条：理解引用计数
OC使用引用计数来管理内存，也就是说，每个对象都有个可以递增或递减的计数器。如果像使某个对象继续存活，那就递增其引用计数；用完了之后，就递减其计数。计数变为0，就表示没人关注此对象了，于是，就可以把它销毁。  
从 OS X 10.8 开始，“垃圾收集器”（GC，garbage collector）已经正式废弃了，而iOS则从未支持过垃圾收集。

#### 引用计数工作原理
在引用计数架构下，对象有个计数器，用以表示当前有多少个事物想令此对象继续存活下去。这在OC中叫做“保留计数”（retain count），可以叫“引用计数”（reference count）。`NSObject`声明了下列三个方法用于操作计数器，以递增或递减其值：

* `retain` 递增引用计数
* `release` 递减引用计数
* `autorelease` 待稍后清理“自动释放池”（autorelease pool）时，再递减保留计数

查看引用计数的方法叫做`retainCount`，此方法不太有用，即使再调试时也如此。  
对象创建出来时，其引用计数至少为1。若想令其继续存活，则调用`retain`方法。若是某部分代码不再使用此对象，不想令其继续存活，那就调用`release`和`autorelease`方法。最终当保留计数归零时，对象就回收了（deallocated），也就是说，系统会将其所占用的内存标记为“可重用”（reuse）。此时，所有指向该对象的引用也都变得无效了。  
对象如果持有指向其它对象的强引用（strong reference），那么前者就“拥有”（own）了后者。也就是说，对象想令其引用的那些对象继续存活，就可以将其“保留”。等用完了之后再释放。  
如果按“引用计数”回溯，那么最终会发现一个“根对象”（root object）。在macOS app中，此对象就是`NSApplication`对象；而在iOS app中，则是`UIApplication`对象。两者都是应用程序启动时创建的单例。

    NSNumber* number = @10;
    [array addObject:number];
    [number release];
    NSLog(@"number = %@", number);
即使上述代码在本例中可以正常执行，也依然不是个好办法。如果调用`release`之后，基于某些原因，其保留计数降至0，那么`number`对象所占内存也许会回收，这样的话，再调用`NSLog`可能就将使程序崩溃了。在这里说“可能”，而没说“一定”，因为对象所占的内存在“解除分配”（deallocated）之后，只是放回“可用内存池”（available pool）。如果执行`NSLog`时尚未复写对象内存，那么该对象仍然有效，此时程序不会崩溃。  
为避免在不经意间使用了无效对象，一般调用完`release`之后都会清空指针。这就能保证不会出现可能指向无效对象的指针，这种指针通常称为“悬挂指针”（dangling pointer）。比方说，可以这样编写代码来防止此情况发生：

    NSNumber* number = @10;
    [array addObject:number];
    [number release];
    number = nil;