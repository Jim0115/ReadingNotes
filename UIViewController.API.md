# UIViewController
#### superclass
`UIResponder`


`UIViewController`类提供了管理iOS App中view的基础。一个view controller（VC）管理组成APP的UI的一部分view集合。VC负责读取和部署这些view，管理和view之间的交互，协调view响应任何相关的数据对象。VC也和其他controller协调，包括其他VC，这有助于管理App的整体界面。  
很少直接创建`UIViewController`的实例。相反，创建`UIViewController`的子类，使用这些对象提供需要的特定行为和视觉效果。  
一个VC的主要任务如下：

* 更新view的内容，通常是响应其下的data的变化。
* 响应用户交互。
* 改变view的大小，管理所有界面的布局。

一个VC紧密联系与其管理的view，同时参与用于处理event的responder chain。VC也是`UIResponder`对象，它在responder chain中的位置介于VC的root view和root view的superview之间。如果VC的view没有处理事件，VC可以选择处理此事件或将其向上传递。  
VC很少独立使用。相反，通常同时使用多个VC，其中的每一个拥有UI的一部分。例如，一个VC显示一个tableview，另一个VC显示table选中的项目。通常，在同一时间只有一个VC的view是可见的。一个VC可能present一个不同的VC用于显示新的view集合。

### Subclassing Notes
每个App包括至少一个`UIViewController`的自定义子类。更普遍的情况是，app爆款很多自定义VC。自定义VC定义app所有的行为，包括其样式和如何响应用户交互。
#### View Management
每个VC管理一个view hierarchy，root view被储存在VC的`view`属性中。root view主要作为view hierarchy中其他view的container。root view的大小和位置取决于拥有其的对象，可能是VC或app的window。被window持有的VC是app的root VC，它的view用于填满整个window。  
VC惰性加载其views。第一次访问`view`属性时加载或创建VC的view。有几种方法指定VC的view。

* 在app的storyboard文件中指定VC和其view。推荐使用Storyboard指定view。当使用Storyboard时，指定view和其与VC的联系。同时指定VC之间的relationship和segue。  
从storyboard加载一个VC，需要在合适的storyboard对象上调用  `instantiateViewControllerWithIdentifier:`方法。Storyboard会创建VC并将其返回到代码中。
* 使用nib文件指定VC的view。一个nib文件能够让你指定单个VC的view，但不能让你指定VC间的relationship和segue。nil文件只储存关于VC自身的最小信息。  
若要使用nib文件初始化一个VC，使用代码创建VC类，使用`initWithNibName:bundle:`对其初始化。当其view被请求时，VC从nib文件中加载。
* 使用`loadView`方法指定VC的view。在那个方法中，创建你的view hierarchy，将其设置为VC的`view`。

以上三种方法有相同的最终结果：创建合适的view集合，通过`view`属性暴露。  
每一个VC都应该是其view的唯一持有者。不能在VC之间共享view。  
一个VC的root view总是被缩放至指定大小。对于view hierarchy中的其它view，使用IB指定约束用于管理每个view在其superview的bounds中的大小和位置。  

### Handling View-Related Notifications
当VC的view的可见性改变时，VC自动调用其自身的方法，因此子类可以响应变化。使用类似`viewWillAppear:`的方法布置view用于显示在屏幕上，使用`viewWillDisappear:`方法保存变化和其它状态信息。

### Handling View Rotations
在iOS 8中，所有方向相关的方法都已经不推荐使用。相反，旋转被看作VC的view的size发生变化。因此将被`viewWillTransitionToSize:withTransitionCoordinator:`方法报告。当界面方向变化时，UIKit在window的root VC上调用此方法。VC随后提醒其child VC，沿着VC hierarchy传递此消息。  
在iOS 6和iOS 7中，app支持的界面方向定义在app的`Info.plist`文件中。VC可以通过重写`supportedInterfaceOrientations`方法限制支持的的方向。通常，系统仅在window的rootVC或VC填满整个窗口时调用此方法。child VC使用由其parent VC提供的window的一部分，不直接参与决定支持的方向。App的方向掩码和VC的方向掩码的交集用于决定VC支持那些方向。  
当一个VC想要在某个方向全屏时，可以重写`preferredInterfaceOrientationForPresentation`方法。  
当一个旋转发生在可见的VC时，`willRotateToInterfaceOrientation:duration:`, `willAnimateRotationToInterfaceOrientation:duration:`, and `didRotateFromInterfaceOrientation:`将在旋转过程中被调用。`viewWillLayoutSubviews`方法也会在view被其superview改变大小和位置后被调用。当VC不是可见的时，这些方法在旋转时都不会被调用。反而，`viewWillLayoutSubviews`方法将在view可见时被调用。  
在启动时，app总是以竖屏方向设置界面。在`application:didFinishLaunchingWithOptions:`返回后，app使用VC的旋转机制旋转view到合适的方向。  

