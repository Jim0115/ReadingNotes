# 储存属性、计算属性和lazy属性

## 储存属性
### Swift
    var a = 10
    var b = "Hello World!"
    
### Objective-C
    NSInteger a = 10;
    NSString* b = @"Hello World!";
    
属性将值储存在对象中。

## 计算属性
### Swift
    var a = 10
    var b = 20
    var sum: Int { return a + b }
    
    print(sum)  // 30
    a = 0
    print(sum) // 20
    
### Objective-C
    NSInteger a = 10;
    NSInteger b = 20;
    
    - (NSInteger)sum {
      return a + b;
    }
    
    NSLog(@"%ld", self.sum); // 30
    a = 0;
    NSLog(@"%ld", self.sum); // 20
    
计算属性不储存值，每次获取时从其他属性中计算出值。Swift中计算属性不能使用let声明。

## lazy属性（延迟储存属性）
lazy属性可以看做储存属性和计算属性的结合。第一次使用的时候为计算属性，之后为储存属性。
### Swift
    class Foo {
      var a = 10
      let b = 20
      
      lazy var lazySum: Int = {
        return self.a + self.b
      }()
    }
    
Swift中lazy属性只能使用于类或结构体的属性中，且必须为var。

    let f = Foo()
    f.a // 10
    f.b // 20
    f.lazySum // 30
    f.a = 0 
    f.lazySum // 30
    f.lazySum = 5
    
在调用过一次后lazy属性即可视为一般的储存属性。

    let f = Foo()

    f.lazySum = 0
    f.lazySum // 0
    
在lazy属性被调用前先对其进行赋值，则其直接变为一般的储存属性。

### Objective-C
在OC中lazy属性即延迟初始化，是OC中一种常用的初始化方法。属性只有在被调用的时候才被初始化。

    - (NSInteger)lazySum {
      if (!_lazySum) {
        _lazySum = self.a + self.b;
      }
      return _lazySum;
    }
    
    self.a = 10;
    self.b = 20;
    NSLog(@"%ld", self.lazySum); // 30
    self.a = 0;
    NSLog(@"%ld", self.lazySum); // 30
    
与Swift中相同，若在get前先对其进行set，则其变为一般的储存属性。
    
      self.a = 10;
      self.b = 20;
      self.lazySum = 40;
      NSLog(@"%ld", self.lazySum); // 40
      
在OC中，0即为false。使用延迟初始化时有可能被计算多次。并不会像Swift中只被调用一次。