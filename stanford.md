# Class 8 @ April 22, 2016
## Animation
    
    + animateWithDuration:delay:options:animations:completion: // of UIView
    
只能作用于frame, transform and alpha

### UIViewAnimationOptions
`BeginFromCurrentState` // interrupt others, in-progress animations of there properties  
`AllowUserInteraction` // allow gestures to get processed while animation is in progress  
`LayoutSubviews` // animate the relayout of subviews along with a parent's animation  动画过程中保证子视图跟随运动  
`repeat` // repeat indefinitely  
`Autoreverse` // play animation forwards, then backwards  
`OverrideInheritedDuration` // if not set, use duration of any in-progress animation  
`OverrideInheritedCurve` // if not set, just interpolate between current and end state image  
`CurveEaseInEaseOut` // slower at the beginning, normal throughout, then slow at end  
`CurseEaseIn` // slower at the beginning, but then constant through the rest  
`CurveLinear` // same speed throughout

---
### 修改view的其他属性时使用

    + (void)transitionWithView:(UIView *)view 
                      duration:(NSTimeInterval)duration
                       options:(UIViewAnimationOptions)options
                    animations:(void (^)(void))animations 
                    completion:(void (^)(BOOL finished))completion
                    
#### Options
`UIViewAnimationOptionsTransitionFlipFrom{Left, Right, Top, Bottom}` // flip view over  
`UIViewAnimationOptionsTransitionCrossDissolve` // dissolving from old to new state  
`UIViewAnimationOptionsTransitionCurl{Up, Down}` // curling up or down 卷起  
  
将在屏幕外渲染新视图，然后通过动画进行视图切换

### 使用动画过渡到新View

    + (void)transitionFromView:(UIView *)fromView
                        toView:(UIView *)toView
                      duration:(NSTimeInterval)duration
                       options:(UIViewAnimationOptions)options
                    completion:(void (^)(BOOL finished))completion
                    
# Class 11 @April 29, 2016
## 动态TableView使用Segue进行跳转时，获取用户点击的cell
`NSIndexPath* indexPath = [self.tableView indexPathForCell:sender];`
## popover
`UIPopoverController`不是`UIViewController`的子类  
`UIStoryboardPopoverSegue`是`UIStoryboardSegue`的子类

    - (void)prepareForSegue:(UIStoryboardSegue)segue sender:(id)sender {
        if ([segue isKindOfClass:[UIStoryboardPopoverSegue class]]) {
            UIPopoverController* popoverController = ((UIStoryboardPopoverSegue *)segue).popoverController;
        }
    }
    
# class 12 @ April 30, 2016
## UIManagedDocument
A container for your Core Data database.
### open or create
    NSFileManager* fileManager = [NSFileManager defaultManager];
    NSURL* documentsDirectory = [[fileManager URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask] firstObject];
    
    NSString* documentName = @"myDocument";
    NSURL* url = [documentsDirectory URLByAppendingPathComponent:documentName];
      
    UIManagedDocument* document = [[UIManagedDocument alloc] initWithFileURL:url];

Check the file exists.  
`BOOL fileExists = [[NSFileManager defaultManager] fileExistsAtPath:[url path]];`  
if it does, open it:  
`[document openWithCompletionHandler:^(BOOL success) { /* do sth */ }]; `  
if it doesn't, create it:  
`[document saveToURL:url forSaveOperation:UIDocumentSaveForCreating completionHandler:^(BOOL success) { /* do sth */ }];`  
**两个方法都是异步的。**
### check the state of document
    if (self.document.documentState == UIDocumentStateNormal) {
        // start using document
    }
    
    enum {
       UIDocumentStateNormal          = 0, 
       UIDocumentStateClosed          = 1 << 0, // have not done the open or create
       UIDocumentStateInConflict      = 1 << 1, // some other device change it via iCloud
       UIDocumentStateSavingError     = 1 << 2, // success will be NO in completion handler

       UIDocumentStateEditingDisabled = 1 << 3 // temporary situation, try again
       UIDocumentStateProgressAvailable = 1 << 4
    };
    typedef NSInteger UIDocumentState;
    
### if the document is ready
    if (self.document.documentState == UIDocumentStateNormal) {
        NSManagedObjectContext* context = self.document.managedObjectContext;
    }
    
### feature
* autosave, but you can save it manually

        [document saveToURL:document.fileURL
           forSaveOperation:UIDocumentSaveForOverwriting
          completionHandler:^(BOOL success) {  }];
        // asynchronous
* autoclose

