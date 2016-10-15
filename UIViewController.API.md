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

`func loadViewIfNeeded()`  
如果view还没被加载，就加载view。iOS 9新增。  

`var viewIfLoaded: UIView? { get }`  
如果VC的view已加载，返回view。否则返回nil。iOS 9新增。

`var title: String?`  
象征VC管理的view的本地化的字符串。  
设置人类可读的string用于描述view。如果view有一个有效的navigation item或tabbar item，设置此属性会同时设置二者的title。  

`var preferredContentSize: CGSize`  
VC的view的倾向大小。  
此属性主要用于在一个popover中显示一个VC的内容，但也可能被用于其他地方。当一个VC正在一个popover中显示时，修改此属性会动画化大小变化。然而，将宽或高修改为0不会产生动画。  

### Presenting View Controllers
`var modalPresentationStyle: UIModalPresentationStyle`  
modally presented VC的呈现风格。 

    enum UIModalPresentationStyle : Int {
        case FullScreen
        case PageSheet
        case FormSheet
        case CurrentContext
        case Custom
        case OverFullScreen
        case OverCurrentContext
        case Popover
        case None
    }
此属性决定了一个VC在被present时将以何种形式显示。在水平compact环境中，model VC总是全屏显示。水平regular环境中，有许多不同选项。所有可能的情况列举在`UIModalPresentationStyle`中。  

`var modalTransitionStyle: UIModalTransitionStyle`  
modal present时的过渡风格。

    enum UIModalTransitionStyle : Int {
        case CoverVertical
        case FlipHorizontal
        case CrossDissolve
        case PartialCurl
    }
此属性决定了VC被`presentViewController:animated:completion:`方法呈现时的动画。必须在present前修改此属性才有效。默认值为`CoverVertical`。  

`var modalInPopover: Bool`  
VC是否应该被一个popover present。  
默认值为false。将此属性设置为true将导致一个VC在显示时禁止在之外的交互。可以使用此属性确保点击popover VC之外的地方不会使其dismiss。  

`func showViewController(_ vc: UIViewController, sender sender: AnyObject?)`  
作为主要内容present一个VC。  
使用此方法，VC不需要知道其是内嵌在一个navigation controller中或是split-view controller中。`UISplitViewController`和`UINavigationViewController`重写此方法，根据其设计处理呈现形式。例如，navigation controller重写此方法，push vc到navigation stack。  
此方法的默认实现是调用`targetViewControllerForAction:sender:`方法定位重写此方法的VC对象。在此对象上调用当前方法，此对象将会以合适的方法显示VC。如果`targetViewControllerForAction:sender:`方法返回nil，此方法将使用window的root VC modelly present `vc`。  

`func showDetailViewController(_ vc: UIViewController, sender sender: AnyObject?)`  
作为次要内容present一个vc。  

`func presentViewController(_ viewControllerToPresent: UIViewController, animated flag: Bool, completion completion: (() -> Void)?)`  
模态present一个VC。  
在水平regular环境中，VC的呈现风格由`modalPresentationStyle`属性指定。在水平compact环境中，VC默认全屏present。如果修改`viewControllerToPresent`对象的presentation controller的delegate，可以动态修改呈现风格。  
调用此方法的对象并不总是处理presentation的对象。每个presentation style有不同的管理规则的行为。例如，全屏presentation必须由全屏的VC产生。如果当前VC不是全屏的，它会向前传递请求到最近的parent，parent处理或继续向前传递。  
在显示VC之前，方法会根据presentation style修改VC的view的大小。对于大多数style，resulting view会根据被present的VC的`modalTransitionStyle`属性确定动画。对于自定义presentation，view使用被present的VC的transitioning delegate确定动画。对于current context presentation，VC会使用当前VC的transition style。  
completion handler在`viewDidAppear:`方法之后在presented VC上被调用。  

`func dismissViewControllerAnimated(_ flag: Bool, completion completion: (() -> Void)?)`  
dismiss被当前VC modally present的VC。  
presenting VC负责dismiss其present的VC。如果你在presented VC上调用此方法，UIKit会要求presenting VC处理dismissal。  
如果连续present多个VC，形成了一个presented VC的栈，在栈下方的VC上调用此方法会立即dismiss其childVC和childVC上方的所有VC。此时，只有最上方view的dismiss会有动画，其他会直接从栈中移除。  
如果想保留被present的VC的引用，在调用此方法前使用`presentedViewController`属性即可。  
completion handler会在`viewDidAppear:`方法之后被调用在presented VC上。  

