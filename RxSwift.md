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

#### from
从一个`Sequence`，例如`Array`, `Dictionary`或`Set`创建`Observable`对象。

    let disposeBag = DisposeBag()
    
    Observable.from(["🐶", "🐱", "🐭", "🐹"])
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
        
#### create
创建一个自定义`Observable`对象。  

	let disposeBag = DisposeBag()
    
    let myJust = { (element: String) -> Observable<String> in
        return Observable.create { observer in
            observer.on(.next(element))
            observer.on(.completed)
            return Disposables.create()
        }
    }
        
    myJust("🔴")
        .subscribe { print($0) }
        .addDisposableTo(disposeBag)
        
#### range
对range中元素按照数字顺序发出事件。

    let disposeBag = DisposeBag()
    
    Observable.range(start: 1, count: 10)
        .subscribe { print($0) }
        .addDisposableTo(disposeBag)
        
#### repeatElement
创建一个无限重复给定元素的`Observable`序列。

    let disposeBag = DisposeBag()

    Observable.repeatElement("🔴")
        .take(3) // 取Observabke的前n个元素
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
        
#### generate
condition为true时，继续执行iterate。

    let disposeBag = DisposeBag()
    
    Observable.generate(
            initialState: 0,
            condition: { $0 < 3 },
            iterate: { $0 + 1 }
        )
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)

#### deferred
对每个subscriber都创建一个新的`Observable`对象。

var count = 1
    
    let deferredSequence = Observable<String>.deferred {
        print("Creating \(count)")
        count += 1
        
        return Observable.create { observer in
            print("Emitting...")
            observer.onNext("🐶")
            observer.onNext("🐱")
            observer.onNext("🐵")
            return Disposables.create()
        }
    }
    
    deferredSequence
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
    
    deferredSequence
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
        
#### error
创建一个只包含一个error元素的`Observable`。

    let disposeBag = DisposeBag()
        
    Observable<Int>.error(TestError.test)
        .subscribe { print($0) }
        .addDisposableTo(disposeBag)

#### doOn
对指定的事件的发生定义一些副作用。

    let disposeBag = DisposeBag()
    
    Observable.of("🍎", "🍐", "🍊", "🍋")
        .do(onNext: { print("Intercepted:", $0) }, onError: { print("Intercepted error:", $0) }, onCompleted: { print("Completed")  })
        .subscribe(onNext: { print($0) })
        .addDisposableTo(disposeBag)
        
### Working with Subjects
Subject既可以是Observer也可以是Observable。所以，Subject可以订阅一个或多个Observable，也可以转发其所订阅的Observable的item，同时也可以发送新的item。

#### PublishSubject
在observer订阅时广播所有新的event。

    let disposeBag = DisposeBag()
    let subject = PublishSubject<String>()
    
    subject.addObserver("1").addDisposableTo(disposeBag)
    subject.onNext("🐶")
    subject.onNext("🐱")
    
    subject.addObserver("2").addDisposableTo(disposeBag)
    subject.onNext("🅰️")
    subject.onNext("🅱️")

Subscription: 1 Event: next(🐶)  
Subscription: 1 Event: next(🐱)   
Subscription: 1 Event: next(🅰️)  
Subscription: 2 Event: next(🅰️)   
Subscription: 1 Event: next(🅱️)   
Subscription: 2 Event: next(🅱️)  
不会发送订阅前的信号。

#### ReplaySubject
对所有subscriber广播新事件，同时使用`bufferSize`属性指定对新subscriber发送之前消息的数量。

    let disposeBag = DisposeBag()
    let subject = ReplaySubject<String>.create(bufferSize: 1)
    
    subject.addObserver("1").addDisposableTo(disposeBag)
    subject.onNext("🐶")
    subject.onNext("🐱")
    
    subject.addObserver("2").addDisposableTo(disposeBag)
    subject.onNext("🅰️")
    subject.onNext("🅱️")
    
Subscription: 1 Event: next(🐶)  
Subscription: 1 Event: next(🐱)  
Subscription: 2 Event: next(🐱)   
Subscription: 1 Event: next(🅰️)  
Subscription: 2 Event: next(🅰️)  
Subscription: 1 Event: next(🅱️)  
Subscription: 2 Event: next(🅱️)  
由于`bufferSize`为1，所以observer2会收到event🐱。

#### BehaviorSubject
对所有subscriber广播新事件，同时会有一个初始事件。如果在订阅时没有已发送的事件，会发送初始事件，否则发送上一条事件。

    let disposeBag = DisposeBag()
    let subject = BehaviorSubject(value: "🔴")
  
    subject.addObserver("1").addDisposableTo(disposeBag)
    subject.onNext("🐶")
    subject.onNext("🐱")
    
    subject.addObserver("2").addDisposableTo(disposeBag)
    subject.onNext("🅰️")
    subject.onNext("🅱️")
    
    subject.addObserver("3").addDisposableTo(disposeBag)
    subject.onNext("🍐")
    subject.onNext("🍊")
    
Subscription: 1 Event: next(🔴)  
Subscription: 1 Event: next(🐶)  
Subscription: 1 Event: next(🐱)   
Subscription: 2 Event: next(🐱)   
Subscription: 1 Event: next(🅰️)   
Subscription: 2 Event: next(🅰️)   
Subscription: 1 Event: next(🅱️)   
Subscription: 2 Event: next(🅱️)  
Subscription: 3 Event: next(🅱️)   
Subscription: 1 Event: next(🍐)   
Subscription: 3 Event: next(🍐)  
Subscription: 2 Event: next(🍐)   
Subscription: 1 Event: next(🍊)  
Subscription: 3 Event: next(🍊)  
Subscription: 2 Event: next(🍊)    
这里subscriber1收到了初始事件，而subscriber2和3收到了订阅前的上一事件。

### Variable
`Variable`封装了一个`BehaviorSubject`，所以其也会发送初始事件或最近的事件。同时`Variable`也维护当前值的状态。`Variable`不会发送错误事件，但会在deinit时自动发送完成事件。

    let variable = Variable("🔴")
    
    variable.asObservable().addObserver("1").addDisposableTo(disposeBag)
    variable.value = "🐶"
    variable.value = "🐱"
    
    variable.asObservable().addObserver("2").addDisposableTo(disposeBag)
    variable.value = "🅰️"
    variable.value = "🅱️"
    
Subscription: 1 Event: next(🔴)  
Subscription: 1 Event: next(🐶)  
Subscription: 1 Event: next(🐱)  
Subscription: 2 Event: next(🐱)  
Subscription: 1 Event: next(🅰️)  
Subscription: 2 Event: next(🅰️)  
Subscription: 1 Event: next(🅱️)  
Subscription: 2 Event: next(🅱️)  
Subscription: 1 Event: completed  
Subscription: 2 Event: completed    

在一个`Variable`实例上调用`asObserver`方法获取其封装的`BehaviorSubject`序列