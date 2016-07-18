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
true表示超出该视图可见bounds的部分将被裁剪第掉。默认值为false

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