`var definesPresentationContext: Bool`  
当此VC或其某个后代present一个VC时，此VC是否被覆盖。  
当使用`UIModalPresentationCurrentContext`或`UIModalPresentationOverCurrentContext`style present一个VC时，此属性控制某个存在于VC hierarchy中的VC真正被新内容覆盖。当一个基于context的presentation发生时，UIKit会从presentingVC开始遍历VC hierarchy。如果找到某个VC的此选项为true，则要求此VC present newVC。如果没有VC定义presentation context，UIKit 会要求window的root VC处理presentation。  
默认值为false。有些系统提供的VC，例如`UINavigationController`，默认值为true。  

`var providesPresentationContextTransitionStyle: Bool`  
VC是否指定其present的VC的transition style。  
当一个VC的`definesPresentationContext`属性为true时，其可以使用其自身的transition style替换其presented VC的。当此属性为true时，当前VC的transition style用于替换presentedVC的style。当此属性为false时，UIKit使用presentedVC的tansition style。此属性默认为false。

`func disablesAutomaticKeyboardDismissal() -> Bool`  
当前的input view是否会在changing control时自动dismiss。  
true：不会dismiss。false：可能被dismiss。  
重写子类的此方法用于允许或禁止当前的input view在从一个需要input view的control切换到到一个不需要input view的control时是否会dismiss。在通常情况下，当用户点击了一个需要input view的control时，系统会自动显示input view。点击一个不需要input view的control会导致input view被dismiss，但不是一定会发生。重写此方法允许或禁止那些特殊情况dismiss input view。  
默认实现中，当VC的`modalPresentationStyle`属性为`.formSheet`时，返回true。其他返回false。因此，系统通常不允许键盘在modal form状态dismiss。  

### Supporting Custom Transitions and Presentations
`weak var transitioningDelegate: UIViewControllerTransitioningDelegate?`  
提供过渡动画，交互controller，自定义呈现controller对象的delegate对象。  
当VC的`modalPresentationStyle`属性为`.custom`时，UIKit会使用此属性中的对象提供VC的过渡和呈现。transition delegate必须实现`UIViewControllerTransitioningDelegate`协议。

`func transitionCoordinator() -> UIViewControllerTransitionCoordinator?`  
返回活动的过渡协调者对象。  
当存在进行中的presentation或dismissal时，此方法返回与此transition关联的transition coordinator。如果没有关联当前VC的执行中的transition，UIKit会向VC的祖先查找coordinator对象。使用此对象创建额外的动画并将其与过渡动画同步。  
Container VC可以重写此方法但大多数情况下都不需要。如果重写了此方法，首先要调用super检查是否有一个合适的coordinator可返回。

`func targetViewControllerForAction(_ action: Selector, sender sender: AnyObject?) -> UIViewController?`  
返回响应action的VC。  
此方法返回重写了`action`参数对应方法的VC。如果当前VC没有重写该方法，UIKit会沿着view hierachy寻找第一个重写该方法的VC。如果没有VC能够处理action，返回nil。  
VC通过其`canPerformAction:withSender:`确定其能否响应某个action。

`var presentationController: UIPresentationController? { get }`  
管理当前VC的最近的presentation controller。  
如果VC或其某个祖先被一个presentation controller管理，此属性包含该对象。如果没有被一个presentation controller管理，此属性返回nil。  
如果还没有present一个VC，访问此属性会创建一个基于VC的`modalPresentationStyle`属性的presentation controller。因此，在访问此属性前必须要先设置好`modalPresentationStyle`属性。  

`var popoverPresentationController: UIPopoverPresentationController? { get }`  
与上一属性类似，如果在访问此属性前将`modalPresentationStyle`设置为`.popover`，那么通过此属性可以获得popover的presentation controller。如果`modalPresentationStyle`为其他值，那么此属性将返回nil。

