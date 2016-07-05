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
    
#### 属性存取方法中的内存管理
如前所述，对象图由相互关联的对象所构成。不光是数组，其它对象也可以保留别的对象，这一般通过访问“属性”来实现，而访问属性时，会用到相关实例变量的获取方法及设置方法。若属性为“strong关系”，则设置的属性值会保留。比如说，有个名为`foo`的属性由名为`_foo`的实例变量所实现，那么，该属性的设置方法会是这样：

    - (void)setFoo:(id)foo {
      [foo retain];
      [_foo release];
      _foo = foo;
    }
    
此方法将保留新值并释放旧值，然后更新实例变量，令其指向新值。顺序很重要。假如还未保留新值就先把旧值释放了，而且两个值又指向同一个对象，那么，先执行的`release`操作可能导致系统将此对象永久回收。而后续的`retain`操作则无法令这个已经彻底回收的对象复生，于是实例变量就成了悬挂指针。

#### 自动释放池
在OC的引用计数架构中，自动释放池是一项重要特性。调用`release`会立刻递减对象的保留计数（而且还有可能令系统回收此对象），然而有时候可以不调用它，改为调用`autorelease`，此方法会在稍后递减计数，通常是在下一次“事件循环”（event loop）时递减，不过也可以执行的更早些（参见第34条）。  
此特性很有用，尤其是在方法返回对象时更应该使用它。在这种情况下，我们并不总是想令方法调用者手工保留其值。比如说：

    - (NSString *)stringValue {
      NSString* str = [[NSString alloc] initWithFormat:@"I an this: %@", self];
      return str;    
    }
此时返回的`str`对象其保留计数比期望值要多1，因为调用alloc会令保留计数加1，而又没有与之对应的释放操作。保留计数多1，就意味着调用者要负责处理多出来的这一次保留操作。必须设法将其抵消。这并不是说保留计数本身一定为1，它可能大于1，不过那取决于`initWithFormat:`方法内的实现细节。你要考虑的是如何将这一次的保留操作抵消掉。  
但是，不能在方法内释放`str`，否则还没等方法返回，系统就把该对象回收了。这里应该使用`autorelease`，它会在稍后释放对象，从而给调用者留下了足够长的时间，使其可以在需要时保留返回值。换句话说，此方法可以保证对象在跨越“方法调用边界”（method call boundary）后一定存活。实际上，释放操作会在清空最外层的自动释放池（参见第34条）时执行，除非你有自己的释放池，否则这个时机指的就是当前线程的下一次事件循环。

    - (NSString *)stringValue {
      NSString* str = [[NSString alloc] initWithFormat:@"I an this: %@", self];
      return [str autorelease];    
    }
修改之后，返回的`NSString`对象必然存活，所以能够像下面这样使用它：

    NSString* str = [self stringValue];
    NSLog(@"The string is: %@", str);
    
由于返回的`str`对象将于稍后自动释放，所以多出来的那一次保留操作到时自然就会抵消，无须在执行内存管理操作。因为自动释放池中的释放操作等到下一次事件循环才会执行，所以`NSLog`语句在使用`str`对象前不需要手动执行保留操作。但是，假如要持有此对象的话（比如将其设置为实例变量），那就需要保留，并在稍后释放：

    _instanceVariable = [[self stringValue] retain];
    // ...
    [_instanceVariable release];
    
#### 保留环
使用引用计数机制时，经常需要注意的问题就是“保留环”（retain cycle），也就是成环状相互引用的多个对象。这将导致内存泄漏，因为循环中的对象引用计数永远不会降为0。对于循环中的每个对象，都有循环中的另一个对象引用着它。  
在GC环境中，这种情况通常被认定为“孤岛”（island of isolation）。此时，垃圾收集器会把三个对象全部回收走。而在引用计数架构下，通常使用“弱引用”（weak reference，参见第33条）来解决此问题，或是从外界命令循环中的某个对象不再保留另外一个对象。这两种方法都能打破保留环。

### 第30条：以ARC简化引用计数
在引用计数的概念中，需要执行保留和释放操作的地方很容易就能看出来。所以Clang编译器项目带有一个“静态分析器”（static analyzer），用于指明程序里引用计数出问题的地方。假设下面这段代码使用MRC管理：

    if ([self shouldLogMsg]) {
      NSString* message = [[NSString alloc] initWithFormat:@"I am object, %p", self];
      NSLog(@"message = %@", message);
    }
    
