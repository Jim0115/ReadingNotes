# APIs of UIView
### Initializing
`init(frame frame: CGRect)`  
使用一个`CGRect`构造一个view，view的`center`和`bound`可以通过`frame`计算出来。  
子类重写此方法时必须先调用父类的init方法。

### Configuring a View’s Visual Appearance
`@NSCopying var backgroundColor: UIColor?`  
view的背景颜色，animatable。默认值是nil，表示透明的view。  

`var hidden: Bool`  
隐藏view，该属性为false时view仍然可能是隐藏的，原因是该视图的直接或间接父视图是隐藏的。  

`var alpha: CGFloat`  
0.0 - 1.0的浮点数，表示视图的透明度，默认为1.0。0.0表示完全透明，1.0表示完全不透明。此属性只作用于当前视图而对当前视图的子视图无影响。

`var opaque: Bool`  
此属性提示绘图系统如何绘制该view。若为true，表示视图完全不透明，绘图系统会进行优化一些绘制选项以提升性能。若为false，绘图系统会将此view与其他内容进行拼接。默认值为true。  
一个不透明的view要求其所有子view的alpha值为1.0且填满bounds，如果view是不透明的但其子view没有填满bounds或部分/全部子view是透明的，将产生不可预测的结果。如果view部分或全部透明，必须设置此选项为false  
只有自定义的`UIView`的子类需要设置此属性。此属性对于系统提供的视图如`UILabel`,`UIButton`,`UITableViewCell`等无意义。

`var tintColor: UIColor!`  
色调。默认为nil。该属性为nil的视图会使用其距离最近父视图的`tintColor`，若在其视图结构中没有设置`tintColor`，会使用系统默认的浅蓝色。

`var tintAdjustmentMode: UIViewTintAdjustmentMode`

    enum UIViewTintAdjustmentMode : Int {
        case Automatic // 与父视图相同
        case Normal // 正常
        case Dimmed // 灰暗 类似于enable = false状态
    }
`tintColor`的调整模式

`var clipsToBounds: Bool`  
true表示超出该视图可见bounds的部分将被裁剪掉。默认值为false

`var clearsContextBeforeDrawing: Bool`  
在绘制前是否自动清除bounds的内容。如果为true，则会在下一次绘制前清空上一次绘制的内容。默认为true  
如果设置此属性为false，则必须保证view的内容在`drawRect:`方法中被正确绘制。如果关于绘制的代码经过严格的优化，设置此属性为false可以提高绘制的性能，尤其是在只有视图的部分内容需要重绘的情况下。

`var maskView: UIView?`  
相关资料暂缺

`class func layerClass() -> AnyClass`  
返回当前类的实例的layer的类型。  
默认返回`CALayer`。子类可以重写此方法返回不同的layer类型。  
此方法只在创建视图的早期被调用一次。

`var layer: CALayer { get }`  
此view用于渲染的Core Animation Layer。
此view是其layer的delegate

### Configuring the Event-Related Behavior
`var userInteractionEnabled: Bool`  
view是否响应事件。  
若为false，用户事件（触碰、键盘点击等）将被忽略并从事件队列中被移除。默认为true。  
在动画过程中，涉及到的view会临时禁用userInteraction，除非使用`UIViewAnimationOptionAllowUserInteraction`选项。

`var multipleTouchEnabled: Bool`  
视图是否处理多点触控事件。默认为false  
当此属性为false时，同一window的其他view仍然可以接收到触摸事件。如果想让此view独占多点触控事件的处理，需要将此属性和`exclusiveTouch`属性同时设为true

`var exclusiveTouch: Bool`  
view是否独占事件的处理  
将此属性设为true将组织view将事件传递给同一window下的其他view，默认为false

### Configuring the Bounds and Frame Rectangles
`var frame: CGRect`  
在父视图的坐标系中当前视图的位置和大小  
若需要在修改此属性后调用`drawRect:`方法重绘视图，需要将view的`contentMode`属性设置为`UIViewContentModeRedraw`  
修改此属性可以动画化。如果修改了view的`transform`属性，则不应该再修改其`frame`，而是通过`center`和`bounds`属性修改其位置和大小。

`var bounds: CGRect`
view在其自身坐标系中的位置和大小  
默认情况下`bounds`的`origin`被设置为(0, 0)，可以修改此`origin`以显示view不同的部分。  `bounds`的`size`与`frame`的`size`相连，意味着修改一个会导致两个产生相同的变化，同时也会导致`center`属性对应变化   
修改此属性可以动画化。  
默认值为(0, 0, frame.width, frame.height)