### Responding to View Events  
`func viewWillAppear(_ animated: Bool)`  
提醒VC，其view将要被加入view hierarchy。  
此方法在VC的view将要被加入一个view hierarchy前被调用，同时也在所有view的动画被配置前。可以重写此方法用于执行与显示view相关的任务。例如，使用此方法改变状态栏的方向或风格以协调被present的view的方向和风格。如果重写此方法，必须在实现中某部分调用super。  
如果某个VC popover了一个VC，那么在被popover的VC dismiss时此方法不会被调用。

`func viewDidAppear(_ animated: Bool)`  
提醒VC，其view已经被加入view hierarchy。  
可以重写此方法执行与present view相关的额外操作。如果重写此方法，必须在实现中某部分调用super。  
如果某个VC popover了一个VC，那么在被popover的VC dismiss时此方法不会被调用。

`func viewWillDisappear(_ animated: Bool)`  
提醒VC，其view将要从一个view hierarchy中被移除。  
此方法在view被移除前被调用，同时也在动画被配置前。  
子类可以重写此方法用于提交修改，resign view的first responder状态，或执行其他相关操作。例如，使用此方法还原在`viewWillAppear:`方法中状态栏的方向和位置做的更改。  
如果重写此方法，必须在实现中某部分调用super。

`func viewDidDisappear(_ animated: Bool)`  
提醒VC，其view已经从一个view hierarchy中被移除。  
重写此方法，执行额外的与view的dismiss或hiding相关的任务。如果重写此方法，必须在实现中某部分调用super。

### Configuring the View’s Layout Behavior
`func viewWillLayoutSubviews()`  
提醒VC其view将要布局view的subviews。  
当一个view的bounds变化时，view会调整其subviews的位置。VC可以重写此方法，在view布局其subview前执行操作。此方法默认实现为空。

`func viewDidLayoutSubviews()`  
提醒VC其view已经布局view的subviews。  
当一个view的bounds变化时，view会调整其subviews的位置，之后系统会调用此方法。然而，此方法被调用并不代表view的subviews的独立布局被修改。每个subview负责调整其自身的布局。  
默认实现为空。

`func updateViewConstraints()`  
当VC的view需要更新其约束时，此方法被调用。  
重写此方法用于优化约束的变化。  
**在变化发生后立即更新约束总是非常清晰和方便。例如，如果想要响应按钮的点击改变约束，直接在按钮的action方法中改变约束即可。**  
**重写此方法的情况只有两种：在对应位置改变约束太慢，view产生了大量的冗余约束。**  
在view上调用`setNeedUpdateConstraints`方法安排改变约束。系统会在布局发生时调用此方法的自定义实现。  
此方法的实现必须尽可能高效。不要禁用所有约束，之后重新启用需要的约束。相反，app必须有某种追踪约束的方法，在每次update pass中使约束生效。只有变化的对象需要变化。在每次update pass中，必须确保有当前状态的合适约束。  
不要在此方法实现中调用`setNeedUpdateConstraints`。  
在实现的最后必须调用super。  

`var bottomLayoutGuide: UILayoutSupport { get }`  
代表屏幕内容的垂直方向底端，用于建立约束。  
此属性在VC在屏幕最前端时才生效。表示垂直方向最底端，不包括tabbar和toolbar。此方法实现了`UILayoutSupport`协议，可以将其用作`NSLayoutConstraint`对象的某个item。  
此属性的值，尤其是，`length`属性的值会在请求时返回。此值被VC或其containerVC（navigation、tab）约束，规则如下：

* 不在container中的VC约束此值用于表示可见的tabbar或toolbar的顶端。或者表示VC的view的底端。
* 包含在containerVC中的VC不设置此值。相反，containerVC约束此值表示可见的tabbar或toolbar的顶端。或者表示VC的view的底端。

如果一个containerVC的toolbar或tabbar是可见不透明的,container会将最前的VC的view布局为底端与bar的顶端相邻。这种情况下，此属性的值为0。  
在`viewDidLayoutSubviews`方法的自定义实现中查询此属性。  
当使用SB布局时，此对象在IB outline view中作为VC对象的child可用。使用IB添加一个bottom layout guide提供对iOS 6的向后兼容性。  