此代码有内存泄漏问题，因为if语句末尾并未释放`message`对象。由于在if语句之外无法引用`message`，所以此对象所占的内存泄漏（没有正确释放已经不再使用的内存）了。判定内存是否泄漏所用的规则很简单：调用`NSString`的`alloc`方法所返回的那个`message`对象的保留计数比期望值要多1。然而却没有与之对应的释放操作来抵消。  
自动引用计数所做的事情与其名称相符，就是自动管理引用计数。ARC将自动把代码改写为下列形式：

    if ([self shouldLogMsg]) {
      NSString* message = [[NSString alloc] initWithFormat:@"I am object, %p", self];
      NSLog(@"message = %@", message);
      [message release];  ///< Added by ARC
    }
    
使用ARC时要注意，引用计数实际上还是要执行的，只不过保留和释放操作现在由ARC自动添加。  
实际上，ARC在调用这些方法时，并不会通过普通的OC消息派发机制，而是直接调用其C语言版本。这样做性能更好，因为保留和释放需要频繁执行，所以直接调用底层函数能节省很多CPU周期。比如说，ARC会调用于`retain`等价的底层函数`objc_retain`。

#### 使用ARC中必须遵循的方法命名规则
将内存管理语义在方法名中表示出来已经成为OC的惯例，而ARC将之确立为硬性规定。这些规则简单地体现在方法名上。若方法名以下列词语开头，则其返回的对象归调用者所有：

* alloc
* new
* copy
* mutableCopy

归调用者的意思是：调用上述四种方法的那段代码要负责释放方法所返回的对象。也就是说，这些对象的保留计数是正值，而调用了这四种方法的那段代码要将其中一次保留操作抵消掉。  
若方法名不以上述一个词语开头，则表示其返回的对象并不归调用者所有。在这种情况下，返回的对象会自动释放，所以其值在跨越方法调用边界后依然有效。要想使对象多存活一段时间，必须令调用者保留它才行。  
维系这些规则所需的全部内存管理事宜均由ARC自动处理，其中也包括在将要返回的对象上调用`autorelease`

    + (EOCPerson *)newPerson {
      EOCPerson* person = [[EOCPerson alloc] init];
      return person;
      // new开头 不会被释放
    }
    
    + (EOCPerson *)somePerson {
      EOCPerson* person = [[EOCPerson alloc] init];
      return person;
      // 自动将返回改写为 return [person autorelease];
    }
    
    - (void)doSomething {
      EOCPerson* personOne = [EOCPerson newPerson];
      // ...
      
      EOCPerson* personTwo = [EOCPerson somePerson];
      // ...
      
      // personOne 由当前block持有
      // personOne 不由当前block持有
      // ARC 会自动添加 [personOne release];
    }
    
除了会自动调用`retain`和`release`方法外，使用ARC还有其他好处，它可以执行一些手工操作很难甚至无法完成的优化。例如，在编译期，ARC会把能够相互抵消的`retain、release、autorelease`操作约简。如果发现在同一个对象上多次执行了`retain、release`操作，那么ARC有时可以成对移除这两个操作。  
ARC也包括运行期组件。此时执行的优化很有意义。前面讲到，某些方法在返回对象前，为其执行了`autorelease`操作，而调用方法的代码可能需要将返回的对象保留，比如：

    _myPerson = [EOCPerson personWithName:@"Bob Smith"];
    
调用`personWithName:`方法会返回新的`EOCPerson`对象，而此方法在返回对象之前，为其调用了`autorelease`方法。由于实例变量是个强引用，所以编译器在设置其值的时候还需要执行一次保留操作。因此，前面那段代码与下面这段MRC代码等效：
    
    EOCPerson* tmp = [EOCPerson personWithName:@"Bob Smith"];
    _myPerson = [tmp retain];

此时应该能看出，`personWithName:`方法中的`autorelease`和上段代码中的`retain`都是多余的。为提升性能，可将二者删去。但是，在ARC环境下编译代码时，必须考虑“向后兼容性”（backward compatibility），以兼容那些不使用ARC的代码。其实本来ARC也可以直接舍弃`autorelease`这个概念，并且规定，所有从方法中返回的对象其保留计数都比期望值多1。但是，这样做就破坏了向后兼容性。  
不过，ARC可以在运行期检测到这一对多余的操作 回校考试3