`var center: CGPoint`  
在父视图坐标系中frame的中心点所在的位置，修改此属性将会导致frame的`origin`改变  

`var transform: CGAffineTransform`  
指定view进行的transform  
transform的原点是view的`center`，如果layer的`anchorPoint`改变则为该点。默认值为`CGAffineTransformIdentity`，即3x3单位矩阵。  
修改此属性可以动画化。  
`transform`属性不对Autolayout产生影响，Autolayout计算基于未经过`transform`的`frame`。  
如果view经过transform，view的`frame`会产生未定义的结果，所以应当被忽略。

### Managing the View Hierarchy
`var superview: UIView? { get }`  
获取当前view的父视图  

`var subviews: [UIView] { get }`
当前view的直接子视图  
通过此属性可以获取自定义view的结构，数组中view的顺序反映了subview的可见顺序，排在数组前面的view的可见性靠后，即`addSubview`总是将新的视图加到数组的末尾。  

`var window: UIWindow? { get }`  
view所在的window，为nil表示view还未被添加到window或其子视图中。

`func addSubview(_ view: UIView)`  
将`view`加到`self.subview`数组的末尾，即最新的view总是在最上方  
如果`view`已经在其他视图中，将会从原结构中移除，再添加到当前视图中  

`func bringSubviewToFront(_ view: UIView)`  
将一个view移到`subviews`数组的最后使其显示在最上层。对于不在其结构中的view调用此方法无效。

`func sendSubviewToBack(_ view: UIView)`  
将一个view移到`subviews`数组的头部使其显示在最下层。对于不在其结构中的view调用此方法无效。  

`func removeFromSuperview()`  
将receiver从其父视图中移除，同时从responder chain 中移除。
调用此方法将会移除父视图中所有与此视图相关的约束。

`func insertSubview(_ view: UIView, atIndex index: Int)`  
在指定index位置插入view。

`func insertSubview(_ view: UIView, aboveSubview siblingSubview: UIView)`  
`func insertSubview(_ view: UIView, belowSubview siblingSubview: UIView)`  
将view插入到某个子视图的上方/下方。  

`func exchangeSubviewAtIndex(_ index1: Int, withSubviewAtIndex index2: Int)`  
交换`subviews`数组中的两个view，以修改其上下关系  

`func isDescendantOfView(_ view: UIView) -> Bool`  
判断view是否为当前view的后代。

### Configuring the Resizing Behavior  
`var autoresizingMask: UIViewAutoresizing`  
位掩码，决定当父视图bounds改变时视图如何改变其自身大小

    struct UIViewAutoresizing : OptionSetType {
        init(rawValue rawValue: UInt)
        static var None: UIViewAutoresizing { get }
        static var FlexibleLeftMargin: UIViewAutoresizing { get }
        static var FlexibleWidth: UIViewAutoresizing { get }
        static var FlexibleRightMargin: UIViewAutoresizing { get }
        static var FlexibleTopMargin: UIViewAutoresizing { get }
        static var FlexibleHeight: UIViewAutoresizing { get }
        static var FlexibleBottomMargin: UIViewAutoresizing { get }
    }

UIViewAutoresizingFlexibleLeftMargin 自动调整与superView左边的距离，保证与superView右边的距离不变。  
UIViewAutoresizingFlexibleRightMargin 自动调整与superView的右边距离，保证与superView左边的距离不变。  
UIViewAutoresizingFlexibleTopMargin 自动调整与superView顶部的距离，保证与superView底部的距离不变。  
UIViewAutoresizingFlexibleBottomMargin 自动调整与superView底部的距离，也就是说，与superView顶部的距离不变。  
UIViewAutoresizingFlexibleWidth 自动调整自己的宽度，保证与superView左边和右边的距离不变。  
UIViewAutoresizingFlexibleHeight 自动调整自己的高度，保证与superView顶部和底部的距离不变。  

`var autoresizesSubviews: Bool`  
当其bounds改变时，是否自动改变子视图大小。默认为true

`var contentMode: UIViewContentMode`
当视图bounds改变时如何布局其内容  

    enum UIViewContentMode : Int {
        case ScaleToFill
        case ScaleAspectFit
        case ScaleAspectFill
        case Redraw
        case Center
        case Top
        case Bottom
        case Left
        case Right
        case TopLeft
        case TopRight
        case BottomLeft
        case BottomRight
    }
使用此属性可以将视图内容固定在某个位置或随视图大小变化。默认为`ScaleToFill`

`func sizeThatFits(_ size: CGSize) -> CGSize`  
对于view，返回其`bounds.size`；对于label等有内容的view，返回其contentSize  

