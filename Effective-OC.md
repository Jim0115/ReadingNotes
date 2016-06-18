# Effective Objective-C 2.0 e-book
### 第1条：了解OC的起源
OC使用消息结构（messaging structure）而非函数调用（function calling）。二者的区别如下：

    // Messaging (OC)
    
    Object* obj = [Object new];
    [Obj performWith:parameter1 and:parameter2];
    
    // Function calling (C++)
    
    Object* obj = new Object;
    obj -> perform(parameter1, parameter2);
    
关键区别在于：使用消息结构的语言，其运行时所应执行的代码由运行环境来决定；而实用函数调用的语言，则由编译器决定。如果范例代码中调用的函数是多态的，那么在运行时就要按照虚方法表（virtual table）来查出到底应该执行哪个函数实现。而采用消息结构的语言，无论是否多态，总是在运行时才会去查找所要执行的方法。实际上，编译器甚至不关心接收消息的对象是何类型。接受消息的对象问题也要在运行时处理，其过程叫做“动态绑定”（dynamic binding）。  
OC的重要工作都由“运行期组件”（runtime component）而非编译器来完成。使用OC的面向对象特性所需的全部数据结构及函数都在运行期组件里面。运行期组件本质上就是一种与开发者所编代码相链接的“动态库”（dynamic library），其代码能把开发者编写的所有程序粘合起来。这样只需要更新运行期组件，即可提升性能。而工作都在“编译期”（compile time）完成的语言，若想获得类似的性能提升，则要重新编译所有代码。  
对象所占内存总是在堆（heap）中，而绝不会分配在栈（stack）中。

    NSString* someString = @"The string";
    NSString* anotherString = someString;
    
表示在栈中分配了两块内存，每块内存都包含了指向同一堆空间的指针。  
分配在堆中的内存必须直接管理，分配在栈上的内存会在其栈帧弹出时自动清理。  

### 第2条：在类的头文件中尽量少引入其他头文件
在头文件中声明一个其他类的对象时，不需要知道这个类的全部细节，只知道有这么一个类即可。为解决这种情况，使用

    @class SomeClass;
    
作为向前声明（forward declaring），在实现文件中引入头文件。将引入头文件的时机尽量延后，只有在确有需要的时候才引用。  
同时，向前声明也解决了两个类相互引用的问题。如果在两个类的声明文件中互相引入对方的头文件，则编译时会导致“循环引用”（chicken-and-egg situation）。当解析其中一个头文件时，编译器会发现它引入了另一个头文件，而那个头文件又回过头引用第一个头文件。使用`#import`而非`#include`指令虽然不会导致死循环，但类无法被正常编译。  

### 第3条：多用字面量语法，少用与之等价的方法
#### 字面数值

    NSNumber* num = [NSNumber numberWithInt:10];
    
    NSNumber* intNum = @1;
    NSNumber* floatNum = @2.5f;
    NSNumber* doubleNum = @3.14159;
    NSNumber* boolNum = @YES;
    NSNumber* charNum = @'a';
#### 字面量数组
    NSArray* animals = [NSArray arrayWithObjects:@"cat", @"dog", @"panda", nil]; // bad
    
    NSArray* animals = @[@"cat", @"dog", @"panda"];
    
    NSString* dog = [animals objectAtIndex:1]; // bad
    
    NSString* dog = animals[1];
    
在使用字面量创建数组时，如果数组元素对象中有nil，则会抛出`attempt to insert nil object from objects[2]'`异常。如果使用`arrayWithObjects`创建数组时，则会从nil位置截断，方法提前结束。  
二者相比，抛出异常终止程序是更加正确的做法。向数组中插入nil通常意味着程序出错。
#### 字面量字典
    NSDictionary* dict = [NSDictionary dictionaryWithObjectsAndKeys:@"Matt", @"firstName", @"Galloway", @"lastName", @10, @"age", nil]; // bad
    
    NSDictionary* dict = @{@"firstName" : @"Matt",
                            @"lastName" : @"Galloway",
                                 @"age" : @28};                               
第一种写法的顺序为 `<value>, <key>, <value>, <key>, ...`，不容易读懂。

    NSString* lastName = [dict objectForKey:@"lastName"];
    NSString* lastName = dict[@"lastName"];
