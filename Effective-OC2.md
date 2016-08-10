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
    
### 第35条：用“僵尸对象”调试内存管理问题
向已回收的对象发送消息是不安全的。这么做有时可以，有时不行。具体可行与否，完全取决于对象所占内存有没有为其他内容所覆盖。而这块内存有没有被覆盖，则无法确定，因此，应用程序只是偶尔崩溃。没有崩溃的情况下，那块内存可能只复用了其中一部分，所以其中的某些二进制数据依然有效。另一种可能是，那块内存恰巧被另外一个有效且存活的对象所占据。这种情况下，是否崩溃取决于新对象能否响应selector。  
启用“僵尸对象”（Zombie Object）功能后，运行期系统会把所有已经回收的实例转化为特殊的“僵尸对象”，而不会真正回收它们。这种对象的内存无法重用，因此不可能遭到覆盖。僵尸对象收到消息后，会抛出异常，其中准确说明了发送过来的对象，并描述了回收之前的那个对象。  

### 第36条：不要使用retainCount
OC使用引用计数来管理内存。每个对象都有一个计数器，其值表明还有多少个其他对象想令此对象继续存活。对象创建好之后，其保留计数大于0。保留与释放操作分别会使该计数递增及递减。当计数变为0时，对象就被系统回收并摧毁了。  
此方法无用的原因在于，他所返回的保留计数只是某个给定时间点上的值。该方法并未考虑到系统会稍后把自动释放池清空，因而不会将后续的释放操作从返回值中减去，这样的话，此值就未必能反映实际的保留计数了。  

### 第37条：理解“块”这一概念
“块”block 是一种可在C, C++及OC代码中使用的“词法闭包”（lexical closure），它极为有用，这主要是因为借由此机制，开发者可将代码块像对象一样传递，令其在不同环境（context）下运行。还有个关键的地方是，在定义“块”的范围内，它可以访问到其中的全部变量。  
#### 块的基础知识
块与函数类似，只不过是直接定义在另一个函数里的，和定义它的那个函数共享同一个范围内的东西。block用`^`符号表示，后面跟着一对花括号，括号里是block的实现代码。  

    ^{
      // Block implementation here
    }
block其实就是个值，而且自有其相关类型。与int、float或OC对象一样，也可以把block赋给变量，然后像使用其他变量一样使用它。block类型的语法和函数指针类似。

    void (^someBlock)() = ^{
      // Block implementation here
    };
    
block类型的语法结构如下：

    return_type (^block_name)(parameters)
    
    int (^addBlock)(int, int) = ^(int a, int b) {
      return a + b;
    };
    
    int sum = addBlock(2, 5); // 7
    
block的强大之处是：在声明它的范围内，所有变量都可以为其所捕获。这也就是说，那个范围内的全部变量，在block内依然可用。

    int additional = 5;
    
    int (^addBlock)(int, int) = ^(int a, int b) {
      return a + b + additional;
    };
    
    int sum = addBlock(2, 5); // 12
    
默认情况下，block捕获的变量是不能在block内修改的。不过，声明变量是可以加上`__block`修饰符，就可以在block内修改了。

    NSArray* array = @[@0, @1, @2, @3, @4, @5];
    __block NSInteger count = 0;
    [array enumerateObjectUsingBlock:^(NSNumber* number, NSUInteger idx, BOOL* stop) {
      if ([number compare:@2] == NSOrderedAscending) {
        count++;
      }
    }];
    
    // count = 2
    
这段代码也演示了“内联块”（inline block）的用法。传给`enumerateObjectUsingBlock:`方法的block并未先赋给局部变量，而是直接内联在函数调用里了。由这种常见的编码习惯也可以看出block为何如此有用。在OC引入block这一特性之前，要想实现相同的功能，就必须传入函数指针或selector，以供枚举方法调用。  
如果block捕获的变量是对象类型，那么就会自动retain它。系统在释放这个block的时候，也会将其一并释放。这就引出了一个与block相关的重要问题。block本身也可视为对象。实际上，在其他OC对象能响应的selector中，有很多也是block能响应的。而最重要之处在于，block本身也和其他对象一样，有引用计数。当最后一个指向block的引用移走之后，block就回收了。回收时也会释放block所捕获的变量，以便平衡捕获时所执行的retain操作。  
如果将block定义在OC类的实例方法("-"开头)中，那么出了可以访问到类的所有实例变量之外，还可以使用`self`变量，所以在声明时无需加`__block`。不过，如果通过读取或写入操作捕获了实例变量，那么也会自动把`self`变量一并捕获了，因为实例变量是与`self`所指代的实例关联在一起的。

    @interface EOCClass
    
    - (void)anInstanceMethod {
      // ...
      void (^someBlock)() = ^{
        _anInstanceVariable = @"something";
        NSLog(@"%@", _anInstanceVariable);
      };
    }
    
    @end
如果某个`EOCClass`实例正在执行`anInstanceMethod`方法，那么`self`变量就指向此实例。由于block中没有明确使用`self`变量，所以很容易就忘记`self`变量其实也被block所捕获了。直接访问实例变量和通过`self`来访问是等价的：

    self -> __anInstanceVariable = @"something";
    
之所以要捕获`self`变量，原因正在于此。`self`也是个变量，因而block在捕获它的时候也会retain。如果`self`指代的那个对象同时也保留了block，那么这种情况通常就会形成“保留环”。参见第40条。

#### block的内部结构
每个OC对象都占据着某个内存区域。因为实例变量的个数及对象所包含的关联数据互不相同，所以每个对象所占的内存区域也有大有小。block本身也是对象，在存放block对象的内存区域中，首个变量是指向`Class`对象的指针`isa`。其余内存里含有block对象正常运转所需的各种信息。

#### 全局block，栈block和堆block
定义block的时候，其所占的内存区域是分配在栈中的。这就是说，block只在定义它的那个范围内有效。下面这段代码就有危险：

    void (^block)();
    
    if (someCondition) {
      block = ^{
        NSLog(@"Block A");
      };
    } else {
      block = ^{
        NSLog(@"Block B");
      };
    }
    
    block();
    
