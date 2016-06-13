# SwiftTips e-book
### 操作符
Swift支持操作符重载，语法如下所示：

    struct Vector2D {
      var x = 0.0
      var y = 0.0
    }
    
    let v1 = Vector2D(x: 1, y: 2)
    let v2 = Vector2D(x: 2, y: 3)
    
    let v3 = Vector2D(x: v1.x + v2.x, y: v1.y + v2.y)
    
    func +(lhs: Vector2D, rhs: Vector2D) -> Vector2D {
      return Vector2D(x: lhs.x + rhs.x, y: lhs.y + rhs.y)
    }
    
    let v4 = v1 + v2
    
由于`+`是Swift中已被声明过的操作符，所以直接进行重写即可，若要新声明一个操作符，需要预先定义：
    
    infix operator +* { // 内积
      associativity none
      precedence 160
    }
    
    func +*(lhs: Vector2D, rhs: Vector2D) -> Double {
      return lhs.x * rhs.x + rhs.y * rhs.y
    }
    
    let v3 = v1 +* v2
    
`infix`表示中位操作符，表示符号作用于两个操作数之间。对应的还有`prefix`、`postfix`  
`associativity`表示结合律的计算顺序。加减法的顺序都是`left`，即多个加减法同时出现时，按照从左到右的顺序进行计算。  
`precedence`表示运算的优先级。Swift中乘除法的优先级是150，加减法的优先级是140。  
操作符应在全局范围定义，不能声明一个局部操作符。操作符应尽量作为其他方法的一个简便写法，而不应该用其实现唯一的逻辑。不要滥用这个特性。

### 接口和类方法中的`Self`
    protocol Copyable {
      func copy() -> Self
    }
    
    class Foo: Copyable {
      var num = 1
      
    //  func copy() -> Self {
    //    let foo = Foo()
    //    foo.num = num
    //    return foo
    //  }
      
      func copy() -> Self {
        let foo = self.dynamicType.init()
        foo.num = num
        return foo
      }
      
      required init() { }
    }
    
### Protocol Extension
从Swift 2 开始，extension可以用于协议，拓展中实现的方法作为实现拓展的类型的默认实现。

    protocol FooProtocol {
    }
    
    extension FooProtocol {
      func foo() {
        print("foo")
      }
    }
    
    class A: FooProtocol {
    
    }
    
    A().foo() // "foo"
    
如果在接口拓展中，实现了接口之外的方法。

    protocol A1 {
      func foo1() -> String
    }
    
    extension A1 {
      func foo1() -> String {
        return "foo1"
      }
      
      func foo2() -> String {
        return "foo2"
      }
    }
    
    class B1: A1 {
      func foo1() -> String {
        return "new foo1"
      }
      
      func foo2() -> String {
        return "new foo2"
      }
    }
    
    let b1 = B1()
    
    b1.foo1() // new foo1
    b1.foo2() // new foo2
    
    let b2 = B1() as A1
    
    b2.foo1() // new foo1
    b2.foo2() // foo2
    
方法的调用遵循以下规则：

* 如果类型推断得到的是实际类型：类中的实现将被调用；如果类型中没有实现，那么接口扩展中的默认实现将被使用
* 如果类型推断得到的是接口
    * 如果方法在接口中进行了定义，那么类型中的实现将被调用；如果类型中没有实现，那么接口扩展中的默认实现将被使用
    * 如果方法没有在接口中定义，直接调用默认实现
    
### 使用GCD实现延迟执行的实现
    typealias Task = (cancel: Bool) -> Void
    
    func delay(time: NSTimeInterval, task: () -> ()) -> Task? {
      func dispatch_later(block: () -> ()) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, Int64(time * Double(NSEC_PER_SEC))), dispatch_get_main_queue(), block)
      }
      
      var closure: dispatch_block_t? = task
      var result: Task?
      
      let delayedClosure: Task = {
        cancel in
        if let internalClosure = closure {
          if cancel == false {
            dispatch_async(dispatch_get_main_queue(), internalClosure)
          }
        }
        closure = nil
        result = nil
      }
      
      result = delayedClosure
      
      dispatch_later { 
        if let delayedClosure = result {
          delayedClosure(cancel: false)
        }
      }
      
      return result
    }
    
    func cancel(task: Task?) {
      task?(cancel: true)
    }
    
### 使用Category(Extension)给已有的类添加成员变量
Obj-C: 导入runtime.h头文件   
`UIButton+Name.h`

    #import <UIKit/UIKit.h>
    
    @interface UIButton (Name)
    
    @property (nonatomic, copy) NSString* name;
    
    @end

`UIButton+Name.m`

    #import "UIButton+Name.h"
    #import <objc/runtime.h>
    
    @implementation UIButton (Name)
    
    // void* key;
    const char key;
    
    - (NSString *)name {
      return objc_getAssociatedObject(self, &key);
    }
    
    - (void)setName:(NSString *)name {
      objc_setAssociatedObject(self, &key, name, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    }

    @end
使用前需要导入类簇头文件。  

---
Swift:
    
    var key: Void?

    extension UIButton {
      var name: String? {
        get {
          return objc_getAssociatedObject(self, &key) as? String
        }
        set {
          objc_setAssociatedObject(self, &key, newValue, .OBJC_ASSOCIATION_RETAIN_NONATOMIC)
        }
      }
    }