像数组一样也可以使用字面量语法进行访问。
#### 可变数组与字典
    [mutableArray replaceObjectAtIndex:1 withObject:@"dog"];
    [mutableDictionary setValue:@"Aloha" forKey:@"firstName"];
    
    mutableArray[1] = @"dog";
    mutableDictionary[@"firstName"] = @"Aloha";
使用下标可以修改可变集合的元素值，相比与标准做法更加简单。

#### 局限性
使用字面量语法，除字符串外，创建出的对象必须属于Foundation框架。如果自定义这些类的子类，则无法用字面量语法创建其对象。一般来说，实现的标准已经很好，很少有人会从中创建自定义子类。  
使用字面量创建的字符串、数组、字典对象都是不可变的（immutable）。若想创建可见版本的对象，需要复制一份：

    NSMutableArray* mutable = [@[@1, @2, @3] mutableCopy];
    
### 第4条：多用类型常量，少用#define预编译指令
定义常量时，尽量少用`#define`定义：

    #define ANIMATION_DURATION 0.3
    
    static const NSTimeIntervar kAnimationDuration = 0.3;
用此方式定义的常量包含类型信息，清楚地描述了常量的含义。  
这种方式定义常量的命名法为：若常量局限于某“编译单元”（translation unit，也就是“实现文件”，implementation file）之内，则在其前面加字母k；若常量在类之外可见，则通常以类名为前缀。  
变量一定要同时使用`static`和`const`来声明。`const`表示变量不可修改，`static`表示该变量仅在定义此变量的编译单元中可见。  
实际上，如果一个变量既声明为`static`，又声明为`const`，那么编译器根本不会创建符号，而是会像`#define`一样，把所有遇到的变量都替换为常量值。  
有时需要对外公开某个常量值，此类常量需要放在"全局符号表"中，以便可以在定义该常量的编译单元外使用。这张变量应该这样定义：

    // In the header file
    extern NSString* const FooStringConstant;
    
    // In the implementation file
    NSString* const FooStringCOnstant = @"sth";
    
这个常量在头文件中“声明”，且在实现文件中“定义”。  
编译器看到头文件中的`extern`关键字，这个关键字是要告诉编译器，在全局符号表中将会有一个名叫`FooStringConstant`的符号。也就是说，编译器无需查看其定义，即允许代码使用此常量。因为它知道，当链接成二进制文件之后，一定能找到这个常量。

### 第5条：用枚举表示状态、选项、状态码
#### 状态：
    enum EOCConnectionState {
        EOCConnectionStateDisconnected,
        EOCConnectionStateConnecting,
        EOCConnectionStateConnected,
    };
    
    typedef enum EOCConnectionState EOCConnectionState;
    
    EOCConnectionState state = EOCConnectionStateConnected;
    
#### 选项：
    enum UIViewAutoresizing {
        UIViewAutoresizingNone                 = 0,
        UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
        UIViewAutoresizingFlexibleWidth        = 1 << 1,
        UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
        UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
        UIViewAutoresizingFlexibleHeight       = 1 << 4,
        UIViewAutoresizingFlexibleBottomMargin = 1 << 5
    }
使用按位或`|`组合多个选项。  
`UIViewAutoresizingFlexibleLeftMargin | UIViewAutoresizingFlexibleRightMargin`  
使用按位与`&`判断是否启用某个选项：  

    if (resizing & UIViewAutoresizingFlexibleWidth) {
      // UIViewAutoresizingFlexibleWidth is set
    }
    
#### Foundation框架中的宏定义
    typedef NS_ENUM(NSUInteger, EOCConnectionState) {
      EOCConnectionStateDisconnected,
      EOCConnectionStateConnecting,
      EOCConnectionStateConnected,
    };
  
    typedef NS_OPTIONS(NSUInteger, EOCPermittedDirection) {
      EOCPermittedDirectionUp = 1 << 0,
      EOCPermittedDirectionDown = 1 << 1,
      EOCPermittedDirectionLeft = 1 << 2,
      EOCPermittedDirectionRight = 1 << 3,
    };
