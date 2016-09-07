## UIControl
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


---

`UIControl`类定义了一类元素的通常行为，包括传递指定action，或对用户交互做出响应。Controls实现了例如button和slider等元素，这些元素被用于便利导航，收集用户输入，或控制内容。Controls使用target-action机制向App报告用户交互行为。  
不要直接创建此类的实例。`UIControl`类是你实现自定义control的拓展点。也可以创建已存在的control类的子类以拓展或修改它们的行为。例如，可以重写类中追踪touch event的方法。  
一个control的状态取决于其外观和其支撑用户交互的能力。Control可以处于数个状态中的一个，这些状态定义于`UIControlState`中。可以基于App的需要通过代码改变这些状态。例如，可以禁用一个control，防止用户与其交互。用户交互也可以改变control的状态。  

### The Target-Action Mechanism
Control使用Target-Action机制报告代码中发生的事件。TA机制简化了App中使用controls的代码。用于替代编写追踪touch event的代码，只需要编写响应指定事件的action方法。例如，编写一个响应slider值变化的方法。Control处理所有追踪touch event的工作，用于决定何时调用你的方法。  
当向control添加action method是，