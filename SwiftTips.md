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