定义在`if`和`else`语句中的两个block都分配在栈内存中。编译器会给每个block分配好栈内存，然而等离开了相应的范围后，编译器可能将分配给block的内存覆盖掉。于是，这两个block只能在对应的`if-else`语句中有效。这样写出的程序可以编译，但运行起来时而正确，时而错误。取决于编译器是否覆盖了待执行的block。  
解决此问题，可以给block发送`copy`消息，将其从栈复制到堆。复制后的block，就可以在定义范围外使用。而且，一旦复制到对象，block就变成了带引用计数的对象了。后续的复制操作不会真的执行复制，只是递增block对象的引用计数。如果不再使用这个block，就应该将其释放，在ARC下会自动释放，在MRC下需要手动调用`release`方法。当引用计数降为0后，heap stack会被系统回收。而stack block无需明确释放，会自动回收。

    void (^block)();
   
    if (someCondition) {
      block = [^{
        NSLog(@"Block A");
      } copy];
    } else {
      block = [^{
        NSLog(@"Block B");
      } copy];
    }
    
    block();
    
除了堆block和栈block外，还有一种全局block（global block）。这种block不会捕捉任何状态，运行时也无须状态参与。block所使用的整个内存区域在编译期就已经确定了，因此，全局block可以声明在全局内存中，不需要每次用到时在栈中创建。此外，全局block的`copy`操作是个空操作，因为全局block绝不会被系统回收。这种实际上block相当于单例。

    void (^block)() = ^{
      NSLog(@"This is a global block");
    };
如果把简单的block当成复杂的block处理，那么就会在复制和回收该block时执行一些无谓的操作。

### 第38条：为常用的block创建typedef
每个block都有其固有类型（inherent type），因而可将其赋值给适当类型的变量。这个类型由block接受的参数和其返回值组成。

    ^(BOOL flag, int value) {
      if (flag) {
        return value * 5;
      } else {
        return value * 10;
      }
    }
此block接受两个类型分别为BOOL和int的参数，返回int类型的值。如果想将其赋给变量，则需要注意其类型。

    int (^variableName)(BOOL flag, int value) = ^(BOOL flag, int value) {
      if (flag) {
        return value * 5;
      } else {
        return value * 10;
      }
    }
与其它类型的变量不同，在定义block变量时，要把变量名放在类型之中，而不要放在右边。这种语法非常难记。因此，我们应该为常用的block起个别名，尤其是在打算把代码发布成API供他人使用时，更应该这样做。  
为了隐藏复杂的block类型，需要用到C语言中“类型定义”（type definition）的特性。`typedef`关键字用于给类型起个易懂的别名。

    typedef int(^EOCSomeBlock)(BOOL flag, int value);
    
    EOCSomeBlock block = ^(BOOL flag, int value) {
      // Implementation
    };
    
通过这项特性，可以把使用block的API做得更加易用些。类里面又些方法可能需要用block来做参数，比如执行异步任务时所用的`completion handler`参数就是block，但凡遇到这种情况，都可以通过定义别名使代码变得更易懂。

    - (void)startWithCompletionHandler:(void (^)(NSData* data, NSError* error))completion;
    
    typedef void(^EOCCompletionHandler)(NSData* data, NSError* error);
    
    - (void)startWithCompletionHandler:(EOCCompletionHandler)completion;
    
另一个好处是，重构block的类型签名时会很方便。比如说，要给原来的`completion handler`再加一个参数，只需修改typedef语句即可。

    typedef void(^EOCCompletionHandler)(NSData* data, NSTimeInterval duration, NSError* error);
    
### 第39条：用handler block降低代码分散程度
为UI编码时，一种常见的范式就是“异步执行任务”（perform task asynchronously）。好处在于：处理UI现实及触摸操作所用的线程，不会因为执行IO或网络之类的耗时操作而阻塞。这个线程通常称为主线程（main thread）。若把异步任务做成同步的，那么在执行任务时，UI就会无法响应用户输入了。如果App一定时间内无响应，就会被系统终止。  
异步方法在执行完任务后，需要以某种手段通知相关代码。实现此功能有很多方法，常用的是设计一个`delegate protocol`，令关注此事件的对象遵从此协议。  
这种做法确实可行，而且没有什么错误。然而如果改用block来写的话，代码会更清晰。block可以令这种API更加紧凑，同时也令开发者调用时更加方便。办法就是：把completion handler定义为block类型，将其当作参数直接传给方法。

### 第40条：用block引用其所属对象时不要出现保留环
使用block时，若不仔细考虑，则很容易出现“保留环”（retain cycle）。  
获取器对象之所以要把completion handler保存在属性中，其唯一的目的就是想在稍后使用这个block。可是获取器一旦运行过block之后，就没有必要再保留它了。在使用之后将保留block的属性置为nil即可。

### 第41条：多用dispatch queue，少用同步锁
在OC中，如果有多个线程想要执行同一段代码，那么有时可能会出问题。这种情况下，通常要使用锁来实现某种同步机制。在GCD出现前，有两种办法，第一种是采用内置的“同步block”（syncchronization block）:

    - (void)synchronizedMethod {
      @synchronized(self) {
        // Safe with self
      }
    }
    
这种写法会根据给定的对象，自动创建一个锁，并等待block中的代码执行完毕。执行到代码结尾处，锁就释放了。这么写通常没错，因为它可以保证每个对象实例都能不受干扰地运行其`synchronizedMethod`方法。然而，滥用`@synchronized(self)`则会降低代码效率，因为共用一个锁的那些同步block，都必须按顺序执行。若是在`self`对象上频繁加锁，那么程序可能要等另一段与此无关的代码执行完毕，才能继续执行当前代码，这样做其实没有什么必要。  
另一个办法是直接使用`NSLock`对象：

    _lock = [[NSLock alloc] init];
    
    - (void)synchronizedMethod {
      [_lock lock];
      // Safe
      [_lock unlock];
    }
也可以使用`NSRecursiveLock`这种“递归锁”（recursive lock），线程能够多次持有该锁，而不会出现死锁（deadlock）现象。  
这两种方法都很好，但也有其缺陷。在极端情况下，同步block会导致死锁，另外，其效率也不见得很高，而直接使用锁对象，一旦遇到死锁，就会非常麻烦。  
替代方案是实用GCD，它能以更简单，更高效的形式为代码加锁。

    - (NSString *)someString {
      @synchronized(self) {
        return _someString;
      }
    }
    
    - (void)setSomeString:(NSString *)someString {
      @synchronized(self) {
        _someString = someString;
      }
    }
    