`func sizeToFit()`  
调整和移动view使其恰好围绕其子视图。大小为`sizeThatFits`方法的返回值。  
此方法不应被override。若想调整大小，应该重写`sizeThatFits`方法。

### Laying out Subviews
`func layoutSubviews()`
布局子视图  
此方法不应该被直接调用。如果想强制布局更新，调用`setNeedsLayout`，优先在下次drawing时更新。如果想强制立刻更新布局，调用`layoutIfNeeded`方法。

`func setNeedsLayout()`  
使当前布局失效，在下次更新循环中触发布局更新。  
在主线程调用此方法。

`func layoutIfNeeded()`  
立即对子视图进行布局。  
此方法将会对以当前视图为根的所有视图树生效。

`class func requiresConstraintBasedLayout() -> Bool`  
表明当前视图是否依赖于基于约束的布局系统。  

`var translatesAutoresizingMaskIntoConstraints: Bool`  
view的autoresizing掩码是否被翻译为Auto Layout的约束。  
若此属性为true，系统会创建一个约束集合复制view的autoresizing mask的行为。这也允许通过修改view的`frame`, `bounds`和`center`确定view的位置和大小，在同时使用frame和Auto Layout。  
注意autoresizing mask约束完全确定了view的大小和位置。因此，不能添加额外的约束去修改view的大小和位置，否则会产生冲突。如果想使用Auto Layout动态计算view的大小和位置，必须设置此属性为`false`。然后向view提供一套无冲突和歧义的约束。  
通过代码创建的view此属性默认为`true`。IB中添加的view此属性默认为`false`。

### Creating Constraints Using Layout Anchors
iOS 9 新增的一种创建约束的方式。使用`NSLayoutAnchor`类下的`func constraintEqualToAnchor(_ anchor: NSLayoutAnchor!) -> NSLayoutConstraint!`方法创建约束。参数有：

* bottomAnchor
* centerXAnchor
* centerYAnchor
* firstBaselineAnchor
* heightAnchor
* lastBaselineAnchor
* leadingAnchor
* leftAnchor
* rightAnchor
* topAnchor
* trailingAnchor
* widthAnchor 
        
等价的两种方式：
    
    NSLayoutConstraint(item: subview,
        attribute: .Leading,
        relatedBy: .Equal,
        toItem: view,
        attribute: .LeadingMargin,
        multiplier: 1.0,
        constant: 0.0).active = true
        
    let margins = view.layoutMarginsGuide    
    subview.leadingAnchor.constraintEqualToAnchor(margins.leadingAnchor).active = true
    
### Managing the View’s Constraints
`var constraints: [NSLayoutConstraint] { get }`  
被view持有的所有的约束

`func addConstraint(_ constraint: NSLayoutConstraint)`  
向view添加约束用于布局其自身或其subviews  
传入的约束所包含的view，必须是此view或其subviews。添加的约束将被view持有。  
在iOS 8之后，直接设置约束的`active`属性为true，而不是将其添加到view上。约束会自动添加到正确的view上。  

`func addConstraints(_ constraints: [NSLayoutConstraint])`  
向view添加一组约束。  
在iOS 8 之后，使用 `NSLayoutConstraint` 的类方法 `activateConstraints:` 代替此方法。约束会自动呗添加到正确的view上。  

`func removeConstraint(_ constraint: NSLayoutConstraint)`  
`func removeConstraints(_ constraints: [NSLayoutConstraint])`  
同上，直接设置约束的`active`属性为false。使用 `NSLayoutConstraint` 的类方法 `deactivateConstraints:` 代替。约束会被自动移除。  

### Working with Layout Guides
`func addLayoutGuide(_ layoutGuide: UILayoutGuide)`  
向view添加一个指定的layout guide。  
将一个layout guide添加到view的`layoutGuides`数组中。同时也会设置guide的`owningView`属性为当前view。  

`var layoutGuides: [UILayoutGuide] { get }`  
返回view持有的所有layout guide对象。

`var layoutMarginsGuide: UILayoutGuide { get }`  
一个代表view的margins的layout guide。  
使用此属性的anchor创建基于margin的约束。  

`var readableContentGuide: UILayoutGuide { get }`  
一个代表view的readable部分的layout guide。  
1. readable content guide不会超过view的layout margin guide  
2. readable content guide在view的layout margin guide中垂直居中  
3. readable content guide的宽度小于等于给予当前动态字体大小的readable width。

`func removeLayoutGuide(_ layoutGuide: UILayoutGuide)`  
从一个view中移除某个layout guide。将这个layout guide的`owningView`属性设为nil。同时将移除包含这个layout guide 的所有约束。  
除非被加入视图结构的某个视图中，否则layout guide不能在约束中被使用。  