通常使用宏定义方式，会针对不同情况进行选择，保证枚举可用。

#### 枚举与switch
枚举类型经常与switch搭配使用。但在switch枚举类型时，最好不要加上default分支。这样的话，如果稍后又加了一种状态，那么编译器就会发出警告，提示新加入的状态没有在switch分支中处理。如果写上了default分支，那么它就会处理这个新状态，从而导致编译器不发出警告信息。

### 第6条：理解“属性”这一概念
    @interface EOCPerson: NSObject {
      @public
        NSString* _firstName;
        NSString* _lastName;
      @private
        NSString* _someInternalData;
    }
    @end
在Java或C++中可以定义实例变量的作用域。在OC中却很少这么做。这种写法的问题是：对象布局在编译期（compile time）就已经固定了。只要碰到访问`_firstName`变量的代码，编译器就把其替换为“偏移量”(offset)，这个偏移量是“硬编码”(hardcode)，表示该变量距离存放对象的内存区域的起始地址有多远。但是如果又加了一个实例变量，则偏移量会指向错误的内存区域。  
如果使用了编译期计算出的偏移量，那么在修改类定义后必须重新编译。OC应对此问题的做法是，把实例变量当做一种储存偏移量所用的“特殊变量”（special variable），交由“类对象”（class object）保管（见14条）。偏移量会在运行期查找，如果类的定义变了，那么存储的偏移量也变了，这样的话，无论何时访问实例变量，总能使用正确的偏移量。这样，不一定要在接口中把全部实例变量都声明好，可以在实现单元声明部分变量，保护与类实现相关的内部信息。

#### 属性特质
##### 原子性
默认情况下，编译器合成的方法会通过锁定机制确保其原子性(atomicity)。如果属性具备nonatomic特质，则不使用同步锁。
##### 读写权限
具备readwrite特质的属性拥有getter和setter。若该属性由@synthesize实现，则编译器会自动生成这两个方法。  
具备readonly特质的属性仅拥有获取方法。只有当属性由@synthesize实现，编译器才会为其合成获取方法。可以将某个属性对外公开为只读属性，然后在“class-continuation分类”中将其重新定义为读写属性。（详见第27条）
##### 内存管理语义
属性用于封装数据，而数据要有“具体的所有权语义”（concrete ownership semantic）。

* `assign` “设置方法”只会执行针对“纯量类型”（CGFloat，NSInteger等）的简单赋值操作。
* `strong` 定义了一种“拥有关系”（owning relationship）。设置新值时，会先保留新值，并释放旧值，然后将新值设置上去。
* `weak` 定义了一种“非拥有关系”（nonowning relationship）。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。在属性所指对象销毁时，属性值被置为nil。
* `unsafe_unretained` 适用于对象类型，不保留新值。当属性所指对象销毁时，属性值不会自动清空。
* `copy` 将传递的新值进行copy。通常适用于拥有可变形式子类的属性，如`NSString`, `NSArray`, `NSSet`, `NSDictionary`等。

##### 方法名
指定存取方法的方法名：

* `getter=<name>` 如果想为某Boolean型变量的获取方法加上"is"前缀，就可以使用这个方法来指定 `@property (nonatomic, getter=isOn) BOOL on;`
* `setter=<name>` 注意：如果自己实现这些存取方法，应该保证其具备相关属性所声明的特质。比如，如果将某个属性声明为copy，就应该在设置方法中拷贝相关对象。  

在其他方法设置属性值，同样要遵守属性定义中宣称的语义。
    
    // EOCPerson.h
    @interface EOCPerson : NSObject

    @property (nonatomic, copy, readonly) NSString* firstName;
    @property (nonatomic, copy, readonly) NSString* lastName;
    
    - (instancetype)initWithFirstName:(NSString*)firstName
                             lastName:(NSString*)lastName;
    
    @end
    
    
    // EOCPerson.m
    @implementation EOCPerson
    
    - (instancetype)initWithFirstName:(NSString *)firstName lastName:(NSString *)lastName {
      if (self = [super init]) {
        _lastName = [lastName copy];
        _firstName = [firstName copy];
      }
      
      return self;
    }
    
    @end
    