滥用`@synchronized(self)`会很危险，因为所有同步block都会彼此争夺同一个锁。要是有多个属性都这么写的话，那么每个属性的同步block都要等其他同步block执行完毕后才能执行，然而我们只想让每个属性独立的同步。  
另外，这种做法虽然能提供某种程度上的“线程安全”（thread safety），但却无法保证访问该对象时绝对是线程安全的。访问属性的操作确实是“原子的”。使用属性时，必定能从其中获取有效值，然而在同一个线程上多次访问getter，每次获取到的结果却未必相同。在两次访问操作之间，其他线程可能会写入新的属性值。  
有种简单的方法可以代替同步block或锁对象，那就是使用“串行同步队列”（serial synchronization queue）。将读取操作及写入操作都安排在同一个队列中，即可保证数据同步。

    _syncQueue = dispatch_queue_create("com.eoc.syncQueue", NULL);
    
    - (NSString *)someString {
      __block NSString* localSomeString;
      dispatch_sync(_syncQueue, ^{
        localSomeString = _someString;
      });
      return localSomeString;
    }
    
    - (void)setSomeString:(NSString *)someString {
      dispatch_sync(_syncQueue, ^{
        _someString = someString;
      });
    }
    
此模式的思路是：把getter和setter都安排在序列化的队列中执行，这样的话，所有针对属性的访问操作就都同步了。全部加锁任务都在GCD中处理，而GCD是在相当深的底层来实现的，于是能够做许多优化。因此，开发者无需担心那些事，只要专心把访问方法写好就行。  
还可以进一步优化。设置方法不一定非得是同步的。setter所用的block，并不需要返回什么值。

    - (void)setSomeString:(NSString *)someString {
      dispatch_async(_syncQueue, ^{
        _someString = someString;
      });
    }
执行异步派发时，需要拷贝block。若拷贝block所用的时间明显超过执行block所花的时间，则这种做法将比原来更慢。然而，若是派发给队列的block要执行更为繁重的任务，那么仍然可以考虑这种备选方案。 
多个setter可以并发执行，而getter和setter之间不能并发执行，利用这个特点，还能写出更快的代码来。这次不用串行队列，而用并发队列（concurrent queue）：

    _syncQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFALUT, 0);
    
    - (NSString *)someString {
      __block NSString* localSomeString;
      dispatch_sync(_syncQueue, ^{
        localSomeString = _someString;
      });
      return localSomeString;
    } 
  
    - (void)setSomeString:(NSString *)someString {
      dispatch_async(_syncQueue, ^{
        _someString = someString;
      });
    }
像这样写代码，还无法正确实现同步。所有读取操作和写入操作都会在同一个队列上执行，不过由于是并发队列，所以读取和写入操作可以随时执行。此问题用一个简单的GCD功能即可解决，就是栅栏barrier。

    void dispatch_barrier_async(dispatch_queue_t queue, dispatch_block_t block);
    void dispatch_barrier_sync(dispatch_queue_t queue, dispatch_block_t block);
    
在队列中，barrier block必须单独执行，不能与其他block并行。这只对并发队列有意义，因为串行队列中的block总是按顺序逐个执行的。并发队列如果发现接下来要处理的block是个barrier block，那么就会一直等到当前所有并发block都执行完毕，才回单独执行这个barrier block。等到barrier block执行过后，再按照正常方式继续向下处理。  
可以用barrier block来实现属性的setter。在setter中使用了barrier block之后，对属性的读取操作依然可以并发执行，但写入操作却必须单独执行了。

    - (void)setSomeString:(NSString *)someString P
      dispatch_barrier_async(_syncQueue, ^{
        _someString = someString;
      });
    }
    
### 第42条：多用GCD，少用performSelector系列方法
OC是一门非常动态的语言，NSObject定义了几个方法，令开发者可以随意调用任何方法。这几个方法可以推迟执行方法调用，也可以指定运行方法所用的线程。这些方程原来很有用，但是在GCD出现之后，就显得不那么必要了。  
其中最简单的是`performSelector:`，它接受一个参数，就是要执行的那个selector

    - (id)performSelector:(SEL)selector
    
    [object performSelector:@selector(selectorName)];
    // equal to
    [object selectorName];
    
这种方式看上去似乎多余。然而，如果selector是在运行期决定的，那么就能体现出这种方法的强大之处了。这就等于在动态绑定之上再次使用动态绑定，因而可以实现下面这种功能：

    SEL selector;
    if (condition1) {
      selector = @selector(foo);
    } else if (condition2) {
      selector = @selector(bar);
    } else {
      selector = @selector(baz);
    }
    [object performSelector:selector];
    
这种编程方式非常灵活，可用来简化复杂的代码。还有一种用法，就是先把selector保存起来，等某个时间发生之后再调用。不管哪种用法，编译器都不知道要执行的selector是什么，这必须到运行期才能确定。然而，使用此特性的代价是，如果再ARC下编译代码，编译器会发出如下警告：

    warning: performSelector may cause a leak because its selector is unknown
    
原因在于，编译器并不知道要调用的selector是什么，因此，也就不了解其方法签名和返回值，甚至连是否有返回值都不清楚。而且，由于编译器不知道方法名，所以就没办法运用ARC的内存管理规则来判定返回值是不是应该释放。因此，ARC采用了比较谨慎的做法，就是不添加释放操作。然而这么做可能导致内存泄漏，因为方法在返回对象时可能已经将其保留了。  
这些方法不是很理想，另一个原因在于：返回值只能是void或对象类型。尽管要执行的selector也可以范围void，但`performSelector:`方法的返回值类型毕竟是id。如果想返回整数或浮点数等类型的值，就需要执行一些复杂的转换操作了，而这种转换很容易出错。  
`performSelector`还有如下几个版本，用于在发消息时顺便传递参数：

    - (id)performSelector:(SEL)selector
               withObject:(id)object;
    - (id)performSelector:(SEL)selector
               withObject:(id)objectA;
               withObject:(id)objectB;
               
    [object performSelector:@selector(setValue:)
                 withObject:newValue];
这些方法貌似有用，其实局限很多。由于参数类型为id，所以必须传入对象。不能传入基本数据类型。此外，selector最多只能接受两个参数，在参数不止两个的情况下，则没有对应的方法能够执行。  
`performSelector`系列方法还有个功能，就是可以延后执行selector，或将其放在另一个线程执行。

    - (void)performSelector:(SEL)aSelector
                 withObject:(nullable id)anArgument
                 afterDelay:(NSTimeInterval)delay;
    - (void)performSelector:(SEL)aSelector
                   onThread:(NSThread *)thr
                 withObject:(nullable id)arg
              waitUntilDone:(BOOL)wait;
    - (void)performSelectorOnMainThread:(SEL)aSelector
                             withObject:(nullable id)arg
                          waitUntilDone:(BOOL)wait;
