## UIResponder API
`UIResponder`定义了一个对象响应和处理事件的接口。它是`UIApplication`，`UIView`及其子类（包括`UIWindow`）的父类。这些类通常被称为responder。  
iOS中有两大类事件：touch event和motion event。处理touch事件的主要方法有`touchesBegan:withEvent:`, `touchesMoved:withEvent:`, `touchesEnded:withEvent:`, and `touchesCancelled:withEvent:`.这些方法的参数关联touch和event，尤其是新的touch或产生改变的touch，这允许responder追踪和处理touch。任何时候手指触碰屏幕，在屏幕上拖动，离开屏幕，都将生成一个`UIEvent`对象。这个event对象包含`UITouch`对象，代表屏幕上所有的手指或刚刚离开的手指。  
iOS 3.0引入了motion events，尤其是晃动设备的motion。处理这些事件的方法有 `motionBegan:withEvent:`, `motionEnded:withEvent:`, and `motionCancelled:withEvent:`。