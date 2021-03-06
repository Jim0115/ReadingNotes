# Core Data
## Chapter 3: The Core Data Stack
![image](http://www.cocoachina.com/cms/uploads/allimg/130911/4673_130911165500_1.png =500x)
* NSManagedObjectModel* NSPersistentStore* NSPersistentStoreCoordinator * NSManagedObjectContext
### NSManagedObjectModel
`NSManagedObjectModel`可以被看作DataBase Schema，从中可以取出表(`NSEntityDescription`)，在表中可以插入行(`NSManagedObject`)。代表整个App的Model。

### NSPersistentStore
This class is the abstract base class for all Core Data persistent stores.  
Core Data provides four store types—SQLite, Binary, XML, and In-Memory (the XML store is not available on iOS)  
具体的数据存储方式，决定数据库中数据以何种方式写入磁盘中。

### NSPersistentStoreCoordinator
`NSPersistentStoreCoordinator`是连接`NSManagedObjectModel`和`NSPersistentStore`的桥梁。负责从`NSPersistentStore`中存取数据。  
`NSPersistentStoreCoordinator`隐藏数据如何储存的细节，原因有以下两点：

1. `NSManagedObjectContext`不需要知道数据以何种方式储存。
2. 如果存在多个`NSPersistentStore`，`NSManagedObjectContext`不需要知道具体的`NSManagedObjectContext`，由`NSPersistentStoreCoordinator`决定如何进行存储。

### NSManagedObjectContext
`NSManagedObjectContext`是最接近栈顶的部分。

* context是一个在内存中的暂存器(scratchpad)，从中取得`NSManagedObject`
* 对于所有managedObject的操作都是在一个context中完成的
* 在调用`NSManagedObjectContext`的`save()`方法前，context的任何操作不会影响到下层的磁盘上的数据。

以下5点为重点：

1. context管理它所获取或创建的object的生命周期，生命周期包括错误处理，反向关系处理和有效性验证；
2. 如果没有关联的context，object不能单独存在。通过object可以获取与其相关联的context；

        let managedContext = employee.managedObjectContext // unowned
3. 一旦object被一个context关联，将不能再更改语气关联的context；
4. 一个App可以使用多个context。事实上，由于context只是一个内存中的暂存器，甚至可以通过不同的context对磁盘中的同一份数据进行读取；
5. context不是线程安全的

## Chapter 4: Intermediate Fetching(中间获取)
### NSFetchRequest: the star of the show
获取一个`NSFetchRequest`的四种方法：
    
    // 1
    let fetchRequest1 = NSFetchRequest()
    let entity = NSEntityDescription.entityForName("Person", inManagedObjectContext: managedContext)
    fetchRequest1.entity = entity
    
    // 2
    let fetchRequest2 = NSFetchRequest(entityName: "Person")
    
    // 3 
    let fetchRequest3: NSFetchRequest? = managedObjectModel.fetchRequestTemplateForName("PeopleFR")
    
    // 4
    let fetchRequest4: NSFetchRequest? = managedObjectModel.fetchRequestFromTemplateWithName("PeopleFR", substitutionVariables: ["NAME" : "Tom"])
    
1. 创建一个`fetchRequest`，从`context`通过name获取一个`entity`，将这个`entity`传给`fetchRequest`作为获取`entity`的类型
2. 方法1的简单方法，直接使用name构造一个`fetchRequest`
3. 使用XCode中在`NSManagedObjectModel`中预定义的`fetchRequest`可以通过此方法获取
4. 在方法3创建的`fetchRequest`中加入额外的条件。

#### NSFetchRequestResultType
设置`fetchResult`的`resultType`属性可以对获取数据的类型进行选择，有以下四种：

    struct NSFetchRequestResultType : OptionSetType {
        init(rawValue rawValue: UInt)
        static var ManagedObjectResultType: NSFetchRequestResultType { get }
        static var ManagedObjectIDResultType: NSFetchRequestResultType { get }
        static var DictionaryResultType: NSFetchRequestResultType { get }
        static var CountResultType: NSFetchRequestResultType { get }
    }
    
* ManagedObjectResultType：返回managed object（默认）
* ManagedObjectIDResultType：返回object唯一的identifier代替object
* NSDictionaryResultType：将每个object的所有属性放入一个字典中，返回所有object构成的字典数组。
* NSCountResultType：返回匹配request的object的个数


    `[<DogWalk.Dog: 0x7fe52b7b59d0> (entity: Dog; id: 0xd000000000040000 <x-coredata://4D07D5C6-732F-46FD-9DB9-CD1A029DC670/Dog/p1> ; data: <fault>), <DogWalk.Dog: 0x7fe52b7b6000> (entity: Dog; id: 0xd000000000080000 <x-coredata://4D07D5C6-732F-46FD-9DB9-CD1A029DC670/Dog/p2> ; data: <fault>)]`
    
    `[0xd000000000040000 <x-coredata://4D07D5C6-732F-46FD-9DB9-CD1A029DC670/Dog/p1>,
    0xd000000000080000 <x-coredata://4D07D5C6-732F-46FD-9DB9-CD1A029DC670/Dog/p2>]`
    
    `[2]`
    
    `[{
        name = Fido;
    }, {
        name = haha;
    }]`
    
对于具体fetchRequest不同resultType的不同结果。

### NSAsynchronousFetchRequest
异步fetchrequest，和`NSFetchRequest`同为`NSPersistentStoreRequest`的子类，构造方法如下：

    NSAsynchronousFetchRequest(fetchRequest: NSFetchRequest) { (NSAsynchronousFetchResult) in /* code here */}

使用

    public func executeRequest(request: NSPersistentStoreRequest) -> NSPersistentStoreResult 

方法执行请求，在request的block中对结果进行操作，方法的返回值不一定可用。**注意循环引用问题**

### NSBatchUpdateRequest
在之前的操作中，修改一个managedObject的内容需要fetch-modify-save，在处理大量数据时很不方便。使用`NSBatchUpdateRequest`可以解决这一问题。`NSBatchUpdateRequest`同样是`NSPersistentStoreRequest`的子类，同样使用`executeRequest`方法执行。类似于SQL中的`update`

    let updateRequest = NSBatchUpdateRequest(entityName: "Dog")
    
    updateRequest.predicate = NSPredicate(format: "name == %@", "bar")
    updateRequest.propertiesToUpdate = ["name" : "foo"]
    updateRequest.resultType = .UpdatedObjectsCountResultType
    
    do {
      let updateResult = try managedContext.executeRequest(updateRequest) as! NSBatchUpdateResult
      // do sth here
    } catch let error as NSError {
      print(error.localizedDescription)
    }
    
将name为"bar"的Dog的name更改为foo的操作如上。  
`NSBatchUpdateRequest`的`resultType`有以下三种：

    public enum NSBatchUpdateRequestResultType : UInt {
      case StatusOnlyResultType // Don't return anything
      case UpdatedObjectIDsResultType // Return the object IDs of the rows that were updated
      case UpdatedObjectsCountResultType // Return the number of rows that were updated
    }
    
### NSBatchDeleteRequest
包装一个`NSFetchRequest`作为删除的标准：

    let fetchRequest = NSFetchRequest(entityName: "Dog")
    fetchRequest.predicate = NSPredicate(format: "name = %@", "Fido")
    let deleteRequest = NSBatchDeleteRequest(fetchRequest: fetchRequest)
    deleteRequest.resultType = .ResultTypeCount
    
    do {
      let deleteResult = try managedContext.executeRequest(deleteRequest) as! NSBatchDeleteResult
      // do sth here
      print(deleteResult.result)
    } catch let error as NSError {
      print(error.localizedDescription)
    }
    
删除所有name为"Fido"的Dog  
与updateRequest类似，有三种对应的`resultType`：

    public enum NSBatchDeleteRequestResultType : UInt {
      case ResultTypeStatusOnly // Don't return anything
      case ResultTypeObjectIDs // Return the object IDs of the rows that were deleted
      case ResultTypeCount // Return the object IDs of the rows that were deleted
    }
    
##### NSBatchUpdateRequest适用于iOS 8以上，NSBatchDeleteRequest适用于iOS 9以上

---

# Chapter 5: NSFetchResultsController
Core Data通常与TableView，通常的做法是从数据fetch到managedObject，用其作为tableView的dataSource。为了简化这个过程，iOS提供了`NSFetchedResultsController`作为二者间的连接桥梁。    
对于通常的fetchRequest，不需要指定其sortDescriptor。但对于`NSFetchResultsController`使用的fetchRequest则必须指定至少一个sortDescriptor，否则程序crash。这样做的原因是为了指定tableView中cell的正确顺序。  
通常创建一个`NSFetchResultsController`的流程如下：

    let fetchRequest = NSFetchRequest(entityName: "Team")
    fetchRequest.sortDescriptors = [NSSortDescriptor(key: "teamName", ascending: true)]
    
    let fetchedResultController = NSFetchedResultsController(fetchRequest: fetchRequest, managedObjectContext: coreDataStack.managedContext, sectionNameKeyPath: nil, cacheName: nil)
    
    do {
      try fetchedResultController.performFetch()
    } catch let error as NSError {
      print(error.localizedDescription)
    }
    
在创建了`NSFetchResultsController`之后一定要调用其`performFetch() throws`方法，否则将不能从数据库中获取到object。  
对于tableView的dataSource，使用以下实现：

    func numberOfSectionsInTableView(tableView: UITableView) -> Int {
      return fetchedResultController.sections!.count
    }
      
    func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
      return fetchedResultController.sections![section].numberOfObjects
    }
      
    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
      let cell = tableView.dequeueReusableCellWithIdentifier("identifier", forIndexPath: indexPath)
      
      let object = fetchedResultController.objectAtIndexPath(indexPath)
      // set the cell here
                
      return cell
    }
    
### 分组
设置`NSFetchResultController`的`sectionNameKeyPath`可以对tableView进行分组，使用`name`属性设置header的title：

    func tableView(tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
      let team = fetchedResultController.sections![section]
      return team.name
    }
    
需要注意，一旦设置分组，必须把`NSFetchResultController`的`sortDescriptor`的第一个设置为与`sectionNameKeyPath`相同，否则将出现不能分到对应分组的情况。

### 监视变化
`NSFetchedResultController`可以监视其model的变化，在变化产生前后通过delegate发送消息。

# Chapter 6: Versioning and Migration
>When is a migration necessary? The easiest answer to this common question is “when you need to make changes to the data model.”

若Core Data仅仅被用作离线缓存时，则可以简单地删除所有数据并重新构建。

### 迁移数据
1. Core Data从一个data store拷贝数据到另一个data store；
2. 根据Relationship Mapping连接所有object；
3. 强制加上目标model的数据约束

如果期间出现错误，原始数据不会被删除。即仅当全部数据被成功拷贝到另一个store后原始数据才会被删除。
