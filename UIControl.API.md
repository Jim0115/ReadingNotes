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
大多数control不会因为此选项改变其外观，但有些会。比如`UISegmentedControl`会追踪被选中的segment。  
默认为false。  

`var highlighted: Bool`  
control是否处于高亮状态。  
Control会响应touch event，自动设置和清除此状态。  
默认为false。  

`var contentVerticalAlignment: UIControlContentVerticalAlignment`  
在垂直方向上内容在control的bounds中的对齐方式。  
对于那些饱含文字或图片内容的controls，使用此选项在control的bounds内对齐content。并不是所有的子类都有可以对齐的content，由子类负责如何应用此属性。默认值为`Top`。

`var contentHorizontalAlignment: UIControlContentHorizontalAlignment`  
水平方向上对齐方式。  

### Accessing the Control’s Targets and Actions
`func addTarget(_ target: AnyObject?, action action: Selector, forControlEvents controlEvents: UIControlEvents)`  
将control与target对象和action method关联。  

* `target`：其action被调用的对象。如果指定为nil，UIKit会沿着responder chain寻找能够响应指定action的对象，将消息发送给此对象。
* `action`：一个关联action method的selector。不能为nil。
* `controlEvents`：指定当何种事件发生时调用action。至少一个。

此方法可以调用多次，想control添加不同的target和action。多次调用此方法，传入相同的参数并不会产生影响。Control维护一个列表用于保存target，action和event。  
Control不会retain参数target。所以需要自己保留target的强引用。  
将`controlEvents`参数指定为0不会阻止之前注册的target和action。想要停止发送事件，调用`removeTarget:action:forControlEvents:`方法。  

`func removeTarget(_ target: AnyObject?, action action: Selector, forControlEvents controlEvents: UIControlEvents)`  
停止向指定的target传递事件。  
使用此方法阻止control向target传递event。如果指定了一个有效的`target`，则会停止指定event的所有action。如果指定`target`为nil，则停止对应event的所有target的所有action。  
尽管此方法不会考虑`action`参数，仍需要指定合适的值。如果指定的TA组合不再有任何有效的control event关联，control会清除其内部对应的数据结构。

`func actionsForTarget(_ target: AnyObject?, forControlEvent controlEvent: UIControlEvents) -> [String]?`  
返回当指定event发生时，参数target将会被调用的所有action。  
返回一个包含字符串的数组，字符串对应selector。若无对应selector，返回nil。  

`func allControlEvents() -> UIControlEvents`  
返回control已关联的action的所有events。  

`func allTargets() -> Set<NSObject>`  
返回此control关联的所有target对象。  

### Triggering Actions
`func sendAction(_ action: Selector, to target: AnyObject?, forEvent event: UIEvent?)`  
调用指定的action。  
此方法使用提供的信息并将其传递到`UIApplication`单例用于派发。如果提供了有效的target，app会在此target上调用action method。如果target为nil，app会搜索responder chain，找到定义此方法的对象。  
子类可以重写此方法用于监控和修改action的派发行为。如果想要继续执行action method，实现应该调用super。

`func sendActionsForControlEvents(_ controlEvents: UIControlEvents)`  
调用与指定事件相关联的action method。  
当你想让control执行与特定事件关联的action时，调用此方法。此方法会遍历control的注册的target和action，对每个关联参数事件的TA组合调用`sendAction:to:forEvent:`。

### Tracking Touches and Redrawing Controls
`func beginTrackingWithTouch(_ touch: UITouch, withEvent event: UIEvent?) -> Bool
`    
当一个touch event进入control的bounds时被调用。  
返回true表示control应该继续追踪touch event。返回false停止追踪。  
默认实现返回true。子类可以重写此方法，用其响应事件。使用参数提供的event判断control的那个部分被点击，设置所有初始状态信息。

`func continueTrackingWithTouch(_ touch: UITouch, withEvent event: UIEvent?) -> Bool`  
当关联control的touch event更新时被调用。  
返回true表示继续追踪事件，返回false表示停止。  
此方法将在touch event在control的bounds里被追踪时持续调用。默认实现返回true。  

`func endTrackingWithTouch(_ touch: UITouch?, withEvent event: UIEvent?)`  
在与control关联的touch event结束时被调用。  
如果重写此方法，必须调用super。  

`func cancelTrackingWithEvent(_ event: UIEvent?)`  
告诉control停止追踪给定event。  
当一个与control相关的touch event被取消时control调用此方法。默认实现取消任何正在进行的追踪并更新control的状态信息。子类重写此方法用于执行任何取消操作。同时，清除与追踪event相关的数据。  
如果重写此方法，必须调用super。  

`var tracking: Bool { get }`  
control当前是否正在追踪touch event。  
当正在追踪touch event时，control将此属性设置为true。当追踪结束或被取消时，control将此属性设为false。  

`var touchInside: Bool { get }`  
一个被追踪的touch event当前是否在control的bounds中。  
当正在追踪一个touch event时，control会更新此属性指示最近的touch是否还在control的bounds中。control使用此信息触发特定的事件。比如，`TouchDragEnter`或`TouchDragExit`等。