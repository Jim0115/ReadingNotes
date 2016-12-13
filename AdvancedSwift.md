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

# Swift 3.0 2016.12.11
### 集合协议 遍历 Iterator 序列 Sequence 和集合Collection
#### 遍历 Iterator
	
	protocol IteratorProtocol {
	  associatedtype Element
	  public mutating func next() -> Self.Element?
	}
	
`associatedtype`指定的关联类型可以通过`typealias`显式指定，也可以通过`next()`函数的返回值确定。  
Iterator是没有值语义的，即每个Iterator的值只能被循环一遍。  

	class PrefixIterator : IteratorProtocol {
	  let string: String
	  var offSet: String.Index
	  
	  init(string: String) {
	    self.string = string
	    offSet = string.startIndex
	  }
	  
	  func next() -> String? {
	    guard offSet < string.endIndex else { return nil }
	    let result = string[string.startIndex...offSet]
	    offSet = string.index(after: offSet)
	    return result
	  }
	}
	
	let pi = PrefixIterator(string: "Hello World!")

	while let prefix = pi.next() {
	  print(prefix)
	}
一个简单的Iterator示例。如果想要再次迭代，就必须生成一个新的Iterator。这就是Iterator是class而非struct的原因。  

#### 序列 Sequence
`Sequence`是构建在`Iterator`之上的一个协议，需要指定一个特定类型的Iterator，并提供一个方法用于创建一个新的Iterator。  

	protocol Sequence {
	  associatedtype Iterator : IteratorProtocol
	  public func makeIterator() -> Self.Iterator
	}

一个基于`PrefixIterator`的sequence实现。

	struct PrefixSequence : Sequence {
	  let string: String
	  func makeIterator() -> PrefixIterator {
	    return PrefixIterator(string: string)
	  }
	}
对于实现了`Sequence`协议的对象，可以使用for-in循环对其进行遍历。

	for prefix in PrefixSequence(string: "Hello World!") {
	  print(prefix)
	}
	
而实际上，for-in遍历只是对Iterator的一种简写：

	var generator = PrefixSequence(string: "Hello").generate()
	while let prefix = generator.next() {
	  print(prefix)
	}
	
#### 集合 Collection
集合是基于序列Sequence的更高层级的协议，为序列添加了可重复迭代，通过索引访问元素的能力。  


	/// 一个能够将元素入队和出队的类型
	protocol QueueType {
	  /// 在 `self` 中所持有的元素的类型
	  associatedtype Element
	  /// 将 `newElement` 入队到 `self`
	  mutating func enqueue(newElement: Element)
	  /// 从 `self` 出队一个元素
	  mutating func dequeue() -> Element?
	}
	
实现一个Queue：

	struct Queue<Element> : QueueType {
	  fileprivate var left = [Element]()
	  fileprivate var right = [Element]()
	  
	  mutating func enqueue(newElement: Element) {
	    right.append(newElement)
	  }
	  
	  mutating func dequeue() -> Element? {
	    if left.isEmpty && right.isEmpty { return nil }
	    if left.isEmpty {
	      left = right.reversed()
	      right.removeAll()
	    }
	    return left.removeLast()
	  }
	}
	
让自定义的队列类`Queue`实现集合Collection协议。实际上，Collection协议是对Sequence和Indexable协议的一个组合。Collection协议中很多方法都有其默认实现，因此，实现一个Collection只需要实现其startIndex和endIndex，通过下标获取对应位置元素，以及对指定的Index找到其下一个Index。  

	extension Queue : Collection {
	  var startIndex: Int { return 0 }
	  var endIndex: Int { return left.count + right.count }
	  
	  subscript(idx: Int) -> Element {
	    if idx < left.endIndex {
	      return left[left.count - idx + 1]
	    } else {
	      return right[idx - left.count]
	    }
	  }
	  
	  func formIndex(after i: inout Int) {
	    i += 1
	  }
	  func index(after i: Int) -> Int {
	    return i + 1
	  }
	}  

