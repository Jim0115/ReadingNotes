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