### Measuring in Auto Layout
`func systemLayoutSizeFittingSize(_ targetSize: CGSize) -> CGSize`  
返回满足view所持有的所有约束的size。

`func intrinsicContentSize() -> CGSize`  
返回view的自然大小，只考虑其自身的属性。  

`func invalidateIntrinsicContentSize()`  
使view自身的content size失效。  
当view的内容变化时，调用此方法使其自身的content size失效。将会允许基于约束的布局系统为其生成一个新的content size。

`func contentCompressionResistancePriorityForAxis(_ axis: UILayoutConstraintAxis) -> UILayoutPriority`  
返回view在指定方向上的抗压缩优先级。优先级越高，越优先满足其content size。

`func setContentCompressionResistancePriority(_ priority: UILayoutPriority, forAxis axis: UILayoutConstraintAxis)`  
设置view的抗压缩优先级。

`func contentHuggingPriorityForAxis(_ axis: UILayoutConstraintAxis) -> UILayoutPriority`  
返回view在指定方向上的抗拉伸优先级。

`func setContentHuggingPriority(_ priority: UILayoutPriority, forAxis axis: UILayoutConstraintAxis)`  
设置view的抗拉伸优先级。

### Aligning Views in Auto Layout
`func alignmentRectForFrame(_ frame: CGRect) -> CGRect`  
返回view用于对齐的矩形。  
基于约束的布局系统使用alignment rectangles来对齐views，而不是使用frame。自定义视图允许基于其内容的位置对齐，同时仍保留frame包含其内容的装饰，例如阴影等。  
默认实现返回被view的`alignmentRectInsets`修改过的frame。自定义view可以重写`alignmentRectInsets`方法指定content在frame中的位置。自定义view需要主观的变化，可以重写`alignmentRectForFrame:` 和 `frameForAlignmentRect:`方法来指定其content的位置。这两个方法必须总是互逆的。

`func frameForAlignmentRect(_ alignmentRect: CGRect) -> CGRect`  
与上一个方法相反，返回view的frame。

`func alignmentRectInsets() -> UIEdgeInsets`  
返回view的frame与其content间的长度。  
默认返回`UIEdgeInsetsZero`。通过重写此方法返回Insets可以使布局系统使用基于view的content对齐，而不是使用其frame。

`var viewForFirstBaselineLayout: UIView { get }`  
`var viewForLastBaselineLayout: UIView { get }`  
返回一个view用于满足first baseline 或 last baseline 的约束。  
对于自定义视图，如果想对其first baseline添加约束，则需要重写此方法返回其对应的view。  
first baseline view 默认返回 last baseline view。所以若二者为同一view，只需要重写last方法即可。 

### Triggering Auto Layout
`func needsUpdateConstraints() -> Bool`  
view的约束是否需要更新。

`func setNeedsUpdateConstraints()`  
控制view的约束是否需要更新。  
当自定义视图中的某个属性的改变会影响约束时，可以调用此方法指示约束需要在将来的某个时间点更新。系统将会调用`updateConstraints`方法。使用此方法作为最佳工具批量处理约束的变化。

`func updateConstraints()`  
更新view的约束。  
重写此方法调整约束的变化。   
只有两种情况需要重写此方法：1. 在适当的地方改变约束太慢；2. view将产生大量重复的变化。  
调用`setNeedsUpdateConstraints`方法计划change。系统将会在布局发生前调用你对`updateConstraints`方法的实现。确认所有对内容必须的约束处于合适的地方。  
对此方法的实现必须尽可能效率高。不要在其中禁用所有约束然后重新启用需要的约束。相反，app必须有一些方法追踪约束。只有需要变化的item需要被改变。  
不要在实现中调用`setNeedsUpdateConstraints`方法。  
在方法的最后调用`super.updateConstraints`。  

`func updateConstraintsIfNeeded()`  
要求view更新其和其subviews的约束。  
当view出发了一个新的layout pass，系统调用此方法确保当前view和其subviews的约束都更新到最新。此方法会被系统自动调用。也可以手动调用用以检查约束是否为最新。  
子类不应重写此方法。  

### Debugging Auto Layout
`func constraintsAffectingLayoutForAxis(_ axis: UILayoutConstraintAxis) -> [NSLayoutConstraint]`  
返回在给定方向上作用于布局的约束。  
此方法只能被用于debug。

`func hasAmbiguousLayout() -> Bool`  
对view的布局产生影响约束是否不够确定view的位置。  
此方法检查是否还有view的其他frame也能满足view的约束。检查开销很大但在debug时很有用。  
此方法只能被用于debug。

