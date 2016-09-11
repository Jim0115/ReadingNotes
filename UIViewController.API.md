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