#### 遵守 ExpressibleByArrayLiteral 协议
	extension Queue : ExpressibleByArrayLiteral {
	  init(arrayLiteral elements: Element...) {
	    // self.left = elements.reversed()
	    // self.right = [] 使用时会出现段错误 11，应调用init方法
	    self.init(left: elements.reversed(), right: [])
	  }
	}

通过实现此协议，可以直接使用一个数组常量来构建一个Queue。  
	
	let q: Queue = [1, 2, 3]
	// 这里q的类型为Queue<Int>
	
#### 遵守 RangeReplaceableCollection 协议
遵守此协议表示可以使用一个新的集合替换当前集合的某个部分（Range）。通过实现此协议，可以得到许多有用的方法比如：`append`, `appendContentsOf`, `removeAtIndex`, `removeRange`, `insertAtIndex`, `removeAll`等方法。  

	extension Queue : RangeReplaceableCollection {
	  mutating func reserveCapacity(_ n: Int) {
	    // 避免在申请内存时不必要的元素复制
	    return
	  }
	  
	  mutating func replaceSubrange<C>(_ subrange: Range<Int>, with newElements: C) where C : Collection, C.Iterator.Element == Element {
	    right = left.reversed() + right
	    left.removeAll(keepingCapacity: true)
	    right.replaceSubrange(subrange, with: newElements)
	  }
	}
	
### 索引
通过enum实现一个单项链表

	enum List<Element> {
	  case End
	  indirect case Node(value: Element, next: List<Element>)
	}
	
	
	extension List {
	  func cons(x: Element) -> List {
	    // 用于向当前链表前添加节点，即创建一个节点并在其后链接已有链表
	    return .Node(value: x, next: self)
	  }
	}
	
一个栈的协议：

	/// 一个进栈和出栈都是常数时间操作的后进先出 (LIFO) 栈
	protocol StackType {
	  associatedtype Element
	  /// 将 `x` 入栈到 `self` 作为栈顶元素
	  ///
	  /// - 复杂度：O(1).
	  mutating func push(x: Element)
	  /// 从 `self` 移除栈顶元素，并返回它
	  /// 如果 `self` 是空，返回 `nil`
	  ///
	  /// - 复杂度：O(1)
	  mutating func pop() -> Element?
	}
	
对List实现StackType协议：

	extension List : StackType {
	  mutating func pop() -> Element? {
	    switch self {
	    case .End:
	      return nil
	    case .Node(let value, let next):
	      self = next
	      return value
	    }
	  }
	  
	  mutating func push(x: Element) {
	    self = self.cons(x: x)
	  }
	}
	
#### 让 List 遵守 Sequence
	extension List : Sequence {
	  func makeIterator() -> AnyIterator<Element> {
	    var current = self
	    return AnyIterator {
	      // 下一个出栈的元素 若当前为.End 返回nil
	      current.pop()
	    }
	  }
	}

遵守 ExpressibleByArrayLiteral

	extension List : ExpressibleByArrayLiteral {
	  init(arrayLiteral elements: Element...) {
	    self = elements.reversed().reduce(.End) { $0.cons(x: $1) }
	  }
	}

## 可选值 Optional
#### 强制解包的时机
当你能确定你的某个值不可能是nil时可以使用强制解包，应当希望它不巧是nil时程序直接挂掉。  

## 结构体和类
主要不同点：

- 结构体和枚举是值类型，类是引用类型。
- 内存的管理方式不同。结构体可以被直接持有和访问，而类的实例只能通过引用间接访问。结构体不会被引用，但是会被复制。
- 除非一个类被声明为`final`，否则其都是可以被继承的。而结构体和枚举是不能被继承的。

## 函数
1. 函数可以像`Int`或`String`那样被赋值给变量，也可以作为另一个函数的输入参数或返回值。  
2. 函数能够“捕获”存在其作用域之外的变量。
3. 有两种方法可以创造函数。`func`关键字和闭包。