`func exerciseAmbiguityInLayout()`  
在一个有歧义布局的所有有效值中随机变换view的frame。  
此方法只能被用于debug。  

### Managing the User Interface Direction
`var semanticContentAttribute: UISemanticContentAttribute`  
对view内容的语义描述，用于决定view在从左到右或从右到左的布局中是否需要翻转。  

    enum UISemanticContentAttribute : Int {
      case Unspecified // default. will filp
      case Playback // for playback controls. will not flip
      case Spatial // will not flip
      case ForceLeftToRight 
      case ForceRightToLeft
    }

`class func userInterfaceLayoutDirectionForSemanticContentAttribute(_ attribute: UISemanticContentAttribute) -> UIUserInterfaceLayoutDirection`  
返回此view在特定语义内容下的方向。

    enum UIUserInterfaceLayoutDirection : Int {
      case LeftToRight
      case RightToLeft
    }

### Configuring Content Margins
`var layoutMargins: UIEdgeInsets`  
指定view的边界与其subview的空间大小。  
默认值为每个方向8 points。  
View controller的root view，会由系统管理margin。上下0 points。左右为16或20 points，取决于当前的size class。此时margin不可被更改。

`var preservesSuperviewLayoutMargins: Bool`  
当前view是否同样考虑其superview的margins。  
eg. 某个view的frame等于其superview的bounds。而其content恰好填满其bounds。若此view启用此选项，则会压缩其content，保证不会遮挡其superview的margin。  
默认为false。

`func layoutMarginsDidChange()`  
提醒view其layout margins变化。  
默认为空实现。子类可以重写此方法，当view的`layoutMargins`属性变化时，此方法将被系统调用。

### Drawing and Updating the View
`func drawRect(_ rect: CGRect)`  
根据传入的rect绘制view。  
参数rect：view需要更新的部分。view第一次被绘制时，rect通常是view整个可见的bounds。在之后的绘制操作中，rect可能只是view的一部分。  
此方法的默认实现为空。使用Core Graphics和UIKit技术的子类重写此方法，在其中实现绘制代码。  
此方法被调用时，UIKit已经为view配置好绘制环境，此时可以调用任何用于渲染内容的绘制方法。此外，UIKit为绘制创建和配置graphics context，同时转换context的origin使其和view的bounds的origin相同。使用`UIGraphicsGetCurrentContext()`方法可以获得此时的graphics context，但不要强引用它，因为每次调用`drawRect:`方法时它都会变化。  
所有的绘制都应该被限制在`rect`参数指定的矩形范围内。同时，如果view的`opaque`属性为true，在`drawRect:`方法的实现中必须使用不透明内容完全填满`rect`。  
如果直接继承自`UIView`，不需要调用super的方法。如果继承自其他view，是否调用super需要视情况而定。  
此方法当view第一次呈现或一些事件发生使view的一部分不可见时调用。不应该直接调用此方法。如果需要重新绘制，调用`setNeedDisplay`或`setNeedsDisplayInRect:`。

`func setNeedsDisplay()`  
将view的整个bounds标记为需要重绘。  
用此方法提醒系统view的内容需要被重绘。view将会在下次drawing cycle被重绘。  
只应该当view的内容变化时调用此方法重绘view。如果只是简单改变了view的大小和位置，view通常不会被重绘。已存在的内容将会根据view的`contentMode`属性进行调整。

`func setNeedsDisplayInRect(_ rect: CGRect)`  
标记view的一部分需要重绘。  

`var contentScaleFactor: CGFloat`  
被view使用的缩放比例。  

`func tintColorDidChange()`  
系统在view的`tintColor`属性变化时调用此方法。包括view继承自父视图的tintColor。  
在你的实现中，根据需要改变view的渲染方式。

### Formatting Printed View Content
`func viewPrintFormatter() -> UIViewPrintFormatter`  
`func drawRect(_ rect: CGRect, forViewPrintFormatter formatter: UIViewPrintFormatter)`

### Managing Gesture Recognizers
`func addGestureRecognizer(_ gestureRecognizer: UIGestureRecognizer)`  
向view添加一个gesture recognizer。  
view和其所有subviews都会收到触摸事件。view对gesture recognizer强引用。

`func removeGestureRecognizer(_ gestureRecognizer: UIGestureRecognizer)`  
将一个gesture recognizer从view上移除。

`var gestureRecognizers: [UIGestureRecognizer]?`  
view上所有gesture recognizer。如果没有，返回一个空数组。