### Implementing a Container View Controller
一个自定义的`UIViewController`子类也可以作为一个container VC。一个container VC管理其持有的其它VC的内容呈现，也被认为是它的child VC。一个child的view可以作为container的view或和container的view一起展现。  
一个container子类必须声明关联其children的public接口。这些方法的实现由你决定，取决于你创建的container的语义。你需要决定多少children可以被你的VC同时显示，何时显示这些children，在你VC的view hierarchy中的何处显示。你的VC类定义被children共享的关系。通过公开一个container的清晰的公共接口，确保children合理使用其逻辑，没有访问过多关于container实现行为的细节。  
Container必须先关联child VC，之后才能将child的root view加入到自己的view hierarchy中。这允许iOS确定从child VC的事件路径。同样，在从view hierarchy中移除了child的root view后，也要断开child VC的连接。用于建立或断开这些连接，你的container调用基类定义的指定方法。这些方法不是被你的container类的client调用的，仅用于你的container实现提供指定容器的行为。  
以下是推荐调用的方法：

* `addChildViewController:`
* `removeFromParentViewController`
* `willMoveToParentViewController:`
* `didMoveToParentViewController:`

当创建一个container VC时，不必重写任何方法。

### Memory Management
在iOS中，内存是关键资源。VC提供内置的关键时刻减少其内存使用的支持。`UIViewController`类提供了一些低内存状态的自动处理通过其`didReceiveMemoryWarning`方法，用于释放不必要的内存。  

### State Preservation and Restoration
如果你向VC的`restorationIdentifier`属性赋值，系统可能会要求VC encode自身，在app转换到background状态时。当保存时，VC会保存其view hierarchy中任何拥有`restoration identifiers`的view。VC不会自动保存任何状态。如果你实现了一个自定义contain VC，你必须手动encode任何child VC。对于encode的每个child都必须有独立的restoration identifiers。

## API
`init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: NSBundle?)`  
Designated Initializer  
从指定bundle的nib文件中返回新的初始化后的VC。  
这是此类的指定构造器。当使用SB定义VC和其关联的view时，不要直接初始化VC的类。相反，当一个segue被触发时，你的VC会自动创建。或者，使用SB对象的`instantiateViewControllerWithIdentifier:`方法。当从一个SB实例化一个VC时，iOS构造一个新的VC通过其`initWithCoder:`方法，设置其`nibName`属性为储存在SB中的nib文件。  
指定的nib文件不会立即被读取。它将在VC的view第一次被访问时读取。如果你想在nib文件读取之后执行额外的初始化操作，重写`viewDidLoad`方法，在此执行操作。  
如果参数`nibName`为nil同时没有重写`loadView`方法，VC会搜索其`nibName`属性描述的nib文件。  


`var nibName: String? { get }`  
VC的nib文件的名字，如果已被指定。  
此属性包括了在初始化时使用`initWithNibName:bundle:`方法指定的值。可能为nil。  
如果你使用一个nib file存储VC的view，推荐在初始化时制度nib file。然而，如果没有指定nib name，也没有重写`loadView`方法，VC会通过其他方法搜索nib文件。特别的，VC会查找拥有何时名称的nib文件，当view被请求时读取nib文件。会按照以下顺序查找nib文件：  

1. 如果VC的类名以"Controller"结尾，例如"MyViewController"，VC会查找nib文件的名称匹配其类名去掉"Controller"的部分，例如`MyView.nib`。
2. 会查找匹配VC类名的nib file。例如`MyViewController`类会查找`MyViewController.nib`文件。

包含平台特定信息的nib name只会在对应平台被读取。比如`MyViewController~ipad.nib`只会在iPad上被读取。

