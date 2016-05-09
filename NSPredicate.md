# NSPredicate
NSPredicate定义了filter的逻辑约束条件。

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


