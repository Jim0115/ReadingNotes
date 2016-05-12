# Advanced Swift
## let & var
    let a = NSMutableArray(array: [1])
    a.addObject(2)
    a = NSMutableArray(array: [1, 2, 3]) // error
    ---
    var a = NSMutableArray(array: [1])
    a.addObject(2)
    a = NSMutableArray(array: [1, 3])
    ---
    let a = NSArray(array: [1, 2])
    ---
    var b = NSArray(array: [1, 2])
    b = NSArray(array: [3])
    b = NSMutableArray(array: [1])
    (b as? NSMutableArray)?.addObject(3)
    (b as! NSMutableArray).addObject(3) // not good
    b // [1, 3]
    ---
    let a = NSMutableArray(array: [1])
    let b = a.copy() as! NSArray
    ---
    let a = NSArray(array: [1])
    let b = a.mutableCopy() as! NSMutableArray
    b.addObject(2)
    b //[1, 2]
    
`let a = NSArray`等价于Swift中`let a = [...]`    
`let a = NSMutableArray`在Swift没有对应方式  
`var a = NSArray`在Swift中没有对应方式，array的内容不可变但可以直接将`a`指向另一个`NSArray`对象甚至是子类`NSMutableArray`。但由于`a`的类型不可改变，所以需要进行强制类型转换。使用可选转换可能会比较安全。  
`var a = NSMutableArray`等价于Swift中的`var a = [...]`。

`copy()`和`mutableCopy()`类似于OC，两个方法的返回值都是`AnyObject`  
Swift使用了`copy-on-write`技术，确保copy后的值只有在真正被修改时才进行copy。

## SequenceType & CollectionType
`SequenceType`是所有集合类型的父协议，提供了for-in访问。

    for element in sequence {
        if ... some condition { break }
    }
     
    for element in sequence {
        // Not guaranteed to continue from the next element.
    }
    
`CollectionType`继承自`SequenceType`和`Indexable`，为可使用index访问的集合类型。但index未必都是Int类型，比如`CharacterView`的index类型为`String.CharacterView.Index`，`Set`的index类型为`SetIndex<Element : Hashable>`。但这二者都间接实现了`SequenceType`协议，可以使用for-in进行遍历。

## GeneratorType
    class ConstantGenerator: GeneratorType { 
      typealias Element = Int      func next() -> Element? {        return 1      }    }
实现`GeneratorType`需要两部分。1. 定义`Element`的类型 2. 实现`next()`方法。  