`var topLayoutGuide: UILayoutSupport { get }`  
代表屏幕内容的垂直方向顶端，用于建立约束。

`var edgesForExtendedLayout: UIRectEdge`  
用于布局的拓展边界。  
此属性仅当VC被嵌在一个例如`UINavigationController`的container中时提供。window的rootVC不响应此属性。默认值为`.All`。  

`var extendedLayoutIncludesOpaqueBars: Bool`  
是否包括不透明bar的拓展布局。默认为false。

`var automaticallyAdjustsScrollViewInsets: Bool`  
VC是否应该自动调整其scrollview的inset。  
默认值为true，允许VC根据被状态栏、导航栏、工具栏和tab栏占用的空间调整期scrollview的inset。将此属性设置为false时可以手动调整inset，适用于有多个scrollview的情况。

### Testing for Specific Kinds of View Transitions
`func isMovingFromParentViewController() -> Bool`  
VC是否在被从parent移除的过程中。  
true表示VC因为从containerVC中被移除，正在消失。  
此方法只有在`viewWillDisappear:`和`viewDidDisappear:`调用此方法才可能返回true。  

`func isMovingToParentViewController() -> Bool`  
与上一方法完全相反。

`func isBeingDismissed() -> Bool`  
VC是否处于被其某个祖先dismiss的过程中。  
true表示VC在之前被present，正处于被dismiss的过程中。  
此方法只有在`viewWillDisappear:`和`viewDidDisappear:`调用此方法才可能返回true。  

`func isBeingPresented() -> Bool`  
与上一方法完全相反。

### Configuring the View Rotation Settings
`func shouldAutorotate() -> Bool`  
VC的content是否应该自动旋转。  
默认返回true。iOS 5之前会默认返回false。  

`func supportedInterfaceOrientations() -> UIInterfaceOrientationMask`  
VC支持的界面方向。返回值不得为0。  

	struct UIInterfaceOrientationMask : OptionSetType {
	    init(rawValue rawValue: UInt)
	    static var Portrait: UIInterfaceOrientationMask { get }
	    static var LandscapeLeft: UIInterfaceOrientationMask { get }
	    static var LandscapeRight: UIInterfaceOrientationMask { get }
	    static var PortraitUpsideDown: UIInterfaceOrientationMask { get }
	    static var Landscape: UIInterfaceOrientationMask { get }
	    static var All: UIInterfaceOrientationMask { get }
	    static var AllButUpsideDown: UIInterfaceOrientationMask { get }
	}
当用户改变设备方向时，系统会在rootVC或最上方全屏VC上调用此方法。如果VC支持新的方向，window和VC会被旋转到新的方向。此方法只在VC的`shouldAutorotate`方法返回true时被调用。  
重写此方法报告VC支持的所有方向。iPad的默认值为`.All`而iPhone的默认值为`.AllButUpsideDown`。  
系统会将App支持的方向与VC支持的方向求交，确定旋转的方向。  

`func preferredInterfaceOrientationForPresentation() -> UIInterfaceOrientation`  
返回present此VC时使用的界面方向。  
系统会在全屏present VC时调用此方法。当VC支持两个以上的方向，其中一个在内容显示上最佳，重写此方法返回该方向。  
如果VC实现了此方法，VC的view会在对应方向上展示（之后可被旋转到其他支持的方向）。如果没有实现此方向，系统会使用当前状态栏方向进行present。  

`class func attemptRotationToDeviceOrientation()`  
尝试旋转所有window到设备所在的方向。  
有些VC可能需要使用app特定的条件确定支持的方向。如果是这样，当支持的方向改变时，App应该立即调用此方法。系统会尝试将VC旋转到当前方向。  

### Adapting to Environment Changes
`func collapseSecondaryViewController(_ secondaryViewController: UIViewController, forSplitViewController splitViewController: UISplitViewController)`  
当一个splitVC过渡到一个compact-width size class时被调用。  
如果没有重写`splitViewController:collapseSecondaryViewController:ontoPrimaryViewController:`方法，默认的行为是显示splitVC的primaryVC。  

`func separateSecondaryViewControllerForSplitViewController(_ splitViewController: UISplitViewController) -> UIViewController?`  
当一个splitVC过渡到regular-width size class时被调用。  
如果没有重写`splitViewController:separateSecondaryViewControllerFromPrimaryViewController:`方法，默认返回前一个secondaryVC。  