`func gestureRecognizerShouldBegin(_ gestureRecognizer: UIGestureRecognizer) -> Bool`  
询问view，gesture recognizer是否被允许持续追踪触控事件。  
返回true表示允许。false将导致gesture recognizer直接过渡到` UIGestureRecognizerStateFailed`状态。  
此方法被调用时，gesture recognizer处于`UIGestureRecognizerStatePossible`状态，询问其是否能过渡到`UIGestureRecognizerStateBegan`状态。  
默认实现返回true。

### Animating Views with Block Objects
`class func animateWithDuration(_ duration: NSTimeInterval, delay delay: NSTimeInterval, options options: UIViewAnimationOptions, animations animations: () -> Void, completion completion: ((Bool) -> Void)?)`  
    
* `duration`: 动画持续时间
* `delay`: 动画开始前的延迟时间
* `option`: 动画的可选项
* `animations`: 一个包含需要执行动画的block。在其中更改view中可动画化的属性。
* `completion`: 当动画结束后执行的block。参数代表此block被调用时动画是否执行完毕。

`class func transitionWithView(_ view: UIView, duration duration: NSTimeInterval, options options: UIViewAnimationOptions, animations animations: (() -> Void)?, completion completion: ((Bool) -> Void)?)`  
创建一个指定view的过渡动画。  
* `view`：执行过渡动画的view
* `duration`：动画持续的时间
* `option`：动画的可选项
* `animations`：包含指定变化的block
* `completion`：动画结束后被调用的block
 
`class func transitionFromView(_ fromView: UIView, toView toView: UIView, duration duration: NSTimeInterval, options options: UIViewAnimationOptions, completion completion: ((Bool) -> Void)?)`
创建从一个view到另一个view的过渡动画。  

* `fromView`：过渡动画的起始view。默认情况，此view将被从其superview中移除。
* `toView`：过渡动画的结束view。默认情况：此view将被添加到`fromView`的superview中。
* `duration`：动画持续的时间
* `options`：动画的可选项
* `completion`：动画结束后被调用的block

此方法提供了一种将一个view替换成另一个view的简单方法。默认，`toView`将替换掉`fromView`的视图层级。如果两个view都已经是视图层次的一部分，可以启用`UIViewAnimationOptionShowHideTransitionViews`选项，将替换操作改为隐藏。  
此动画将立即执行，除非当前有正在执行的动画。

`class func animateKeyframesWithDuration(_ duration: NSTimeInterval, delay delay: NSTimeInterval, options options: UIViewKeyframeAnimationOptions, animations animations: () -> Void, completion completion: ((Bool) -> Void)?)`  
创建视图基于关键帧的动画。
在`animation`block中，使用`UIView`的类方法`addKeyframeWithRelativeStartTime:relativeDuration:animations:`一次或多次添加关键帧。若不在其中添加关键帧，则此方法表现为和普通的动画方法相同。

`class func addKeyframeWithRelativeStartTime(_ frameStartTime: Double, relativeDuration frameDuration: Double, animations animations: () -> Void)`  
指定某个关键帧的时间、长度和操作。  

* `frameStartTime`：帧开始时间。范围是0-1。0表示动画开始时，1表示动画结束时。
* `frameDuration`：帧长度。范围和意义与上一参数相同。
* `animations`：动画block

通过此方法在上一方法中添加关键帧，即可实现在指定时间产生指定长度的动画。

`class func performSystemAnimation(_ animation: UISystemAnimation, onViews views: [UIView], options options: UIViewAnimationOptions, animations parallelAnimations: (() -> Void)?, completion completion: ((Bool) -> Void)?)`  
执行指定系统动画，在执行系统动画的同时执行自定义动画。

* `animation`：需要执行的系统动画。只有`Delete`一项，当动画完成时将view移除视图结构。
* `views`：需要在其上执行动画的view
* `options`：动画的可选项
* `parallelAnimations`：在系统动画执行同时执行的动画，和系统动画有相同的启动时间和持续时间。在动画中，不要修改`views`参数数组中视图的属性。
* `completion`：动画完成时调用的block。

`class func animateWithDuration(_ duration: NSTimeInterval, delay delay: NSTimeInterval, usingSpringWithDamping dampingRatio: CGFloat, initialSpringVelocity velocity: CGFloat, options options: UIViewAnimationOptions, animations animations: () -> Void, completion completion: ((Bool) -> Void)?)`  
执行一个动画，动画的时间曲线对应于物理弹跳。  

