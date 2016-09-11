# UIViewController
#### superclass
`UIResponder`


`UIViewController`类提供了管理iOS App中view的基础。一个view controller（VC）管理组成APP的UI的一部分view集合。VC负责读取和部署这些view，管理和view之间的交互，协调view响应任何相关的数据对象。VC也和其他controller协调，包括其他VC，这有助于管理App的整体界面。  
很少直接创建`UIViewController`的实例。相反，创建`UIViewController`的子类，使用这些对象提供需要的特定行为和视觉效果。  
一个VC的主要任务如下：

* 更新view的内容，通常是响应其下的data的变化。
* 响应用户交互。
* 改变view的大小，管理所有界面的布局。

一个VC紧密联系与其管理的view，同时参与用于处理event的responder chain。VC也是`UIResponder`对象，它在responder chain中的位置介于VC的root view和root view的superview之间。

