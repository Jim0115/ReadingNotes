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
不过，ARC可以在运行期检测到这一对多余的操作，也就是`autorelease`及紧跟其后的`retain`。为了优化代码，在方法中返回自动释放的对象时，要执行一个特殊函数。此时不直接调用对象的`autorelease`方法，而是改为调用`objc_autoreleaseReturnValue`。此函数会检视当前方法返回后要执行的那段代码。若发现那段代码上要在返回的对象上执行`retain`操作，则设置全局数据结构中的一个标志位，而不执行`autorelease`操作。如果方法返回了一个自动释放的对象，而调用方法的代码要保留此对象，那么不直接执行`retain`，而是改为执行`objc_retainAutoreleasedReturnValue`函数。此函数要检测刚才提到的标志位，若已经置位，则不执行`retain`操作。设置并检测标志位，要比调用`autorelease`和`retain`更快。

#### 变量的内存管理语义
ARC也会处理局部变量与实例变量的内存管理。默认情况下，每个变量都是指向对象的强引用。

    @interface EOCClass : NSObject {
      id _object;
    }
    
    @implementation EOCClass 
    - (void) setup {
      _object = [EOCOtherClass new];
    }
    @end
    
在MRC时，实例变量并不会自动保留其值，而在ARC环境下则会这样做。也就是说，在ARC下，`setup`方法会变成：

    - (void)setup {
      id tmp = [EOCOtherClass new];
      _object = [tmp retain];
      [_tmp release];
    }
    
在此情况下，`retain`和`release`可以消去。所以，ARC会将这两个操作化简掉，实际执行的代码还是和原来一样。不过，在编写setter时，使用ARC会简单一些。如果不用ARC，那么需要像下面这样写：

    - (void)setObject:(id)object {
      [object retain];
      [_object release];
      _object = object;
    }
    
在应用程序中，可用下列修饰符来改变局部变量与实例变量的语义：

* `__strong`：默认语义，保留此值。
* `__unsafe_unretained`：不保留此值，这样做可能不安全，因为等到再次使用变量时，其对象可能已经回收了
* `__weak`：不保留此值，但是变量可以安全使用，如果系统回收了这个对象，变量会自动置为nil
* `__autoreleasing`：把对象“按引用传递”（pass by reference）给方法时，使用这个特殊的修饰符。此值在方法返回时自动释放。

#### ARC如何清理实例变量
ARC也负责对实例变量进行内存管理。要清理其内存，ARC就必须在dealloc时生成必要的清理代码（cleanup code）。凡是具备强引用的变量，都必须释放，ARC会在dealloc方法中插入这些代码。在MRC时，使用以下的dealloc方法：

    - (void)dealloc {
      [_foo release];
      [_bar release];
      [super dealloc];
    }
在ARC下，不需要这样写。但是，如果有非OC的对象，比如`CoreFoundation`中的对象或是由`malloc()`分配在堆中的内存，仍然需要清理。然而不需要调用父类的`dealloc`方法。

    - (void)dealloc {
      CFRelease(_coreFoundationObject);
      free(_heapAllocatedMemoryBlob);
    }
    
### 第31条：在`dealloc`方法中只释放引用并解除监听
对象在经历其生命周期后，最终会被系统所回收，此时就要执行`dealloc`方法了。在每个对象的生命周期里，此方法仅执行**一次**，也就是引用计数降为0的时候。然而具体何时执行是无法保证的。也可以理解为：我们能够通过人工观察`retain`和`release`的位置，来预估此方法何时执行。但实际上，程序库会以开发者察觉不到的方式操作对象，从而使回收对象的时机和预期的不同。绝对应该自己调用`dealloc`方法。运行期系统会在适当的时候调用它。  
在`dealloc`方法中做些什么呢？主要就是释放对象所拥有的引用，也就是把所有OC对象都释放掉，ARC会通过自动生成的方法自动添加这些释放代码。对象拥有的其他非OC的对象也要释放，比如`CoreFoundation`对象就必须手动释放。  
在`dealloc`方法中，通常还要做一件事，就是把原来配置过的观测行为（observation behavior）都清理掉。如果用`NSNotificationCenter`给此对象订阅（register）过某种通知，那么一般应该在这里注销（unregister）。  
`dealloc`方法可以这样写：

    - (void)dealloc {
      CFRelease(coreFoundationObject);
      [[NSNotificationCenter defalutCenter] removeObserver:self];
    }
    
如果使用MRC，要将当前对象拥有的OC对象逐个释放，最后还需要调用`[super dealloc]`。ARC会自动执行此操作。  
虽说应该于`dealloc`中释放引用，但开销较大或系统稀缺的资源则不在此列。像是文件描述符，socket，大块内存等。不能指望`dealloc`方法必定会在某个特定的时期调用，因为有一些无法预料的东西可能也持有此对象。在这种情况下，如果非要等到系统调用`dealloc`方法时再释放，那么保留这些资源的时间就有些过长了。通常的做法是，实现另一个方法，当应用程序用完资源后，就调用此方法。

