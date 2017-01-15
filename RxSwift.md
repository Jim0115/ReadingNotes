# RxSwift
## 为什么使用RxSwift
在iOS中，对某个事件的响应有多种处理方式。对于`UIControl`，需要使用action去响应。对于notification，需要创建Observer。对于异步请求，需要使用closure回调。对于某个值的变化，需要使用KVO去观察。Rx试图使用统一的框架去处理所有不同类型事件的响应。  
RxSwift是Reactive的Swift实现。

## 基础概念
### 任何Observable实例只是一个Sequence
`Observable`对`Sequence`的关键优势是可以异步的接受元素。这是RxSwift的精髓。

- `Observable`等价于`Sequence`
- `subscribe`方法等价于`makeIterator`
- `subscribe`方法订阅一个`Obervable`对象，自动会收到`Observable`对象发出的事件，而不必调用`next`方法。

如果一个`Observable`对象发出了一个事件(`Event.next(Element)`)，它可以继续发出更多事件。然而，如果发出了一个错误事件(`Event.error(ErrorType)`)或完成事件(`Event.completed`)，则其不能再继续发出事件。

### Observable和Observer(subscriber)
如果一个Observable没有subscriber，其closure将不会被执行。  
`subscribe`方法返回一个`Disposable`对象

### 创建和订阅`Observable`
#### never
创建一个不发出任何事件的sequence

	let disposeBag = DisposeBag()
    let neverSequence = Observable<String>.never()
    
    let neverSequenceSubscription = neverSequence
        .subscribe { _ in
            print("This will never be printed")
    }
    
    neverSequenceSubscription.addDisposableTo(disposeBag)
    
#### empty
创建一个只会发出一个`completed`事件的`Observable`

    let disposeBag = DisposeBag()
    
    Observable<Int>.empty()
        .subscribe { event in
            print(event) // complete
        }
        .addDisposableTo(disposeBag)

#### just
创建一个只有单个元素的`Observable`

	let disposeBag = DisposeBag()
    
    Observable.just("🔴")
        .subscribe { event in
            print(event) 
        }
        .addDisposableTo(disposeBag)
    // next(🔴) 
    // completed
    
#### of
创建一个拥有固定元素数量的`Observable`

    let disposeBag = DisposeBag()
    
    Observable.of("🐶", "🐱", "🐭", "🐹")
        .subscribe(onNext: { element in
            print(element)
        })
        .addDisposableTo(disposeBag)
    // 🐶
	// 🐱
	// 🐭
	// 🐹
	
`subscribe(onNext:)`方法会忽略error和completed。对应的还有`subscribe(onError:)`和`subscribe(onCompleted:)`，分别针对error和completed进行处理。