这些方法太过局限。例如，执行delay的那些方法都无法处理带有两个参数的selector。而能够指定执行线程的方法也与之类似，所以也不是特别通用。如果要用，就得把许多参数打包到字典中，然后在方法中提取出来，这样会增加开销，还可能会出bug。  
最主要的替换方案就是使用block。而且，`performSelector`系列方法提供的功能，都能通过在GCD中使用block来实现。延后执行可以使用`dispatch_after`来实现，在另一个线程上执行任务可以通过`dispatch_async`和`dispatch_sync`来实现。

    dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5.0 * NSEC_PER_SEC));
    dispatch_after(time, dispatch_get_main_queue(), ^{
      [self doSomething];
    });
    
    dispatch_async(dispatch_get_main_queue(), ^{
      [self doSomething];
    });
    
### 第43条：掌握GCD及Operation Queue的使用时机
在执行后台任务时，GCD并不一定是最佳方式。还有一种技术叫做`NSOperationQueue`，它虽然与GCD不同，但是却与之相关，开发者可把操作以`NSOperation`子类的形式放在队列中，而这些操作也能够并发执行。实际上，OperationQueue在底层是用GCD来实现的。  
在二者的差别中，首先要注意：GCD是纯C的API，而OperationQueue是OC的对象。在GCD中，任务用block来表示，而block是个轻量型数据结构。相反，operation则是个更为重量级的OC对象。虽说如此，但GCD并不总是最佳方案。有时候采用对象所带来的开销微乎其微，而带来的好处却大大超过其缺点。  
用`NSOperationQueue`类的`addOperationQueueWithBlock:`方法搭配`NSBlockOperation`类来使用operation queue，其语法与GCD方式非常类似。使用`NSOperation`和`NSOperationQueue`的好处如下：

* 取消某个operation。如果使用operation queue，那么想要取消operation是很容易的。运行任务前，可以在`NSOperation`对象上调用`cancel`方法，该方法会设置对象内的标志位，用以表明此任务不需执行，不过已经启动的任务无法取消。若是使用GCD，那就无法取消了。
* 指定operation间的依赖关系。一个operation可以依赖其他多个operation。开发者能够指定operation间的依赖关系，使特定的operation必须在另一个operation顺利执行完毕后方可执行。
* 通过KVO机制监控`NSOperation`对象的属性。`NSOperation`对象有许多属性都适用于KVO来监听，比如可以用`isCancelled`属性来判断任务是否已取消，又比如可以通过`isFinished`属性来判断任务是否已经完成。如果想在某个任务变更其状态时得到通知，KVO会很有用。
* 指定operation的优先级。operation的优先级表示此operation与队列中其他operation之间的优先关系。优先级高的operation先执行，优先级低的后执行。GCD则没有直接实现此功能的方法。GCD的队列的确有优先级，不过那是针对整个队列来说的，不是针对每个block来说的。

`NSOperation`对象也有“线程优先级”（thread priority），这决定了运行此操作的线程处于何种优先级上。

### 第44条：通过Dispatch Group机制，根据系统资源状况来执行任务
“派发组”（dispatch group）是GCd的一项特性，能够把任务分组。调用者可以等待这组任务执行完毕，也可以在提供callback函数之后继续执行，这组任务完成时，调用者会得到通知。这个功能有很多用途，其中最重要的用法，就是将要并发执行的多个任务和为一组，于是调用者就可以知道这些任务何时才能全部执行完毕。  
创建dispatch group：
    
    dispatch_group_t group = dispatch_group_create();
    
想把任务编组，有两个办法，第一种是用下面这个函数：

    void dispatch_group_async(dispatch_group_t group, dispatch_queue_t queue, dispatch_block_t block);
    
它是普通`dispatch_async`函数的变体，比原来多一个参数，用于表示待执行的block所归属的组。还有种办法能够指定任务所属的dispatch group，就是使用下面这一对函数：

    void dispatch_group_enter(dispatch_group_t group);
    void dispatch_group_leave(dispatch_group_t group);
    
    // 方法一等价于
    dispatch_group_enter(group);
    dispatch_async(queue, ^{
      // task here
      dispatch_group_leave(group);
    });
    
在使用dispatch group时，如果调用enter之后，没有相应的leave操作，那么这一组任务就永远做不完。  
下面这个函数可等待dispatch group执行完毕：

    long dispatch_group_wait(dispatch_group_t group, dispatch_time_t timeout);
    
此函数接受两个参数，一个是要等待的group，另一个是代表等待时间的timeout。timeout参数表示函数在等待dispatch group执行完毕时，应该阻塞多久。如果执行dispatch group所需的时间小于timeout，则返回0，否则返回非0。此参数也可以取常量`DISPATCH_TIME_FOREVER`，这表示函数会一直等着dispatch group执行完，而不会超时（time out）。  
除了使用上述函数等待dispatch group执行完毕外，也可以换个办法，使用：

    void dispatch_group_notify(dispatch_group_t group, dispatch_queue_t queue, dispatch_block_t block);
    
与wait函数不同的是，开发者可以向此函数传入block，等dispatch group执行完毕之后，block会在给定的queue上执行。假如线程不应阻塞，而开发者又想在那些任务全部完成时得到通知，就可以使用此方法。  
如果想令数组中的每个对象都执行某项任务，并且想等待所有任务执行完毕，那么就可以使用这个GCD特性来实现。

     dispatch_queue_t queue = dispatch_get_global_queue(QOS_CLASS_USER_INTERACTIVE, 0);
     dispatch_group_t dispatchGroup = dispatch_group_create();
     for (id object in collection) {
       dispatch_group_async(dispatchGroup, queue, ^{ [object performTask]; });
     }
     
     dispatch_group_wait(dispatchGroup, DISPATCH_TIME_FOREVER);
     