### 第7条：在对象内部尽量直接访问实例变量
通过属性和直接访问实例变量有一下几点区别：

* 由于不经过Objective-C的“方法派发”（见第11条）步骤，所以直接访问实例变量速度较快。
* 直接访问实例变量时，不会调用其设置方法，这就绕过了“内存管理语义”。
* 直接访问实例变量，不会触发“键值观测”（Key-Value Observing, KVO）通知。这样做是否会产生问题，还取决于具体的对象行为。
* 通过属性访问有助于排查错误，因为可以给“获取方法”和“设置方法”添加断点，监控属性的调用者及其访问时机。

一个合理的折中方案是，写入实例变量时，通过“设置方法”，读取实例变量时，直接读取。使用此种方法需要注意几个问题：  
    
1. 在初始化方法中，总是应该直接访问实例变量，因为子类可能会override设置方法
2. 另外，使用“惰性初始化”（lazy initialization）。在这种情况下，必须通过“获取方法”来访问属性，否则，实例变量永远不会被初始化。

### 第8条：理解“对象等同性”
OC中`==`操作比较的是两个指针本身，而不是其所指的对象。应该使用NSObject协议中声明的`isEqual`方法来判断两个对象的等同性。一般来说，两个不用类型的对象总是不相等的。某些对象提供了特殊的“等同性判定方法”，如果已经知道两个对象同属于一个类，就可以使用这种方法。

    NSString* foo = @"Badger 123";
    NSString* bar = [NSString stringWithFormat:@"Badger %d", 123];
  
    NSLog(@"%d %d %d", foo == bar, [foo isEqual:bar], [foo isEqualToString:bar]); // 0 1 1 
    
`isEqualToString:`即为`NSString`实现的独有的判断方法。传递给该方法的对象必须是`NSString`，否则结果未定义。调用该方法比调用`isEqual:`快，后者还要执行额外的步骤，因为不知道受测对象的类型。

`NSObject`协议中有两个用于判断等同性的关键方法：

    - (BOOL)isEqual:(id)object;
    - (NSUInteger)hash;
    
`NSObject`对这两个类的默认实现是：当且仅当“指针值”完全相等时，两个对象才相等。如果想在自定义对象中正确override此方法，需要遵守一定约定。如果`isEqual:`方法判断两个对象相等，那么其`hash`方法也必须返回同一个值。但是，如果两个对象的`hash`方法返回同一个值，那么`isEqual:`方法未必认为二者相等。  
对于一个拥有`firstName`,`lastName`,`age`属性的`EOCPerson`的`isEqual:`方法可以写成：

    - (BOOL)isEqual:(id)object {
      if (self == object) return YES;
      if ([self class] != [object class]) return NO;
      
      EOCPerson* otherPerson = (EOCPerson*)object;
      
      if (![_firstName isEqualToString:otherPerson.firstName]) return NO;
      if (![_lastName isEqualToString:otherPerson.lastName]) return NO;
      if (_age != otherPerson.age) return NO;
      
      return YES;
    }
    
`hash`方法的一种实现方法：
    
    - (NSUInteger)hash {
      NSUInteger firstNameHash = [_firstName hash];
      NSUInteger lastNameHash = [_lastName hash];
      
      return firstNameHash ^ lastNameHash ^ _age;
    }
    
如果需要经常判断相等性，需要自己创建等同判定方法，因为无需检测参数类型，所以能大大提升检测速度。编写判定方法时，也应该一并重写`isEqual:`方法。常见的实现方式为：如果受测的参数与接收消息的对象属于同一个类，那么就调用自己的判定方法，否则就交给超类来判断。

    - (BOOL)isEqualToPerson:(EOCPerson*)anotherPerson {
      if (self == anotherPerson) return YES;
      
      if (![_firstName isEqualToString:anotherPerson.firstName]) return NO;
      if (![_lastName isEqualToString:anotherPerson.lastName]) return NO;
      if (_age != anotherPerson.age) return NO;
      
      return YES;
    }
    
    - (BOOL)isEqual:(id)object {
      if (self.class == [object class]) {
        return [self isEqualToPerson:object];
      } else {
        return [super isEqual:object];
      }
    }

