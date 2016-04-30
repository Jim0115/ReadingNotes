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