* `duration`：动画持续时间
* `delay`：动画开始前延迟时间
* `dampingRatio`：接近静止状态的阻尼比率。平滑减速而没有摆动，使用1。越接近0的值摆动越强烈。
* `velocity`：初始弹跳速度。1表示1秒完成一次动画的全程。如果整个动画需要移动200points，那么0.5表示初始速度为100pt/s。
* `options`：动画的可选项。
* `animations`：动画的block。不能为NULL
* `completion`：动画结束时调用的block。

`class func performWithoutAnimation(_ actionsWithoutAnimation: () -> Void)`  
禁用一个view的过渡动画。  

### Using Motion Effects
`func addMotionEffect(_ effect: UIMotionEffect)`  
应用一个motion effect于view。  
`UIMotionEffect`是一个抽象类，常用其子类`UIInterpolatingMotionEffect`。产生的效果类似于锁屏界面。图片在手机方向改变时会调整其位置。

`var motionEffects: [UIMotionEffect]`   

`func removeMotionEffect(_ effect: UIMotionEffect)`  

### Preserving and Restoring State
`var restorationIdentifier: String?`  
决定view是否支持状态恢复的标识符。  
此属性表明view的状态信息是否应该被保留，也用于在恢复过程中标示出view。默认为nil，表示view的状态不需要被储存。赋值一个string对象使得持有此view的VC知道view有相关状态需要保存。  
view必须实现`encodeRestorableStateWithCoder:`和`decodeRestorableStateWithCoder:`方法赋值此属性才有意义。使用这些方法写入view指定的状态信息，随后使用这些数据恢复view之前的状态。  
important  
只是设置此属性并不足以保证view的状态被保留和恢复。持有view的VC，以及VC所有的父VC
，都必须有restoration identifier。  

`func encodeRestorableStateWithCoder(_ coder: NSCoder)`  
编码view的状态相关信息。  
如果app支持状态保留，重写此方法。只应该保存view用于恢复其当前状态的数据。不要储存view自身，不要储存启动时可以通过其他方法推断出的数据。  
只有少数view需要储存其状态信息。大多数view只需要使用从VC获得的数据。此方法适用于拥有**用户自定义**状态且在App重新启动时会丢失。  
在实现中，此方法可以编码其他引用的可存储的view和VC对象。  
除了view和VC，其他对象执行正常的序列化，需要实现`NSCoding`协议。在解码过程中，将创建一个新对象并使用从压缩文件中得到的数据初始化。  
推荐在实现的某部分调用super，使父类有机会保存其状态信息。

`func decodeRestorableStateWithCoder(_ coder: NSCoder)`  
解码和恢复view的状态相关信息。  
此方法的实现中应使用存储的状态信息恢复view之前的状态。如果在上一方法中调用了super，此方法也应调用。

### Capturing a View Snapshot
`func snapshotViewAfterScreenUpdates(_ afterUpdates: Bool) -> UIView`  
返回基于当前view的内容的快照。  
参数`afterUpdates`：快照是否合并最近的修改。传入false表示捕获屏幕当前状态，不包括最近的变化。

### Identifying the View at Runtime 
`var tag: Int`  
用于标记view对象的integer。  
默认值为0。

`func viewWithTag(_ tag: Int) -> UIView?`  
返回tag为指定值的view。  
此方法搜索当前view及其所有subviews。会搜索整个视图树，直到找到对应的tag或全部遍历。

### Converting Between View Coordinate Systems
`func convertPoint(_ point: CGPoint, toView view: UIView?) -> CGPoint`  
将一个point的值从self的坐标系转换到`toView`的坐标系。  
如果参数`view`为nil，则将point转换到window的坐标系中。否则，参数`view`和`self`都必须属于同一个UIWindow object。

`func convertPoint(_ point: CGPoint, fromView view: UIView?) -> CGPoint`  
将一个point的值从`fromView`的坐标系转换到self的坐标系。

`func convertRect(_ rect: CGRect, toView view: UIView?) -> CGRect`  
将一个rect的值从self的坐标系转换到`toView`的坐标系。 

`func convertRect(_ rect: CGRect, fromView view: UIView?) -> CGRect`  
将一个rect的值从`fromView`的坐标系转换到self的坐标系。

### Hit Testing in a View
`func hitTest(_ point: CGPoint, withEvent event: UIEvent?) -> UIView?`  
返回view包含指定点的最远的后代。  

* `point`：view的bounds中的某个点
* `event`：授权调用此方法的UIEvent，如果在外部调用，可以指定为nil