### 第9条：以“类簇模式”隐藏实现细节
假设有一个处理雇员的类，每个雇员都有名字和薪水两个属性，管理者可以命令其执行工作。但是，各种雇员的工作内容却不同。经理在带领雇员做项目时，无需关心每个人如何完成其工作，只需指示其开工即可。  

    // EOCEmployee.h
    typedef NS_ENUM(NSUInteger, EOCEmployeeType) {
      EOCEmployeeTypeDeveloper,
      EOCEmployeeTypeDesigner,
      EOCEmployeeTypeFinlance,
    };
    
    
    @interface EOCEmployee : NSObject
    
    @property (nonatomic, copy) NSString* name;
    @property (nonatomic, assign) NSUInteger salary;
    
    + (instancetype)employeeWithType:(EOCEmployeeType)type;
    
    - (void)doWork;
    
    @end
    
    
    // EOCEmployee.m
    @implementation EOCEmployee
    
    + (instancetype)employeeWithType:(EOCEmployeeType)type {
      switch (type) {
        case EOCEmployeeTypeDeveloper: {
          return [EOCEmployeeTypeDeveloper new];
          break;
        }
        case EOCEmployeeTypeDesigner: {
          return [EOCEmployeeTypeDesigner new];
          break;
        }
        case EOCEmployeeTypeDesigner: {
          return [EOCEmployeeTypeDesigner new];
          break;
        }
      }
    }
    
    - (void)doWork {
      // subclasses implement this.
    }
    
    @end
    
每个实体类都继承自抽象父类：

    @interface EOCEmployeeDeveloper : EOCEmployee
    @end
    
    @implementation EOCEmployeeDeveloper
    
    - (void)doWork {
      [self writeCode];
    }
    
    @end
    
如果对象的类位于某个类簇中，那么在使用内省时就要格外小心。你觉得创建的是某个类的实例，实际上创建的却是其某个子类的实例。

#### Cocoa里的类簇
大部分collection都是类簇中的抽象基类。在使用NSArray的alloc方法来获取实例时，该方法首先会分配一个属于某类的实例。此实例充当“站位数组”（placeholder array）。该数组稍后会转为另一个类的实例，而那个类则是NSArray的实体子类。

    id maybeAnArray = @[@1, @2];
    if ([maybeAnArray class] == [NSArray class]) {
      // will never be hit
    }
    
`[maybeAnArray class]`所返回的类绝对不可能是NSArray，因为由NSArray的初始化方法所返回的那个实例其类型是隐藏在类簇公共接口(public facede)后面的某个内部类型(internal type)。  
判断某对象是否属于某个类簇中，不要直接检测两个“类对象”是否等同，而应该采用下列代码：

    id maybeAnArray = @[@1, @2];
    if ([maybeAnArray isKindOfClass:[NSArray class]]) {
      // will be hit
    }
    
### 第10条：在既有类中使用关联对象存放自定义数据
对象关联类型

关联类型 | 等效的@property属性
------------ | ------------- 
OBJC_ASSOCIATION_ASSIGN | assign 
OBJC_ASSOCIATION_RETAIN_NONATOMIC | nonatomic, retain
OBJC_ASSOCIATION_COPY_NONATOMIC | nonatomic, copy
OBJC_ASSOCIATION_RETAIN | retain
OBJC_ASSOCIATION_COPY | copy

`void objc_setAssociatedObject(id object, void* key, id value, objc_AssociationPolicy policy)` 此方法以给定的键和策略为某对象设置关联对象值。  
`id objc_getAssociatedObject(id object, void* key)` 此方法根据给定的键从某对象中获取相应的关联对象值。  
`void objc_removeAssociatedObject(id object)` 此方法移除指定对象的全部关联对象。

通常使用静态全局变量做键。