`var nibBundle: NSBundle? { get }`  
如果存在，返回VC的nib bundle。

### Interacting with Storyboards and Segues
`var storyboard: UIStoryboard? { get }`  
返回VC的来源SB。  
如果VC不是使用SB实例化的，此选项为nil。

`func shouldPerformSegueWithIdentifier(_ identifier: String, sender sender: AnyObject?) -> Bool`  
判断拥有指定identifier的segue是否会被执行。  
默认返回true。

`func prepareForSegue(_ segue: UIStoryboardSegue, sender sender: AnyObject?)`  
提醒VC一个segue将要被执行。  
默认实现为空。子类可以重写此方法，使用其配置新VC，优先于其显示。segue对象包括了transition信息，包括前后两个VC的引用。  
由于segue可以由多种方式触发，可以使用参数`segue`和`sender`的信息确定不同的逻辑。  

`func performSegueWithIdentifier(_ identifier: String, sender sender: AnyObject?)`  
从当前VC的SB文件中初始化拥有指定identifier的segue。  
通常，segue自动初始化，不会使用此方法。然而，可以在segue不能从SB文件中配置时使用此方法。例如，可能从一个响应晃动或加速器事件的自定义action handler中调用此方法。  
当前VC必须是从SB中读取的。否则此方法会产生异常。  

`func allowedChildViewControllersForUnwindingFromSource(_ source: UIStoryboardUnwindSegueSource) -> [UIViewController]`  
返回应该被搜索到的作为unwind segue的destination的child VC。  
UIKit在搜索一个unwind segue的destination时调用此方法。默认实现返回`childViewControllers`的内容减去`childViewControllerContainingSegueSource:`方法返回的VC。如果需要修改搜索顺序可以重写此方法。例如，一个navigation controller会反转顺序，从navigation stack顶部的VC开始搜索。  

`func childViewControllerContainingSegueSource(_ source: UIStoryboardUnwindSegueSource) -> UIViewController?`  
返回包含unwind segue的source的child VC。  

`func canPerformUnwindSegueAction(_ action: Selector, fromViewController fromViewController: UIViewController, withSender sender: AnyObject) -> Bool`  
在一个VC上调用决定其是否响应一个unwind action。  
当一个unwind segue被触发后，UIKit使用此方法和`allowedChildViewControllersForUnwindingFromSource:`方法确定一个处理unwind segue的合适VC。  
默认实现返回true，在当前VC实现了action同时不是`fromViewController`时。

`func unwindForSegue(_ unwindSegue: UIStoryboardSegue, towardsViewController subsequentVC: UIViewController)`  
当一个unwind segue过渡到新VC时被调用。作用类似于`prepareForSegue:`

### Managing the View  
`var view: UIView!`  
返回VC管理的view。  
此属性储存VC的view hierarchy的root view。默认为nil。  
如果访问此属性且其当前为nil时，VC会自动调用`loadView`方法创建view。  
每个VC对象是其view的基持有者。不应该在多个VC对象间共享一个view对象。唯一的例外是container VC。  
由于访问此属性会导致view被自动加载，可以使用`isViewLoaded`方法判断view当前是否在内存中。访问`isViewLoaded`时不会加载view。  
`UIViewController`类在低内存条件下会自动将此属性设置为nil，在VC对象被释放时也是如此。  

`func isViewLoaded() -> Bool`  
返回view当前是否被加载到内存中。  
调用此方法时不会尝试加载view。

`func loadView()`  
创建VC管理的view。  
不要直接调用此方法。VC在被请求其`view`属性但当前为nil时调用此方法。此方法加载或创建view，将其赋值`view`属性。  
如果VC有一个关联的nib文件，此方法从nib文件中读取view。  
如果使用IB创建view并初始化VC，不要重写此方法。  
如果手动创建view，可以重写此方法。如果这么做，将view hierarchy的root view赋值给`view`属性。创建的view对象必须是唯一的实例，不应和其他任何VC共享。此方法的自定义实现不要调用super。  
如果想执行任何额外的初始化操作，在`viewDidLoad`中执行。  

`func viewDidLoad()`  
在VC的view被加载到内存之后被调用。  
无论以何种方法创建view，此方法都会被调用。通常在此处执行一些额外的初始化操作。  

