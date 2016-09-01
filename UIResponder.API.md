## UIResponder API
`UIResponder`定义了一个对象响应和处理事件的接口。它是`UIApplication`，`UIView`及其子类（包括`UIWindow`）的父类。这些类通常被称为responder。  
iOS中有两大类事件：touch event和motion event。处理touch事件的主要方法有`touchesBegan:withEvent:`, `touchesMoved:withEvent:`, `touchesEnded:withEvent:`, and `touchesCancelled:withEvent:`.这些方法的参数关联touch和event，尤其是新的touch或产生改变的touch，这允许responder追踪和处理touch。任何时候手指触碰屏幕，在屏幕上拖动，离开屏幕，都将生成一个`UIEvent`对象。这个event对象包含`UITouch`对象，代表屏幕上所有的手指或刚刚离开的手指。  
iOS 3.0引入了motion events，尤其是晃动设备的motion。处理这些事件的方法有 `motionBegan:withEvent:`, `motionEnded:withEvent:`, and `motionCancelled:withEvent:`。  
iOS 4.0中，`UIResponder`中添加了`remoteControlReceivedWithEvent:`方法用于处理远程控制事件。  

### Managing the Responder Chain  
`func nextResponder() -> UIResponder?`  
返回下一个responder，若无则返回nil。  
`UIResponder`不会自动储存或设置next responder，默认返回nil。子类必须重写此方法以设置next responder。`UIView`返回持有其的VC或其superview。`UIViewController`返回其view的superview（通常是UIWindow）。`UIWindow`返回application对象。`UIApplication`对象返回nil。  

`func isFirstResponder() -> Bool`  
self是否为first responder。  

`func canBecomeFirstResponder() -> Bool`  
询问对象其能否成为first responder。默认返回false。  
子类必须重写此方法，获得称为first responder的能力。  
对于不在view hierarchy中的view，此方法返回的结果不确定。因此不允许对这种view发送此消息。  

`func canBecomeFirstResponder() -> Bool`  
提醒self将成为其window的first responder。  
返回true表示self接受此状态，false表示拒绝此状态。默认返回true。  
子类可以重写此方法用于更新状态或执行某些操作。比如在textfield被选中时选中其所有文字。  
调用此方法时一个responder成为first responder。在view上调用此方法的条件是view是某个view hierarchy的一部分。判断的方法是view的`window`属性是否返回一个`UIWindow`对象。  

`func canResignFirstResponder() -> Bool`  
self是否愿意退出first responder状态。  
默认返回true。在一个textfield正在编辑中时可能重写此方法返回false直到其完成编辑。  

`func resignFirstResponder() -> Bool`  
提醒self，其被要求退出其window的first responder状态。  
默认返回true，表示已经退出。如果重写此方法，必须调用super。

### Managing Input Views
`var inputView: UIView? { get }`  
当self为first responder状态时，需要显示的自定义input view。  
此属性通常用于提供一个view替换系统为`UITextField`和`UITextView`对象提供的键盘。  
此属性的返回值为nil。如果一个responder需要一个自定义view用于从用户处获取输入，需要重新生命此属性为read-write，用其管理自定义输入view。当self成为first responder时，会自动present这个view。同样，在退出first responder时，会自动dismiss这个view。  

`var inputViewController: UIInputViewController? { get }`  
当self为first responder状态时，需要显示的自定义input view controller。  
其余属性同上，但优先级较inputView低。  

`func reloadInputViews()`  
当self为first responder时，更新自定义 input view。  
使用此方法更新自定义input view。view将被立即替换，没有动画。如果self不是first responder，此方法无效果。

### Responding to Touch Events  