### 第11条：理解objc_msgSend的作用
在对象上调用方法是Objective-C中经常使用的功能。用OC的术语来说，这叫做“传递消息”（pass a message）。消息有“名称”（name）或“选择子”（selector），可以接受参数，而且可能还有返回值。  
C语言使用“静态绑定”（static binding），也就是说，在编译期就能决定运行时所应调用的函数。

    #include <stdio.h>
    
    void printHello() {
      printf("Hello world\n");
    }
    
    void printGoodbye() {
      printf("Goodbye, world!");
    }
    
    void doTheThing(int type) {
      if (type == 0) {
        printHello();
      } else {
        printGoodbye();
      }
    }
编译器在编译代码的时候就已经知道程序中有`printHello`和`printGoodbye`这两个函数了，所以会直接生成调用这些函数的指令。而函数地址实际上是硬编码在指令中的。

    #include <stdio.h>
    
    void printHello() {
      printf("Hello world\n");
    }
    
    void printGoodbye() {
      printf("Goodbye, world!");
    }
    
    void doTheThing(int type) {
      void (*fnc)();
      if (type == 0) {
        fnc = printHello();
      } else {
        fnc = printGoodbye();
      }
      fnc();
    }
所要调用的函数直到运行期才能确定。只有一个函数调用指令，不过待调用的函数地址无法硬编码在指令之中，而是要在运行期读取出来。  
在OC中，如果向某对象传递消息，那么就会使用动态绑定机制来决定需要调用的方法。在底层，所有方法都是普通的C语言函数，然而对象收到消息后，究竟该调用哪个方法则完全由运行期决定，甚至可以在程序运行时改变，这些特性使得OC成为一门真正的动态语言。  
<br/>
`id return = [someObject messageName:parameter];`  
`someObject`叫做“接收者”（receiver），`messageName`叫做“选择子”（selector）。选择子和参数合起来称为“消息”（message）。编译器看到此消息后，将其转换为一条标准的C语言函数调用，所调用的函数乃是消息传递机制中的核心函数，叫做`objc_msgSend`：

    void objc_msgSend(id self, SEL cmd, ...)
`objc_msgSend`函数会依据接收者与选择子的类型来调用适当的方法。为了完成此操作，该方法需要在接收者所属的类中搜寻其“方法列表”（list of methods），如果能找到与选择子名称相符的方法，就跳转到其实现代码。若是找不到，就沿着继承体系继续向上查找，等找到合适的方法之后再跳转。如果最终还是找不到相符的方法，那就执行“消息转发”（message forwarding）。见第12条。  
`objc_msgSend`会将匹配结果缓存在“快速映射表”（fast map）里面，每个类都有这样一块缓存，如果稍后还向该类发送与选择子相同的消息，那么执行起来就很快。虽然“快速执行路径”不如“静态绑定的函数调用”那么快，但有缓存也不会慢很多。实际上，“消息转发”（message dispatch）并非应用程序的瓶颈所在。  
特殊情况的消息调用函数：  

* `objc_msgSend_stret` 如果待发送的消息要返回结构体，那么可交由此函数处理。只有当CPU的寄存器能够容纳下消息返回类型时，这个函数才能处理此消息。若是返回值无法容纳于CPU寄存器中，那么就由另一个函数执行派发。此时，那个函数会通过分配在栈上的某个变量来处理消息所返回的结构体。
* `objc_msgSend_fpret` 如果消息返回的是浮点数，那么可交由此函数处理。在x86等架构上特殊处理“浮点数寄存器”
* `objc_msgSendSuper` 如果要给超类发消息，交由此函数处理。

`objc_msgSend`等函数一旦找到应该调用的方法实现之后，就会“跳转过去”。之所以能这样做，是因为OC对象的每个方法都可以视为简单的C函数，其原型如下：

    <return_type> Class_selector(id self, SEL _cmd, ...)
    
每个类都有这样一张表格，其中的指针都会指向这个函数，而选择子的名字正是查表时所用的键。

