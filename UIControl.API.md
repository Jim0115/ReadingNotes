# UIControl
#### 父类
`UIView`

#### 子类
* `UIButton`
* `UIDatePicker`
* `UIPageControl`
* `UIRefreshControl`
* `UISegmentedControl`
* `UISlider`
* `UIStepper`
* `UISwitch`
* `UITextField`

---

### UIControlState

    struct UIControlState : OptionSetType {
        init(rawValue rawValue: UInt)
        static var Normal: UIControlState { get }
        static var Highlighted: UIControlState { get }
        static var Disabled: UIControlState { get }
        static var Selected: UIControlState { get }
        static var Focused: UIControlState { get }
        static var Application: UIControlState { get }
        static var Reserved: UIControlState { get }
    }

* normal：通常状态，默认状态。被弃用但没被选中和高亮。
* highlighted：高亮状态。当一个touch event进入control的bounds中，control进入高亮状态。当touch event结束或touch event离开control的bounds，control退出高亮状态。可以通过control的`highlighted`属性获得此状态信息。  
* disabled：禁用状态。在禁用效果的control中用户交互不产生效果，control会使用淡色外观表示其被禁用。`enabled`属性。
* selected：选中状态。对于许多controls，此状态在行为和外观上不产生效果。一些子类，例如`UISegmentedControl`，使用此状态改变其外观。`selected`属性。  
* focused：聚焦状态。在一个基于焦点的导航系统中，当一个control收到焦点时进入此状态。 iOS 9加入。
* application：额外的control状态，应用可用。
* reserved：内部framework保留使用的control状态。

同时可能有多个状态。control可能根据其不同状态有不同的配置。

### UIControlEvents
    struct UIControlEvents : OptionSetType {
        init(rawValue rawValue: UInt)
        static var TouchDown: UIControlEvents { get }
        static var TouchDownRepeat: UIControlEvents { get }
        static var TouchDragInside: UIControlEvents { get }
        static var TouchDragOutside: UIControlEvents { get }
        static var TouchDragEnter: UIControlEvents { get }
        static var TouchDragExit: UIControlEvents { get }
        static var TouchUpInside: UIControlEvents { get }
        static var TouchUpOutside: UIControlEvents { get }
        static var TouchCancel: UIControlEvents { get }
        static var ValueChanged: UIControlEvents { get }
        static var PrimaryActionTriggered: UIControlEvents { get }
        static var EditingDidBegin: UIControlEvents { get }
        static var EditingChanged: UIControlEvents { get }
        static var EditingDidEnd: UIControlEvents { get }
        static var EditingDidEndOnExit: UIControlEvents { get }
        static var AllTouchEvents: UIControlEvents { get }
        static var AllEditingEvents: UIControlEvents { get }
        static var ApplicationReserved: UIControlEvents { get }
        static var SystemReserved: UIControlEvents { get }
        static var AllEvents: UIControlEvents { get }
    }

* TouchDown：按下事件
* TouchDownRepeat：重复的按下事件。重复次数记录在`UITouch`的`tapCount`中，一定大于1。
* TouchDragInside：手指在control的bounds中拖动时触发。持续拖动持续触发。
* TouchDragOutside：手指拖动出control的bounds。持续拖动持续触发。
* TouchDragEnter：手指从其他位置拖入control的bounds。仅在进入时触发一次。
* TouchDragExit：手指从control的bounds中拖动到bounds外。仅在离开时触发一次。
* TouchUpInside：最常用。手指点击control的bounds并在bounds中离开。
* TouchUpOutside：手指点击control的bounds，在bounds外离开。
* TouchCancel：系统事件取消了当前control的touch。
* ValueChanged：touch改变了control的某些值。
* PrimaryActionTriggered：button触发的语义action。iOS 9。
* EditingDidBegin：`UITextField`开始编辑。
* EditingChanged：`UITextField`中文字变化。
* EditingDidEnd：`UITextField`编辑结束。通过离开其bounds。
* EditingDidEndOnExit：`UITextField`编辑结束。
* AllTouchEvents：所有的touch event。
* AllEditingEvents：`UITextField`所有的editing event。
* ApplicationReserved：应用保留。
* SystemReserved：系统保留。
* AllEvents：所有event，包括系统event。