此方法通过在视图结构中反复调用`pointInside:withEvent:`方法来决定哪个子视图应该受到触摸事件。如果`pointInside:withEvent:`返回true，那么子视图结构继续调用`pointInside:withEvent:`方法直到找到其最靠前的且包含指定点的view。如果view不包含指定点，则视图结构的分支都将被忽略。很少需要直接调用此方法，但可能重写此方法隐藏从subview接受到的touch event。  
此方法忽略隐藏的view，禁用userInteraction的view，alpha值小于0.01的view。此方法也不会考虑view的内容。因此，一个view仍有可能被返回即使其在指定点部分的content是透明的。  
处于view的bounds外部的点不会生效，即使这个点确实处于view的某个subview的内部。这可能发生在view的`clipsToBounds`属性为false时。

`func pointInside(_ point: CGPoint, withEvent event: UIEvent?) -> Bool`  
判断view是否包含指定点。  

### Ending a View Editing Session
`func endEditing(_ force: Bool) -> Bool`
使view或其包含的某个textfield退出first responder。返回true表示已经退出。  
此方法查看当前view和其subviews，寻找作为first responder的text field。如果找到，则要求此text field退出first responder。如果参数`force`为true，则不询问直接退出。  
在测试中不论参数为true或false。textfield的`canResignFirstResponder`方法都会被调用。而如果`
canResignFirstResponder`返回false，不会退出编辑状态。此时此方法的返回值与参数相同，而textfield的`isFirstResponder`方法都返回true。

### Observing View-Related Changes
`func didAddSubview(_ subview: UIView)`  
`func willRemoveSubview(_ subview: UIView)`  
`func willMoveToSuperview(_ newSuperview: UIView?)`  
`func didMoveToSuperview()`   
`func willMoveToWindow(_ newWindow: UIWindow?)`  
`func willMoveToWindow(_ newWindow: UIWindow?)`  
view的subviews、superview、window改变前后view将收到对应方法的通知。

### Observing Focus


---
### CGAffineTransform 仿射变换
![image](https://docs-assets.developer.apple.com/published/8a0bbde8e5/equation01_2x_fabc9070-1967-4d6f-a086-17ab5fcfef6d.png)  

    Swift: 
    CGAffineTransform(a: CGFloat, b: CGFloat, c: CGFloat, d: CGFloat, tx: CGFloat, ty: CGFloat)
    
    Objective-C:
    CGAffineTransformMake(CGFloat a, CGFloat b, CGFloat c, CGFloat d, CGFloat tx, CGFloat ty);
    
由于最后一列总为(0, 0, 1)，所以只需要六个参数即可确定变换矩阵。  
变换的公式为：  
![image](https://docs-assets.developer.apple.com/published/8a0bbde8e5/equation02_2x_71f7e62f-7cbe-4670-9b34-924b49e48f72.png)  
即：  
![image](https://docs-assets.developer.apple.com/published/8a0bbde8e5/equation03_2x_b4b74916-ba29-4c3c-8fa2-ada82ad5c659.png)

#### 常用方法
`func CGAffineTransformMakeRotation(_ angle: CGFloat) -> CGAffineTransform`  
以`center`为轴顺时针旋转angle弧度 (M_PI, M_PI_2...)  
![image](http://wangshijiedemacbook-pro.local:55667/Dash/enxijbwn/documentation/GraphicsImaging/Reference/CGAffineTransform/Art/equation10_2x.png)

`func CGAffineTransformMakeScale(_ sx: CGFloat, _ sy: CGFloat) -> CGAffineTransform`  
以`center`为中心缩放`sx`、`sy`倍。  
![image](http://wangshijiedemacbook-pro.local:55667/Dash/enxijbwn/documentation/GraphicsImaging/Reference/CGAffineTransform/Art/equation08_2x.png)

`func CGAffineTransformMakeTranslation(_ tx: CGFloat, _ ty: CGFloat) -> CGAffineTransform`  
将view向x、y方向移动tx、ty  
![image](http://wangshijiedemacbook-pro.local:55667/Dash/enxijbwn/documentation/GraphicsImaging/Reference/CGAffineTransform/Art/equation06_2x.png)  

#### 对已有transform进行操作
`func CGAffineTransformTranslate(_ t: CGAffineTransform, _ tx: CGFloat, _ ty: CGFloat) -> CGAffineTransform`平移  
`func CGAffineTransformScale(_ t: CGAffineTransform, _ sx: CGFloat, _ sy: CGFloat) -> CGAffineTransform`缩放  
`func CGAffineTransformRotate(_ t: CGAffineTransform, _ angle: CGFloat) -> CGAffineTransform`旋转  
`func CGAffineTransformInvert(_ t: CGAffineTransform) -> CGAffineTransform`转置  
`func CGAffineTransformConcat(_ t1: CGAffineTransform, _ t2: CGAffineTransform) -> CGAffineTransform`乘积