### 第12条：理解消息转发机制
若想令类能理解某条消息，必须以代码实现出具体的方法。但是，在编译期向类发送了其无法解读的消息并不会报错，因为在运行期可以继续向类中添加方法，所以编译器在编译时还无法确知类中到底会不会有某个方法实现。当对象接收到无法解读的消息后，就会启动“消息转发”（message forwarding）机制，程序员可以经由此过程告诉对象应该如何处理未知消息。  
消息转发分为两大阶段。第一阶段先征询接收者所属的类，看其能否动态添加方法，以处理当前这个“未知的选择子”（unknown selector），这叫做“动态方法解析”（dynamic method resolution）。第二阶段涉及“完整的消息转发机制”（full forwarding mechanism）。如果运行期系统已经把第一阶段执行完了，那么接收者自己就无法再以动态新增的手段来响应包含该选择子的消息了。此时，运行期系统会请求接收者以其他手段来处理消息相关的方法调用。这又细分成两小步。首先，请接收者看看有没有其他对象能处理这条消息。若有，则运行期系统会把消息转给那个对象，于是消息转发过程结束，一切如常。若没有“备援的接收者”（replacement receiver），则启动完整的消息转发机制，运行期系统会把与消息有关的全部细节都封装到NSInvocation对象中，再给接收者最后一次机会，令其设法解决当前还未处理的这条消息。

#### 动态方法解析
对象在收到无法解读的消息后，首先将调用其所属类的下列类方法：

    + (BOOL)resolveInstanceMethod:(SEL)selector
该方法的参数就是那个未知的选择子，其返回值为Boolean类型，表示这个类能否新增一个实例方法用以处理此选择子。在继续往下执行转发机制之前，本类有机会新增一个处理此选择子的方法。如果要加入尚未实现的方法是类方法，调用`resolveClassMethod:`。  
使用这种办法的前提是：相关方法的实现代码已经写好，只等着运行时插入类中即可。此方案常用来实现`@dynamic`属性。比如说，要访问Core Data框架中NSManagedObjects对象的属性时就可以这么做，因为实现这些属性所需的存取方法在编译期就能确定。  

    id autoDictionaryGetter(id self, SEL _cmd);
    void autoDictionarySetter(id self, SEL _cmd, id value);
    
    + (BOOL)resolveInstanceMethod:(SEL)sel {
      NSString* selectorString = NSStringFromSelector(sel);
      if ( /* selector is from a @dynamic property */ ) {
        if ([selectorString hasPrefix:@"set"]) {
          class_addMethod(self, sel, (IMP)autoDictionaryGetter, "v@:@");
        } else {
          class_addMethod(self, sel, (IMP)autoDictionaryGetter, "@@:");
        }
        return YES;
      }
      return [super resolveInstanceMethod:sel];
    }

#### 备援接收者
当前接收者还有第二次机会能处理未知的选择子，在这一步中，运行期系统会问它：能不能把这条消息转给其他接收者来处理。

    - (id)forwardingTargetForSelector:(SEL)selector
方法参数表示未知的选择子，若当前接收者能找到备援对象，则将其返回，若找不到，就返回nil。通过此方案，我们可以用组合(composition)来模拟出“多重继承”（multiple inheritance）的某些特性。在一个对象内部，可能还有一系列对象，该对象可经由此方法将能够处理某选择子的相关内部变量返回，这样的话，在外界看来，好像是该对象亲自处理了这些消息似的。

#### 完整的消息转发
如果转发算法已经来到这一步的话，那么唯一能做的就是启用完整的消息转发机制了。首先创建NSInvocation对象，把尚未处理的那条消息有关的全部细节都封于其中。此对象包含选择子、目标（target）及参数。在触发NSInvocation对象时，“消息派发系统”（message-dispatch system）将亲自出马，把消息指派给目标对象，通过如下方法：

    - (void)forwardInvocation:(NSInvocation *)invocation
这个方法可以实现的很简单：只需改变调用目标，使消息在新目标上得以调用即可。然而这样实现出的方法与“备援接收者”方案实现的方法等效，所以很少有人采用这么简单的实现。比较有用的实现方式是：在触发消息前，先以某种方式改变消息内容，比如追加另外一个参数，或是改换选择子，等等。  
实现此方法时，若发现某调用操作不应由本类处理，则需调用超类的同名方法。这样的话，继承体系中的每个类都有机会处理此请求，直到NSObject。如果最后调用了NSObject类的方法，那么该方法还好继而调用`doesNotRecognizeSelector:`以抛出异常，此异常表示选择子最终未能得到处理。  