---

`UIControl`类定义了一类元素的通常行为，包括传递指定action，或对用户交互做出响应。Controls实现了例如button和slider等元素，这些元素被用于便利导航，收集用户输入，或控制内容。Controls使用target-action机制向App报告用户交互行为。  
不要直接创建此类的实例。`UIControl`类是你实现自定义control的拓展点。也可以创建已存在的control类的子类以拓展或修改它们的行为。例如，可以重写类中追踪touch event的方法。  
一个control的状态取决于其外观和其支撑用户交互的能力。Control可以处于数个状态中的一个，这些状态定义于`UIControlState`中。可以基于App的需要通过代码改变这些状态。例如，可以禁用一个control，防止用户与其交互。用户交互也可以改变control的状态。  

### The Target-Action Mechanism
Control使用Target-Action机制报告代码中发生的事件。TA机制简化了App中使用controls的代码。用于替代编写追踪touch event的代码，只需要编写响应指定事件的action方法。例如，编写一个响应slider值变化的方法。Control处理所有追踪touch event的工作，用于决定何时调用你的方法。  
当向control添加action method时，使用`addTarget:action:forControlEvents:`方法指定action method和定义此方法的对象。（使用IB与之类似。）Target可以是任何对象，但通常是root view包含此control的VC。如果target指定为nil，则会沿着responder chain寻找指定的方法。
可用于action的方法签名有3种形式。

    @IBAction func doSomething()
    @IBAction func doSomething(sender: UIButton)
    @IBAction func doSomething(sender: UIButton, forEvent event: UIEvent)
    
sender表示此方法调用者，event表示触发此方法的`UIEvent`对象。  
当用户以指定方式与control交互时，action method被调用。`UIControlEvents`类型定义了可以被control报告的所有类型。当配置一个control时，必须指定哪个事件将会触发你的方法。对于一个button，可能会使用`TouchDown`或`TouchUpInside`。对于一个slider，通常使用`ValueChanged`。  
当一个控制指定的事件发生时，control会调用所有关联的action method。Action method通过当前的`UIApplication`对象进行分配，这将找到合适的对象处理消息，如果需要，沿着responder chain向上传递。  

### 创建一个 UIControl 的子类
创建`UIControl`的子类将是你能够访问内置的TA机制，简化事件处理的过程。可以通过以下两种方式创建已存在control的子类，修改其行为。

1. 重写一个已存在control的`sendAction:to:forEvent:`方法用于监控和修改派发action method到target的过程。可能会使用此方法对指定object、selector或event修改派发行为。
2. 重写 `beginTrackingWithTouch:withEvent:`, `continueTrackingWithTouch:withEvent:`, `endTrackingWithTouch:withEvent:`, and `cancelTrackingWithEvent:`方法用于追踪发生在control中的touch event。可以根据追踪信息执行额外的action。总是使用这些方法追踪touch event，而不是使用定义在`UIResponder`中的方法。  


如果直接继承自`UIControl`，子类要负责设置和管理control的appearance。使用追踪方法追踪events，同时更新状态和发送action。

## API
### Configuring the Control’s Attributes
`var state: UIControlState { get }`  
control的状态，通过位掩码指定。Swift中此种类型称为`OptionSetType`。  
一个control在一个时间点可能有不止一个状态。例如，一个control可以是同时highlighted和focused的。  

`var enabled: Bool`  
control是否是可用的。  
true表示可用，false为不可用。一个可用的control能够响应user interactions，不可用的control会忽略touch event，并将自身表示成不同样子。设置此属性为true将向`state`中添加`Disable`；设置为false将移除。  
新创建的control默认为true。

`var selected: Bool`  
control是否处于被选中状态。  
大多数control不会因为此选项改变其外观，但有些会。