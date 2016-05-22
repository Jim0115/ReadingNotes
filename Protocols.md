# Protocols
![image](http://7xt1ag.com1.z0.glb.clouddn.com/Screen%20Shot%202016-05-22%20at%2019.12.31.png =450x)

### Equatable 能力
Instances of conforming types can be compared for value equality using operators == and !=.  
实现了`Equatable`协议的类的对象可以进行判断是否相等。
    
    class Foo: Equatable {
      var value: Int
      init() {
        value = 0
      }
    }
    
    func ==(lhs: Foo, rhs: Foo) -> Bool {
      return lhs.value == rhs.value
    }
    
    var a = Foo()
    var b = Foo()
    a == b  // true
    a.value = 1
    a == b // false
    
### Comparable 能力
Instances of conforming types can be compared using relational operators, which define a strict total order.  
`Comparable`继承自`Equatable`，实现了`Comparable`的类的对象可以进行比较判断大小。  
`Comparable`包含`>` `>=` `<` `<=`操作，实现时只需要实现`==`和`<`，其他的自动生成。

    class Foo: Comparable {
      var value: Int
      init() {
        value = 0
      }
    }
    
    func ==(lhs: Foo, rhs: Foo) -> Bool {
      return lhs.value == rhs.value
    }
    
    func <(lhs: Foo, rhs: Foo) -> Bool {
      return lhs.value < rhs.value
    }
    
    var a = Foo()
    var b = Foo()
    
    a.value = 3
    
    a < b // false
    a <= b // false
    a > b // true
    a >= b // true
    
### IntegerLiteralConvertible 能力
Conforming types can be initialized with integer literals.  
实现协议的类型可以由`IntegerLiteralType`字面量初始化。  

    class Foo: IntegerLiteralConvertible {
      var value: Int
      typealias IntegerLiteralType = Int
      init() {
        value = 0
      }
      required init(integerLiteral value: IntegerLiteralType) {
        self.value = value
      }
    }
    
    let f = Foo(integerLiteral: 10)
    f.value  // 10
---
typealias 为一个已经存在的类型取个别名  
associatedtype 在协议中作为一个类型的占位名称

    protocol Foo {
      associatedtype T : Comparable, CustomStringConvertible
      func foo(value: T) -> Bool
    }
    
    class Bar: Foo {
      typealias T = Int
      func foo(value: T) -> Bool {
        print(value)
        return true
      }
    }

### SignedNumberType 是什么
Instances of conforming types can be subtracted, arithmetically negated, and initialized from 0.