### 第32条：编写“异常安全代码”时留意内存管理问题
纯C中没有异常，而cpp和OC都支持异常。实际上，在当前的运行期系统中，二者的异常相互兼容，即，从其中一门语言中抛出的异常可以由另一门语言的“异常处理程序”（exception handler）来捕获。  
捕获异常时，要注意将`try`块内创立的对象清理干净。

### 第33条：以弱引用避免保留环
经常出现的情况是，几个对象以某种方式互相引用，从而形成环（cycle）。由于OC使用引用计数架构，这种情况通常会泄漏内存，因为最后没有别的对象会引用环中的对象。这样的话，环里的对象就无法为外界访问了。蛋对象之间尚有引用，这些引用使它们都能继续存活，而不会被系统回收。  
最简单的保留环由两个对象组成，它们互相引用对方。  
避免保留环出现的最佳方式就是弱引用。这种引用通常用来表示“非拥有关系”（nonowning relationshiip）。将属性声明为`unsafe_unretained`即可。  
属性attribute中的`unsafe_unretained`表明，属性值可能不安全，而且不归此实例拥有，如果系统回收了属性所指的对象，在其上调用方法会导致崩溃。由于本对象并不保留属性对象，因此其有可能被系统回收。  
用`unsafe_unretained`修饰的属性，其语义与`assign`等价。然而，`assign`通常只用于“整体类型”（int，float，结构体等），`unsafe_unretained`则多用于对象类型。这个词本身就表明其所修饰的属性可能无法安全使用（unsafe）。  
OC中还有一项与ARC相伴的特性，`weak`属性，它与`unsafe_unretained`的作用完全相同。然而，只要系统把属性回收，属性值就会自动设为nil。  
一般来说，如果不拥有某对象，那就不要保留它。这条规则对collection例外，collection虽然不直接拥有其内容，但是它要代表自己所属的那个对象来保留这些元素

### 以“自动释放池块”降低内存峰值
OC对象的生命期取决于其引用计数。在OC的引用计数架构中，有一项特性叫做“自动释放池”（autorelease pool）。释放对象有两种方式：一种是调用`release`方法，使其保留计数立即递减；另一种是调用`autorelease`方法，将其加入“自动释放池”中。自动释放池用于存放那些需要在稍后某时刻释放的对象。清空（drain）释放池时，系统会向其中的对象发送`release`消息。  
创建自动释放池的语法如下：

    @autoreleasepool {
      // ...
    }
    
一般情况下不用担心自动释放池的创建问题。 macOS和iOS应用程序分别运行于Cocoa和Cocoa Touch环境中。系统会自动创建一些线程，比如说主线程是GCD机制中的线程，这些线程默认都有自动释放池，每次执行“事件循环”（event loop）时，就会将其清空。因此，不需要自己来创建“自动释放池块”。通常只有一个地方需要创建自动释放池，那就是在`main`函数里。  
从技术角度看，不是非得有个“自动释放池块”才行。因为块的末尾恰好就是应用程序的终止，此时会释放所有内存。虽然如此，如果不写这个块，那么由`UIApplecationMain`函数所自动释放的那些对象，就没有自动释放池可以容纳了，此时系统会发出警告。这个池可以理解为最外围捕捉全部自动释放对象所用的池。  

    for (int i = 0; i < 100000; i++) {
      [self doSomethingWithInt:i];
    }
    
如果`doSomethingWithInt:`方法要创建临时对象，那么这些对象很可能在自动释放池里。比如说，可能是一些临时字符串。但是，即使这些对象在调用完方法之后就不再使用了，它们也依然处于存活状态，因为目前还在自动释放池里，等待系统稍后将其释放并回收。然而，autoreleasepool要等待线程执行下一次runloop时才会清空。这就意味着，在for循环执行时会持续创建新对象，并加入自动释放池中。所有这些对象都要等到for循环执行结束才会释放。这样一来，在执行for循环时，应用程序所占内存就会持续上涨。等到所有临时对象都释放后，内存用量又会突然下降。  
这种情况不甚理想，尤其当循环长度无法预知，必须取决于用户输入时更是如此。比如说，要从数据库里读取需对对象许多对象。代码可能会这么写：

    NSArray* databaseRecords = /* ... */;
    NSMutabelArray* people = [NSMutableArray new];
    for (NSDictionary* record in databaseRecords) {
      EOCPerson* person = [[EOCPerson alloc] initWithRecord:record];
      [people addObject:person];
    }
    
若记录中有很多条，则内存中也会有很多不必要的临时对象，它们本来应该提前回收的。如果把循环内的代码包裹在“自动释放池块”中，那么在循环中自动释放的对象就回放在这个池，而不是线程的主池里面。

    NSArray* databaseRecords = /* ... */;
    NSMutabelArray* people = [NSMutableArray new];
    for (NSDictionary* record in databaseRecords) {
      @autoreleasepool {
        EOCPerson* person = [[EOCPerson alloc] initWithRecord:record];
        [people addObject:person];
      }
    }