### KVO on managedObjectContext

    - (void)viewDidAppear:(BOOL)animated {
        [super viewDidAppear:animated];
        [center addObserver:self
                   selector:@selector(contextChanged:)
                       name:NSManagedObjectContextDidSaveNotification
                     object:document.managedObjectContext];
    }
    
    - (void)viewWillDisappear:(BOOL)animated {
        [center removeObserver:self
                          name:NSManagedObjectContextDidSaveNotification
                        object:document.managedObjectContext];
        [super viewWillDisappear:animated];
    }
    
### Merging changes
`- (void)mergeChangesFromContextDidSaveNotification:(NSNotification *)notification;`

## Core Data
### Inserting objects into the database
    NSManagedObjectContext* context = aDocument.managedObjectContext;
    NSManagedObject* photo = [NSEntityDescription insertNewObjectForEntityForName:@"Photo" inManagedObjectContext:context];
    
### 生成NSManagedObject的子类
#### 为什么实现文件中只有使用@Dynamic标记的属性？
OC在运行时，如果调用了属性的setter或getter，而setter和getter并没有被实现，将会自动尝试使用`setValue:Forkey:`和`valueForKey:`如果失败，返回unknown selector错误，否则程序将正确执行。

### Querying
#### NSSortDescriotor
      NSSortDescriptor* sortDescripter = [NSSortDescriptor sortDescriptorWithKey:@"title"
                                                                       ascending:YES
                                                                        selector:@selector(localizedStandardCompare:)];
                                                                        
#### `request.sortDescriptors`
request中的sortDescriptors，包含多个sortDescriptor，对NSManagedObject进行多依据的排序。
#### NSPerdicate
specify exactlt **which objects** we want from the database.
#### 返回内容
返回nil，有错误发生，检查错误  
返回空数组，没有找到满足查询条件的object    
#### some others
从数据库中获取到的object保存在数组中，但这些数组中的对象仅仅是占位符，只有当使用这些数组中的对象时，才真正获取到这些对象。
#### Thread Safety
NSManagedObjectContext不是线程安全的，但是由于Core Data通常速度很快，极少使用到多线程。  
如果需要串行的执行操作，使用:

    [context performBlock:^{
        // do stuff with context in its safe queue (the queue it was created on)
    }];
    // or
    [context performBlockAndWait:^{}];
    
# Class 13 @ May 3, 2016
## Core Date and UITableView
### NSFetchedResultsController
hooks an `NSFetchRequset` up to a `UITableViewController`  
Usually you'll have a `NSFetchResultsController` @property in your `UITableViewController`.  
It will be hooked up to an `NSFetchRequest` that returns the data you want to show in your table.  
Then use it to answer all your `UITableViewDataSource` protocol's questions.

    - (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
      return self.resultController.sections.count;
    }
    
    - (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
      return [self.resultController.sections objectAtIndex:section].numberOfObjects;
    }
    
**Very important method**   
`- (NSManagedObject *)objectAtIndexPath:(NSIndexPath *)indexPath;`

    - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
      RecordCell *cell = [tableView dequeueReusableCellWithIdentifier:@"recordCell"
                                                         forIndexPath:indexPath];
      // example
      Photo* photo = (Photo *)[self.fetchedResultsController objectAtIndexPath:indexPath];
      
      // configure the cell
           
      return cell;
    }
    
#### How to create?
    NSFetchRequest* request = [NSFetchRequest fetchRequestWithEntityName:@"Record"];
    request.sortDescriptors = @[[NSSortDescriptor sortDescriptorWithKey:@"date" ascending:false selector:@selector(compare:)]];
    //    request.predicate = [NSPredicate predicateWithFormat:@"format"];
    _resultController = [[NSFetchedResultsController alloc] initWithFetchRequest:request managedObjectContext:self.context sectionNameKeyPath:@"date" cacheName:@"recordCache"];

cacheName为nil时不会缓存

# Class 15 @May 14, 2016
### Embed Segue
使用一个Container将另一个VC显示为当前VC的一部分。  
系统将在初始化是自动调用`performSegue`方法，在`prepareForSegue`方法中对`segue.destinationViewController`进行操作即可对ContainerVC进行初始化。  
调用`performSegue`方法将会出错

# Class 16 @ May 14, 2016
### Unwind Segue
使用unwind segue进行返回，可以一次解开多层。  
unwind segue是唯一一种不创建新VC的segue，只是返回之前的VC。

### dismissViewController
Dismisses the view controller that was presented modally by the view controller.  
实际上应该调用`presentingViewController.dismissViewControllerAnimated...`  
If you call this method on the presented view controller itself, UIKit asks the presenting view controller to handle the dismissal.

### UITextField
`Editing Changed`用来记录输入文本的改变