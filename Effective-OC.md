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