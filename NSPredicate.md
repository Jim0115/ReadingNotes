# NSPredicate
NSPredicate定义了filter的逻辑约束条件。使用NSPredicate进行操作的对象必须继承自NSObject。

## 基础运算符
* 等于：`=, ==`
* 大于等于：`>=, =>`
* 小于等于：`<=, =<`
* 大于：`>`
* 小于：`<`
* 不等于：`!=, <>`
* 介于：`BETWEEN` `1 BETWEEN { 0 , 33 }`   
` [NSPredicate predicateWithFormat: @"attributeName BETWEEN %@", @[@1, @10]];`

## Boolean Value Predicate
* `TRUEPREDICATE`  
* `FALSEPREDICATE`

## 基础逻辑运算
* 且：`AND, &&`
* 或：`OR, ||`
* 非：`NOT, !`

## 字符串比较
* `BEGINWITH` 左式 begin with 右式
* `CONTAINS` 左式 包含 右式
* `ENDWITH` 左式 end with 右式
* `LIKE` 左式 相似于 右式（以`?`和`*`作为通配符，用法类似于SQL）
* `MATCHES` 左式 匹配 正则表达式  
`"self matches %@", "^(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$"`
* `[c]`表示忽略大小写 `[d]`表示忽略音调 `CONTAINS[cd]`表示忽略大小写和音调的包含

## 筛选集合类型
* `ANY, SOME` 集合中的任意一个元素满足条件
* `ALL` 集合中的所有元素都满足条件
* `NONE` 集合中没有满足条件的元素
* `IN` 左式元素是否在集合中   
`[NSPredicate predicateWithFormat: @"attribute IN %@", aCollection]`  
collection必须是NSArray，NSSet，NSDictionary或对应可变形式。
* 