![image](http://7xt1ag.com1.z0.glb.clouddn.com/Screen%20Shot%202016-06-15%20at%2019.45.47.png =600x)  
步骤越往后，处理消息的代价就越大。最好能在第一步就处理完，这样的话，运行期系统就可以将此方法缓存起来。如果这个类的实例稍后收到同名选择子，那么根本无须启动消息转发。若想在第三步里把消息转给备援的接收者，那还不如把转发操作提前到第二步。因为第三步只是修改了调用目标，这项改动放在第二步执行会更简单，不然，还得创建并处理完整的NSInvocation。

#### 动态解析的完整例子
代码见`EOCAutoDictionary`  

`@dynamic string, number, date, opaqueObject;`  
将属性设置为`@dynamic`，编译器不会自动合成实例变量和生成存储方法。  
重写父类的`resolveInstanceMethod:`方法：

    + (BOOL)resolveInstanceMethod:(SEL)sel {
      NSString* selectorString = NSStringFromSelector(sel);
      
      if ([selectorString hasPrefix:@"set"]) {
        class_addMethod(self,
                        sel,
                        (IMP)autoDictionarySetter,
                        "v@:@");
      } else {
        class_addMethod(self,
                        sel,
                        (IMP)autoDictionaryGetter,
                        "@@:");
      }
      return YES;
    }
首次在`EOCAutoDictionary`实例上访问某个属性时，运行期系统还找不到对应的选择子，因为所需的选择子既没有直接实现，又没有自动合成。假设现在要写入`opaqueObject`属性，系统就会以`setOpaqueObject:`为选择子调用上面的方法。同理，在读取该属性时，系统也会以`opaqueObject`为选择子调用上述方法。无论读写方法，都会调用`class_addMethod`方法向类中动态添加方法。第三个参数为函数指针，指向待添加的方法。最有一个参数则为函数的“类型编码”。  
在使用点语法进行读写时(`dict.object = @"some string";`)，传入的读写方法分别为`object`和`setObject:`，而在使用`setValue:forKey:`和`valueForKey:`方法时(`[dict setValue:@"some string" forKey:@"key"];`)，传入的读写方法分别为`getKey`和`setKey:`。二者在读取方法上有差别。但通常不会使用这种方法去处理`setValue:forKey:`和`valueForKey:`方法，因为只要简单地重写上述两个方法即可达到目的。
#### 备援接收者例子
    - (NSArray *)innerArray {
      if (!_innerArray) {
        _innerArray = @[@1, @2, @3];
      }
      return _innerArray;
    }
    
    - (NSString *)innerString {
      if (!_innerString) {
        _innerString = @"some string";
      }
      return _innerString;
    }
    
    - (id)forwardingTargetForSelector:(SEL)aSelector {
      if ([self.innerString respondsToSelector:aSelector]) {
        return self.innerString;
      } else if ([self.innerArray respondsToSelector:aSelector]) {
        return self.innerArray;
      }
      return [super forwardingTargetForSelector:aSelector];
    }
使用组合代替继承的做法，使用内部变量处理对应的selector，实现变相的多继承。但是此种方法不满足里氏替换原则，即不能用此方法产生的所谓子类去替换父类，只能使用新生成的类替父类处理消息。

    NSLog(@"%@", [(NSString *)dict uppercaseString]); // SOME STRING
    NSLog(@"%@", [(NSArray *)dict firstObject]); // 1
    
### 第13条：用“方法调配技术”调试“黑盒方法”
    Method lower = class_getInstanceMethod([NSString class], @selector(lowercaseString));
    Method upper = class_getInstanceMethod([NSString class], @selector(uppercaseString));
    
    method_exchangeImplementations(lower, upper);
    
    NSString* str = @"Aloha";
    
    NSLog(@"%@ %@", [str lowercaseString], [str uppercaseString]); // ALOHA aloha
    
使用`method_exchangeImplementations`方法可以交换两个方法的实现，不过通常不会交换已经写好的两个方法，因为这些方法已经实现的很好了，没有交换的必要了。通常的做法是通过这一手段为已有的方法实现添加新功能。比如说，想要在调用`lowercaseString`方法时记录某些信息，就可以通过交换实现。