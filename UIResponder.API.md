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
`func touchesBegan(_ touches: Set<UITouch>, withEvent event: UIEvent?)`  
当一个或多个手指触摸落在view或window上时提醒responder。  
`UIResponder`中此方法的默认实现为空。然而UIKit中`UIResponder`的直接子类，尤其是`UIView`，将会沿着responder chain向上传递message。在自定义子类的实现中，调用super的实现即可。不要直接将此
发送给next responder。例如，如果重写此方法而没有调用super，必须同样重写处理touch event的其他方法。  
多点触控默认被禁用。设置对应view的`multipleTouchEnable`属性以启用。  

`func touchesMoved(_ touches: Set<UITouch>, withEvent event: UIEvent?)`  
当一个或多个与事件关联的手指在view或window上移动时提醒responder。  

`func touchesEnded(_ touches: Set<UITouch>, withEvent event: UIEvent?)`  
当一个或多个手指离开view或window上时提醒responder。  
当一个responder收到此消息时，应该清理在`touchesBegan:withEvent:`建立的任何状态信息。

`func touchesCancelled(_ touches: Set<UITouch>?, withEvent event: UIEvent?)`  
当一个系统事件（如低内存警告）取消了一个touch event时此方法被发送给responder。  
此方法被调用于系统中断需要取消touch event。此方法将生成一个`phase`属性为`.Cancelled`的`UITouch`对象。此方法出现的原因可能是app不再是active状态或view从window中被移除。  
当一个responder收到此消息时，应该清理在`touchesBegan:withEvent:`建立的任何状态信息。

### Responding to Motion Events
`func motionBegan(_ motion: UIEventSubtype, withEvent event: UIEvent?)`  
提示responder一个motion event已经开始。  
对于motion，iOS只会在motion启动和结束时产生通知。比如，不会汇报单独的晃动。接收motion event的responder必须是first responder。