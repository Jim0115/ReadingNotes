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