若不应阻塞当前线程，则可用notify函数代替wait：

    dispatch_queue_t notifyQueue = dispatch_get_main_queue();
    dispatch_group_notify(dispatchGroup, notifyQueue, ^{ // task after completion});
    
notify回调时所选的队列，应该根据具体情况来定。常见的是使用主队列，也可使用自定义的串行或全局并行队列。  
事实上，并不是所有任务都需要在同一个队列中。可以把某些任务放在优先级高的队列中，同时仍然把所有任务归到一个dispatch group中，并在执行完毕后获得通知。

    dispatch_queue_t lowPriorityQueue = dispatch_get_global_queue(QOS_CLASS_BACKGROUND, 0);
    dispatch_queue_t highPriorityQueue = dispatch_get_global_queue(QOS_CLASS_USER_INTERACTIVE, 0);
  
    dispatch_group_t dispatchGroup = dispatch_group_create();
  
    for (id object in lowPriorityObjects) {
      dispatch_group_async(dispatchGroup, lowPriorityQueue, ^{
        [object performTask];
      });
    }
  
    for (id object in highPriorityObjects) {
      dispatch_group_async(dispatchGroup, highPriorityQueue, ^{
        [object performTask];
      });
    }
  
    dispatch_group_notify(dispatchGroup, dispatch_get_main_queue(), ^{
      // run after completing tasks
    });

如果把任务提交到串行队列中，dispatch group的作用可以被替代。因为任务是逐个执行的，所以只需要在提交完全部任务后再提交一个block作为结束block即可。  
为了执行队列中的block，GCD会在适当的时机自动创建新线程或复用旧线程。如果使用并发队列，那么其中又可能会有多个线程，这也就意味着多个操作可以并发执行。在并发队列中，执行任务所用的并发线程数量，取决于各种因素，而GCD主要是根据系统资源状况来判定这些因素的。  
遍历某个collection，并在其每个元素上执行任务，这也可以用另一个GCD函数实现：

    void dispatch_apply(size_t iterations, dispatch_queue_t queue, void(^block)(size_t));
    
此函数会将`block`追加到`queue`中`iterations`次，每次block的参数值都会递增。  
`dispatch_async`函数是同步的，即阻塞当前线程直到所有任务执行完毕为止。使用时应注意死锁可能。

### 第45条：使用dispatch_once来执行只需运行一次的线程安全代码

    + (instancetype)sharedInstance {
      static EOCClass* sharedInstance = nil;
      static dispatch_once_t onceToken;
      dispatch_once(&onceToken, ^{
        sharedInstance = [[self alloc] init];
      });
      return sharedInstance;
    }
使用`dispatch_once`可以简化代码并且彻底保证线程安全，无需担心加锁或同步。由于每次调用时都必须使用相同的token，所以要声明为static。把变量定义在static作用域中，可以保证贬编译器每次执行`sharedInstance`方法时都会复用这个变量，而不会创建新变量。  
此外，`dispatch_once`更高效。

### 第46条：不要使用dispatch_get_current_queue
使用GCD时，经常需要判断当前代码正在哪个队列上执行。

    dispatch_queue_t dispatch_get_current_queue()
    
此函数返回当前正在执行代码的队列。不过用的时候要格外小心。  
此函数有种典型的错误用法(antipattern)，就是用它检测当前队列是不是某个特定队列，试图以此来避免async dispatch时可能出现的死锁问题。

    - (NSString *)someString {
      __block NSString* localSomeString;
      dispatch_sync(_syncQueue, ^{
        localSomeString = _someString;
      });
      return localSomeString;
    }
    
    - (void)setSomeString:(NSString *)someString {
      dispatch_async(_syncQueue, ^{
        _someString = someString;
      });
    }
    
这种写法的问题是，getter可能会死锁，假如调用getter的队列恰好是本例中的`_syncQueue`，那么`dispatch_sync`就一直不会返回。  
如果使用`dispatch_get_current_queue`，或许可以用其改写这个方法，若当前队列是同步操作所针对的队列，就不执行dispatch，直接执行block。

    - (NSString *)someString {
      __block NSString* localSomeString;
      dispatch_block_t accessorBlock = ^{
        localSomeString = _someString;
      };
      
      if (dispatch_get_current_queue() == _syncQueue) {
        accessorBlock();
      } else {
        dispatch_sync(_syncQueue, accessorBlock);
      }
      
      return localSomeString;
    }
    
这种做法可以处理一些简单情况。不过仍有死锁的危险。

    dispatch_queue_t queueA = dispatch_queue_create("eoc.queueA", DISPATCH_QUEUE_SERIAL);
    dispatch_queue_t queueB = dispatch_queue_create("eoc.queueB", DISPATCH_QUEUE_SERIAL);
    
    dispatch_sync(queueA, ^{
      dispatch_sync(queueB, ^{
        dispatch_sync(queueA, ^{
          // Deadlock
        });
      });
    });
    
这段代码执行到最内层的dispatch操作时总会死锁，因为此操作是针对queueA队列的，必须等最外层的`dispatch_sync`执行完毕才行，而最外层的`dispatch_sync`又不可能执行完毕，因为它要等最内层的`dispatch_sync`执行完，于是就死锁了。如果使用`dispatch_get_current_queue`来检测：

    dispatch_sync(queueA, ^{
      dispatch_sync(queueB, ^{
        dispatch_block_t block = ^{ /* operation */};
        if (dispatch_get_current_queue() == queueA) {
          block();
        } else {
          dispatch_sync(queueA, block);
        }
      });
    });
    
然而这样依然会死锁，因为`dispatch_get_current_queue`返回的是当前队列，在本例中就是queueB。  
在这种情况下，正确的做法是：不要把存取方法做成可重入的，而是应该确保同步操作所用的队列绝不会访问属性。这种队列之应该用来同步属性。由于dispatch queue是一种极为轻量的机制，所以，可以创建多个队列以保证每项属性都有自己专用的同步队列。  
使用队列时还要注意另一个问题，这个问题会在意想不到的地方导致死锁。队列之间会形成一套层次体系，这意味着排在某条队列中的block，会在其上级队列(parent queue 父队列)里执行。层级里地位最高的那个队列总是“全局并发队列”（global concurrent queue）。  
![image](http://7xt1ag.com1.z0.glb.clouddn.com/Screen%20Shot%202016-08-02%20at%2018.51.56.png)  
排在队列B或队列C中的block，稍后会在队列A中依序执行。于是，排在队列A、B、C中的block总是要彼此错开执行。然而，队列D中的block，则有可能与队列A中的block（包括队列B和队列C中的block）并行，因为A和D的目标队列是个并发队列。如有必要，并发队列可以用多个线程并行执行多个block，这取决于系统资源。  
由于队列间有层级关系，所以“检查当前队列是否为执行sync dispatch所用的队列”这种办法并不总是有效。比如说，排在队列C中的block会认为当前队列就是队列C，而如果在此block中对队列A进行sync dispatch，则同样会导致死锁。  


### 第47条：熟悉系统框架
将一系列代码封装成动态库（dynamic library），并在其中放入描述其接口的头文件，这样做出来的东西就叫框架。有时为iOS平台构建的第三方框架所使用的是静态库（static library），这是因为iOS不允许在其中包含动态库。这些东西严格来讲并不是真正的框架，然而也经常视为框架。不过，所有iOS平台的系统框架仍然使用动态库。  
开发者会碰到的主要框架就是Foundation。Foundation框架中的类，使用NS这个前缀。  
还有个与Foundation相伴的框架，叫做CoreFoundation。虽然从技术上来讲，CoreFoundation不是OC框架，但它却是编写OC应用程序时所应熟悉的重要框架。Foundation框架中的许多功能都可以在此框架中找到对应的C语言API。CoreFoundation和Foundation不仅名字相似，而且还有更紧密的联系。有个功能叫“无缝桥接”（tollfree bridging），可以把CoreFoundation中的C语言数据结构平滑转换成Foundation中的OC对象。  
除了Foundation和CF之外，还有很多系统库：

* `CFNetwork` 此框架提供了C语言级别的网络通信能力，它将“BSD套接字”（BSD socket）抽象成易于使用的网络接口。而Foundation则将该框架里的部分内容封装成OC的接口。
* `CoreAudio` 该框架提供的C语言API可用来操作设备上的音频硬件。
* `AVFoundation` 此框架所提供的OC对象可用来回放并录制音视频。
* `CoreData` 此框架提供的OC接口可将对象持久化到数据库中。
* `CoreText` 此框架提供的C语言接口可以高效执行文字排版及渲染操作。

OC编程的一项重要特点，就是经常需要使用底层的C语言API。用C语言提供的API的好处是，可以绕过OC的运行时系统，从而提升执行速度。当然，由于ARC只负责OC的对象，所以使用这些API时要注意内存管理问题。

### 第48条：多用block枚举，少用for循环
在编程中经常需要列举collection中的元素，当前的OC语言有多种方法实现此功能。
#### for循环
    NSArray* anArray = ...;
    for (int i = 0; i < anArray.count; i++) {
      id object = anArray[i];
      // do sth
    }
    
    NSDictionary* aDictionary = ...;
    NSArray* keys = [aDictionary allKeys];
    for (int i = 0; i < keys.count; i++) {
      id key = keys[i];
      id value = aDictionary[key];
      // do sth
    }
  
    NSSet* aSet = ...;
    NSArray* objects = [aSet allObjects];
    for (int i = 0; i < objects.count; i++) {
      id object = objects[i];
      // do sth
    }
根据定义，字典和Set都是无序的，无法通过下标访问其中的值。于是就需要先获取字典中的所有key或set中的所有对象，这两种情况下，都可以在获取到的有序数组上遍历。创建这个附加数组有额外开销。  
for循环可以实现反向遍历，此时使用for循环比其他方式简单许多。

#### 使用OC 1.0的NSEnumerator来遍历
NSEnumerator是个抽象类，其中只定义了两个方法，供其具体子类（concrete subclass）来实现：

    - (NSArray *)allObjects;
    - (id)nextObject;

其中关键的方法是`nextObject`，它返回枚举里的下个对象。每次调用该方法时，其内部数据结构都会更新，使得下次调用能返回下个对象。等到枚举中的全部对象都已返回之后，再调用就讲返回nil，这表示达到枚举末端了。

    NSArray* anArray = ...;
    NSEnumerator* enumerator = [anArray objectEnumerator];
    id object;
    while ((object = [enumerator nextObject]) != nil) {
      // do sth
    }
这种写法的功能和标准的for循环相似，但是代码却多了一点。其真正优势在于：不论遍历哪种collection，都可以采用这套相似的语法。  
使用`NSEnumerator`还有个好处，就是有多种enumerator可供使用。可以用`[anArray reverseObjectEnumerator]`来反向遍历collection中的元素。

#### 快速遍历
OC 2.0引入了快速遍历这一功能。它为for循环开设了in关键字。这个关键字大幅简化了遍历collection所需的语法：

    NSArry* anArray = ...;
    for (id object in anArray) {
      // do sth
    }
由于NSEnumerator对象也实现了`NSFastEnumeration`协议，所以能用来执行反向遍历。

    NSArry* anArray = ...;
    for (id object in [anArray reverseObjectEnumeration]) {
      // do sth
    }
这种办法是语法最简单且效率最高的。与传统的for循环不同，这种方法无法轻松获取当前遍历操作所针对的下标。

#### 基于block的遍历方式
在当前的OC中，最新的一种做法就是基于block来遍历。

    - (void)enumerateObjectsUsingBlock:(void(^)(id object, NSUInteger idx, BOOL* stop))block
    - (void)enumerateObjectsUsingBlock:(void (^)(ObjectType obj, BOOL *stop))block
    - (void)enumerateKeysAndObjectsUsingBlock:(void (^)(KeyType key, ObjectType obj, BOOL *stop))block
    
此方法大大胜过其他方法的地方在于，遍历时可以直接从block获取更多信息。在遍历数组时，可以知道当前所针对的下标。而在遍历字典时，可以同时获取键和值，从而省去了根据给定key获取value这一步。  
用此方法可以执行反向遍历。可向其中传入“选项掩码”（option mask）：

    - (void)enumerateObjectsWithOptions:(NSEnumerationOptions)opts usingBlock:(void (^)(ObjectType obj, NSUInteger idx, BOOL *stop))block
    - (void)enumerateObjectsWithOptions:(NSEnumerationOptions)opts usingBlock:(void (^)(ObjectType obj, BOOL *stop))block
    - (void)enumerateKeysAndObjectsWithOptions:(NSEnumerationOptions)opts usingBlock:(void (^)(KeyType key, ObjectType obj, BOOL *stop))block
    
    typedef NS_OPTIONS(NSUInteger, NSEnumerationOptions) {
      NSEnumerationConcurrent = (1UL << 0),
      NSEnumerationReverse = (1UL << 1),
    };
    
### 第49条：对自定义其内存管理语义的collection使用无缝桥接
OC的系统库包含相当多的collection类，其中有各种数组、各种字典、各种set。Foundation框架定义了这些collection及其他各种collection所对应的OC类。与之相似，CF框架也定义了一套C语言API，用于操作这些collection及其他collection的数据结构。`NSArray`时Foundation框架中的数组类，而`CFArray`则是CF框架中的等价物。这两种创建数组的方式也许有区别，然而有一项强大的功能可在这两个类型间平滑转换，它就是“无缝桥接”（toll-free bridging）。  
C语言级别的数据结构和OC中的类或对象并不相同。例如，`CFArray`要通过`CFArrayRef`来引用，而这是指向`struct__CFArray`的指针。`CFArrayGetCount`这种函数则可以操作此struct，获得数组大小。这和OC中的对应物不同。在OC中，可以创建NSArray对象，并在其上调用`count`方法，以获取数组大小。  

    NSArray* anNSArray = @[@1, @2, @3, @4, @5];
    CFArrayRef aCFArray = (__bridge CFArrayRef)(anNSArray);
    NSLog(@"size of array = %ld", CFArrayGetCount(aCFArray));
    // size of array = 5
    
转换操作中的`__bridge`告诉ARC，如何处理转换所涉及的OC对象。`__bridge`的意思是，ARC本身仍具备这个OC对象的所有权。`__bridge_retained`则与之相反，意味着ARC将交出对象的所有权。

    NSArray* anNSArray = @[@1, @2, @3, @4, @5];
    CFArrayRef aCFArray = (__bridge_retained CFArrayRef)(anNSArray);
    NSLog(@"size of array = %ld", CFArrayGetCount(aCFArray));
    CFRelease(aCFArray);

与之相似，反向转换可以通过`__bridge_transfer`来实现。比方说，想把CFArrayRef转换成NSArray*，并且想令ARC获得对象所有权，那么就可以采用此种转换方式。  
为何要用到这种功能呢？Foundation框架中的OC类所具备的某些功能，是CF框架中的C语言数据结构所不具备的，反之亦然。在使用Foundation框架中的字典对象时会遇到一个大问题，就是其键的内存管理语义为copy，而值的语义却是retain。除非使用无缝桥接，否则无法改变其语义。  
CF框架中的字典类型叫做CFDictionary。其可变版本成为CFMutableDictionary。创建CFMutableDictionary时，可以通过下列方法改变key和value的内存管理语义：

    CFMutableDictionary
    
### 第50条：构建缓存时选用NSCache而非NSDictionary
开发应用时，经常遇到的一个问题是，下载的图片应该如何缓存。首先想到的办法就是把内存中的图片保存到字典里，这样的话，稍后使用就无须再次下载了。相比于NSDictionary，NSCache类更好，它是Foundation框架专为处理这种任务而设计的。  
NSCache胜过NSDictionary之处在于，当系统资源将要耗尽时，它可以自动删减缓存。此外，NSCache还会先行删减“最久未使用的”（lease recently used）对象。若想自己编写代码来为字典添加此功能，则会十分复杂。  
NSCache不会“拷贝”key，而是“保留”它。NSCache对象不拷贝键的原因在于：很多时候，键都是由不支持copy的对象来充当的。另外，NSCache是线程安全的。即：在不编写加锁代码的情况下，多个线程便可以同时访问NSCache。  
开发者可以操控缓存删减其内容的时机。有两个尺度可供调整，其一是缓存中的对象总数，其二是所有对象的“总开销”（overall cost）。开发者在将对象加入缓存时，可为其指定“开销值”。当对象总数或总开销超过上限时，缓存就可能会删减其中的对象，在系统资源紧张时也会这么做。然而，“可能”会删减某个对象，并不意味着“一定”会删减这个对象。删减对象时所遵照的顺序，由具体实现来定。这说明，想通过调整“开销值”迫使缓存优先删除某对象，不是个好主意。  
向缓存中添加对象时，只有在能很快计算出“开销值”的情况下，才应该考虑采用这个尺度。

    @implementation EOCClass {
      NSCache* _cache;
    }
    
    - (instancetype)init {
      if (self = [supaer init]) {
        _cache = [NSCache new];
        
        _cache.countLimit = 100;
        
        _cache.totalCostLimit = 5 << 10 << 10;
      }
      return self;
    }
    
    - (void)downloadDataForURL:(NSURL *)url {
      NSData* cachedData = [_cache objectForKey:url];
      if (cachedData) {
        // cache hit
        // [self useData:cachedData];
      } else {
        // cache miss
        EOCNetworkFetcher* fetcher = [[EOCNetworkFetcher alloc] initWithURL:url];
        [fetcher startWithCompletionHandler:^(NSData *data) {
          [_cache setObject:data forKey:url cost:data.length];
          // [self useData:cachedData];
        }];
      }
    }
    
    @end
    
下载数据所用的URL，就是缓存的key。若缓存未命中（cache miss），则下载数据并将其放入缓存。而数据的“开销值”则设为其长度。创建NSCache时，则其中可缓存的总对象数目上限设为100，将“总开销”上限设为5MB。不过，由于“开销值”以“字节”为单位，所以要将MB转换为B。  
还有个类叫做`NSPurgeableData`，和`NSCache`搭配起来用，效果很好。此类是`NSMutableData`的子类，而且实现了`NSDiscardableContent`协议。如果某个对象所占的内存能够根据需要随时丢弃，那么就可以实现该协议所定义的接口。也就是说，当系统资源紧张是，可以把保存`NSPurgeableData`对象的那块内存释放掉。  
如果需要访问某个`NSPurgeableData`对象，可以调用其`beginContentAccess`方法，告诉它现在还不应丢弃自己所占据的内存。用完之后，调用`endContentAccess`方法，告诉它可以在必要时丢弃自己所占据的内存。这些调用可以嵌套，类似于引用计数。  
如果将`NSPurgeable`对象加入`NSCache`，那么当该对象为系统所丢弃时，也会自动从缓存中移除。通过`NSCache`的`evictsObjectsWithDiscardedContent`属性，可以开启或关闭此功能。

    - (void)downloadDataForURL:(NSURL *)url {
      NSPurgeableData* cachedData = [_cache objectForKey:url];
      if (cachedData) {
        // cache hit
        
        [cachedData beginContentAccess];
        
        // [self useData:cachedData];
        
        [cachedData endContentAccess];
      } else {
        // cache miss
        EOCNetworkFetcher* fetcher = [[EOCNetworkFetcher alloc] initWithURL:url];
        [fetcher startWithCompletionHandler:^(NSData *data) {
          NSPurgeableData* purgeableData = [NSPurgeableData dataWithData:data];
          
          [_cache setObject:purgeableData forKey:url cost:purgeableData.length];
          
          // [self useData:data];
          
          [purgeableData endContentAccess];
        }];
      }
    }
创建好`NSPurgeableData`对象后，其“purge计数”会多1，所以无须调用`beginContentAccess`，但在使用结束后要调用`endContentAccess`，抵消多出的这个1。

### 第51条：精简initialize和load的实现代码
有时候，类必须先执行某些初始化操作，然后才能正常使用。在OC中，绝大多数类都继承自NSObject这个父类，而该类有两个方法，可用来实现这种初始化操作。

    + (void)load
对于每个加入运行期系统的类和category来说，必定会调用此方法，而且仅调用一次。当包含类或category的程序库载入系统时，就会执行此方法。对iOS而言，通常就是应用启动的时候。如果category和其所属的类都定义了load方法，则先调用类里的，在调用category里的。  
`load`的问题在于，执行该方法时，运行期系统处于“脆弱状态”（fragile state）。在执行子类的load方法之前，必然会执行所有父类的load方法，而如果代码还依赖了其他程序库，那么程序库里相关类的load方法也必定会先执行。然而，根据某个给定的程序库，却无法判断出其中各个类的载入顺序。因此，在load类中使用其他类是不安全的。  
load方法并不想普通的方法一样，他不遵从那套继承规则。如果某个类本身没实现load方法，那么不管其各级父类是否实现此方法，都不会被调用。此外，类和category中都可能出现load代码，两种实现都会被调用，且类中的实现先执行。  
另一个方法是`initialize`方法

    + (void)initialize
    
对于每个类来说，该方法会在程序首次使用该类之前调用，且只调用一次。其与load方法有几点区别。第一，它是“惰性调用的”，也就是说，只有当系统用到相关的类时，才会被系统调用。  
另外，运行期系统在执行该方法时，时处于正常状态的。因此，此时可以安全使用任意类中的任意方法。而且，运行期系统也能确保`initialize`方法一定在线程安全环境中执行。即，只有执行`initialize`的那个线程可以操作类或类实例。其他线程都要先阻塞，等着`initialize`执行完。  
最后，如果某个类未实现`initialize`方法，就会运行其父类的实现。

    @import Foundation;
    
    @interface EOCBaseClass : NSObject
    @end
    
    @implementation EOCBaseClass
    + (void)initialize {
      NSLog(@"%@ initialize", self);
    }
    @end
    
    @interface EOCSubClass : EOCBaseClass
    @end
    
    @implementation EOCSubClass
    @end
    
首次使用`EOCSubClass`时，控制台输出：

    EOCBaseClass initialize
    EOCSubClass initialize
    
这两个方法的实现代码要尽量精简。在里面设置一些状态，使本类能够正常运作就可以了，不要执行那种耗时太久或需要加锁的任务。对于某个类来说，任何线程都可能成为初次用到它的那个线程，并导致其初始化。如果这个线程碰巧是主线程，那么初始化期间就会一直阻塞，导致应用程序无响应。很难预测是那个线程会最先用到这个类。  
第二，开发者无法控制类的初始化时机。类在使用前，肯定要初始化，但编写程序时不能令代码依赖特定的时间点，否则会很危险。  
最后，如果某个类的实现代码很复杂，那么其中可能会直接或间接使用到其他类。如果那些类尚未初始化，则系统会强迫其初始化。然而，本类的初始化方法此时尚未运行完毕。其他类在运行其initialize方法时，有可能会依赖本类中的某些数据，而这些数据此时尚未初始化好。  

    #import <Foundation/Foundation.h>
    
    static id EOCClassAInternalData;
    @interface EOCClassA : NSObject
    @end
    
    static EOCClassBInternalData;
    @interface EOCClassB : NSObject
    @end
    
    @implementation EOCClassA 
    
    + (void)initialize {
      if (self == [EOCClassA class]) {
        [EOCClassB doSthWithInternalData];
        EOCClassAInternalData = [self setupInternalData];
      }
    }
    
    @end
    
    @implementation EOCClassB
    
    + (void)initialize {
      if (self == [EOCClassB class]) {
        [EOCClassB doSthWithInternalData];
        EOCClassBInternalData = [self setupInternalData];
      }
    }
    
    @end
    
若是ClassA先初始化，那么ClassB随后也会初始化，它会在自己的初始化方法中调用ClassA的`doSthWithInternalData`，而此时ClassA内部的数据还没准备好。  
所以说，initialize方法只应该设置内部数据。不应该在其中调用任何方法，即使是本类自己的方法。若某个全局状态无法在编译期初始化，则可以放在initialize里来做。  

    //EOCClass.h
    #import <Foundation/Foundation.h>
    
    @interface EOCClass : NSObject
    @end
    
    
    //EOCClass.m
    #import "EOCClass.h"
    
    static const int kInterval = 10;
    static NSMutableArray* kSomeObject;
    
    @implementation EOCClass
    
    + (void)initialize {
      if (self == [EOCClass class]) {
        kSomeObject = [NSMutableArray arrayWithObjects:@1, @2, nil];
      }
    }
    
    @end
整数可以在编译期定义，然而可变数组不行，因为它是个OC对象，所以创建实例之前必须先激活运行期系统。某些OC对象也可以在编译期创建，例如NSString实例。

### 第52条：别忘了NSTimer会保留目标对象
计时器是一种很方便也很有用的对象。Foundation框架中有个类叫做`NSTimer`，开发者可以指定相对或决定时间，用以到时执行任务。也可以以一定间隔重复执行任务。  
timer要和runloop相关联，运行runloop时会触发任务。创建NSTimer时，可以将其安排在当前的runloop中，也可以先创建好，然后自己调度。无论采用哪种方法，timer必须放在runloop中才能正常触发任务。  
由于timer会保留目标对象，所以反复执行任务通常会导致应用程序出问题。也就是说，很容易产生循环引用。  

    // EOCClass.h
    #import <Foundation/Foundation.h>
    
    @interface EOCClass : NSObject
    
    - (void)startPolling;
    - (void)stopPolling;
    
    @end
    
    
    // EOCClass.m
    
    @implementation EOCClass {
      NSTimer* _poolTimer;
    }
    
    - (void)startPolling {
      _poolTimer = [NSTimer scheduledTimerWithTimeInterval:5
                                                    target:self
                                                  selector:@selector(p_doPoll)
                                                  userInfo:nil
                                                   repeats:YES];
    }
    
    - (void)p_doPoll {
      
    }
    
    - (void)stopPolling {
      [_poolTimer invalidate];
      _poolTimer = nil;
    }
    
    - (void)dealloc {
      [_poolTimer invalidate];
    }
    
    
    @end