### Managing Child View Controllers in a Custom Container
`var childViewControllers: [UIViewController] { get }`  
当前VC的child VC数组。  
此属性不包括任何presentedVC。此属性智能呗自定义containerVC读取。  

`func addChildViewController(_ childController: UIViewController)`  
将一个VC作为当前VC的child添加。  
此方法在currentVC和参数`childController`之间建立一个parent-child关系。这种关系在将childVC的view嵌入currentVC的内容中时是必要的。如果参数`childController`已经是其他containerVC的child，在其被添加前会先被移除。  
此方法只能被自定义containerVC调用。如果重写此方法，必须在实现中调用super。  

`func removeFromParentViewController()`  
将VC从其parent中移除。  
此方法只能被自定义containerVC调用。如果重写此方法，必须在实现中调用super。

`func transitionFromViewController(_ fromViewController: UIViewController, toViewController toViewController: UIViewController, duration duration: NSTimeInterval, options options: UIViewAnimationOptions, animations animations: (() -> Void)?, completion completion: ((Bool) -> Void)?)`  
在VC的两个childVC之间过渡。  
此方法将secondVC的view添加到view hierarchy中，之后执行`animation`block中的动画。在动画结束时，将firstVC的view从view hierarchy中移除。  
此方法只能被自定义containerVC调用。如果重写此方法，必须在实现中调用super。 

`func shouldAutomaticallyForwardAppearanceMethods() -> Bool`  
appearance methods是否转发给childVC。  
默认实现返回true。实现了containment逻辑的`UIViewController`的子类可能重写此方法用于控制方法的转发。如果此方法返回false，当child的view将要出现或消失时，必须负责通知child。通过调用child的`beginAppearanceTransition:animated:`和`endAppearanceTransition`方法。  

`func beginAppearanceTransition(_ isAppearing: Bool, animated animated: Bool)`  
告诉child controller其显示将要改变。  

- `isAppearing`：true表示childVC的view将要被添加到view hierarchy中，false表示将要被移除
- `animated`：动画

如果实现了一个自定义container controller，使用此方法告诉child其view将要出现或消失。不要直接在此方法中调用`viewWillAppear:`, `viewWillDisappear:`, `viewDidAppear:`或`viewDidDisappear:`。  

`func endAppearanceTransition()`  
如果实现了一个自定义container controller，使用此方法告诉child，view过渡已经完成。  

`func setOverrideTraitCollection(_ collection: UITraitCollection?, forChildViewController childViewController: UIViewController)`  
改变特定的childVC的traits。  
`UITraitCollection`：iOS 8引入的，用于描述设备的环境信息，包括缩放比例，水平/垂直方向上的size class，设备类型，force touch可用性。  
通常，traits在从parentVC到childVC传递时不会被修改。当实现了自定义的containerVC时，可以使用此方法修改任何包含的childVC的traits以更合适布局。进行此项修改会改变与此child相关的其他VC的行为。也可以使用此方法强制使所有childVC使用同样的traits，忽视其所在的真实traits环境。  

`func overrideTraitCollectionForChildViewController(_ childViewController: UIViewController) -> UITraitCollection?`  
获得一个childVC的trait collection。  
使用此方法获得一个childVC的trait collection。之后可以使用上一方法设置childVC的trait collection。  

### Responding to Containment Events
`func willMoveToParentViewController(_ parent: UIViewController?)`  
VC将要被添加到containerVC或从containerVC中移除时被调用。  
子类可以重写此方法，用于确定其何时被加入到container中。  
如果实现自定义containerVC，在调用childVC的`removeFromParentViewController`之前必须调用其`willMoveToParentViewController:`方法。  
当自定义container调用`addChildViewController:`方法时，其会在添加前自动调用childVC的`willMoveToParentViewController:`方法。   

`func didMoveToParentViewController(_ parent: UIViewController?)`  
VC被添加到containerVC或从containerVC中移除之后被调用。  
如果实现自定义containerVC，必须在过渡动画结束后调用`didMoveToParentViewController:`。如果没有过渡动画，在`addChildViewController:`方法结束后立即调用`didMoveToParentViewController:`。  
`removeFromParentViewController`会在childVC被移除后自动调用`didMoveToParentViewController:`。  

### Getting Other Related View Controllers
`var presentingViewController: UIViewController? { get }`  
present此VC的VC。  
当使用`presentViewController:animated:completion:`方法modally present一个VC时，被present的VC此属性被设置为present其的VC。如果当前VC未被modally present，但其某个祖先是，则此属性包含present当前VC祖先的VC。如果两种情况都不符合，此属性为nil。  

`var presentedViewController: UIViewController? { get }`  
被此VC present的VC或VC hierarchy中其某个祖先。  
当使用`presentViewController:animated:completion:`方法present一个VC时，调用此方法的VC会设置此属性为被present的VC。如果当前VC没有present另一个VC，则此属性为空。  

`weak var parentViewController: UIViewController? { get }`  
接收者的parentVC。  
如果接收者是某个containerVC的childVC，此属性为containerVC。如果接收者没有parent，此属性为nil。  
在iOS 5之前，如果一个view没有parentVC，同时其被present，则此属性会返回presentingVC。在iOS 5之后，这种情况不会再发生。使用`presentingViewController`访问presentingVC。  

`var navigationController: UINavigationController? { get }`  
在VC hierarchy中，最近的navigation controller祖先。  
如果VC或其某个祖先是一个navigation controller的child，此属性返回该navigation controller。如果VC没有被包含在某个navigation controller中，此属性返回nil。  

`var splitViewController: UISplitViewController? { get }`  
在VC hierarchy中，最近的split controller祖先。  
如果VC或其某个祖先是一个split controller的child，此属性返回该split controller。如果VC没有被包含在某个split controller中，此属性返回nil。  

`var tabBarController: UITabBarController? { get }`  
返回VC hierarchy中，最近的tabbar controller祖先。  
如果VC或其某个祖先是一个tabbar controller的child，此属性返回该tabbar controller。如果VC没有被包含在某个tabbar controller中，此属性返回nil。 

### Handling Memory Warnings
`func didReceiveMemoryWarning()`  
当App收到内存警告时，此方法被发送给VC。  
App不应该直接调用此方法。相反，系统会在可用内存低时调用此方法。  
可以重写此方法，释放掉被VC占用的额外内存。如果重写此方法，必须调用super。  

### Managing State Restoration
`var restorationIdentifier: String?`  
VC是否支持状态恢复的identifier。  
此属性代表VC和其内容是否应该被保留，同时使用此属性在恢复过程中标记VC。此属性的默认值为nil，表示VC不会被储存。对此属性赋值一个string对象使系统知道VC应该被保存。而且，string的内容标记VC的用途。  
在随后的启动中，UIKit会要求重新创建上次App运行时的VC。  
简单的设置此属性并不足以确保VC被保留和恢复。VC的所有parent必须也有restoration identifier。  

`var restorationClass: AnyObject.Type?`  
当恢复app状态时负责重写创建VC的类。  
如果一个VC有关联的restoration class，该类的`viewControllerWithRestorationIdentifierPath:coder:`方法将在恢复时被调用。该方法负责返回指定VC的VC对象。如果没有指定VC的restoration class，状态恢复engine会要求app delegate提供一个VC对象。  

`func encodeRestorableStateWithCoder(_ coder: NSCoder)`  
为VC编码状态相关的信息。  
不要直接调用此方法。系统会在状态保留过程中调用此方法，给你的VC或VC子类储存状态相关信息的机会。  
当确定需要储存的数据时，写入能够恢复到当前状态的最少的数据。写入的信息应该是不能轻易重新创建的，例如用户当前的选择。可能也需要储存VC正在使用对象的引用，但绝对不要直接写入数据对象本身。相反，应该储存足够的信息用于从app的主数据结构中再次取回数据对象。  
此方法不是将app的数据结构于磁盘持久化的选项。应该将数据存在iCloud或使用现有技术存于本地文件系统。此方法只是用于保存配置状态或其他与app的UI相关的信息。应该考虑向coder写入的任何数据都是可清除的，并准备在之后的启动过程中将这些数据设为不可用。  
