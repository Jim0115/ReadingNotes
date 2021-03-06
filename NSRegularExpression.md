# NSRegularExpression
### NSRegularExpression 不是 NSExpression 的子类
### 构造方法
`init(pattern pattern: String, options options: NSRegularExpressionOptions) throws`

#### NSRegularExpressionOptions
    struct NSRegularExpressionOptions : OptionSetType {
        init(rawValue rawValue: UInt)
        static var CaseInsensitive: NSRegularExpressionOptions { get }
        static var AllowCommentsAndWhitespace: NSRegularExpressionOptions { get }
        static var IgnoreMetacharacters: NSRegularExpressionOptions { get }
        static var DotMatchesLineSeparators: NSRegularExpressionOptions { get }
        static var AnchorsMatchLines: NSRegularExpressionOptions { get }
        static var UseUnixLineSeparators: NSRegularExpressionOptions { get }
        static var UseUnicodeWordBoundaries: NSRegularExpressionOptions { get }
    }
    
* CaseInsensitive：忽略大小写
* AllowCommentsAndWhitespace：忽略空格以及#开头的注释
* IgnoreMetacharacters：将整个pattern视作一个字符串
* DotMatchesLineSeparators：允许`.`匹配换行符
* AnchorsMatchLines：允许`^`和`$`匹配行的开始和结束位置
* UseUnixLineSeparators：只把`\n`视作换行符（默认匹配所有类型换行符）
* UseUnicodeWordBoundaries: Use Unicode TR#29 to specify word boundaries
### 保留字
* [
* ()
* \
* *
* +
* ?
* {}
* ^
* $
* .
* |
* /

使用以上字符时需要用反斜线进行转义

### 语法
`3 (am|pm)`会匹配"3 am"或"3 pm"，但不会匹配"3"。`(A|B|C)`表示匹配A，B，C中的任意一个  
`Nov(ember)?`会匹配"Nov"和"November"，问号表示括号内的内容为可选的  
字符组表示匹配一组字符中的某个字符，包含在`[]`中，如：`t[abc]`会匹配"ta","tb","tc"，但不会匹配"tab","tac"等  
如果字符连续出现，可以使用范围定义一个字符组，如：`10[0-9]`匹配"100" ~ "109"。字符组不限于数字，可以使用`p[a-d]`匹配"pa", "pb", "pc"和"pd"  
使用`^`符号表示排除在外的字符，如`t[^a-c]`会匹配"ta", "tb"和"tc"外其他以"t"开头的字符串  

---

* `.`匹配任一字符，`a.b`匹配"aab", "axb", "a@b", "a!b"等字符串
* `\w`匹配"word-like"**字符**，包括数字、字母、下划线，但不匹配其他标点符号和字符串。`hello\w`会匹配"helloa", "hello1", "hello_"，不会匹配"hello!", "hello~"等
* `\d`匹配数字，通常等价于`[0-9]`。使用`\d\d?:\d\d`会匹配时间字符串，如"5:30", "18:25"等
* `\b`匹配额外的字符，如空格，标点符号等等。`to\b`会匹配"to the moon", "to!", "to "等，但不会匹配"tomorrow"，通常用于对单词进行匹配
* `\s`匹配空白字符，如空格，tab，换行符等。`hello\s`匹配"hello "
* `^`用于一行的开始。`^hello`会匹配"hello there"，而不会匹配"say hello to sb"
* `$`用于一行的结束。`the end$`会匹配"it was the end"，而不会匹配"the end is coming"
* `*`匹配它之前的元素0次或多次。`12*3`会匹配"13", "123", "1223", "122222223"等
* `+`匹配它之前的元素1次或多次。`12+3`会匹配"123", "1223", "1222223"
* `{}`表示匹配之前元素的最小个数和最大个数。如`12{1, 3}3`会匹配"123", "1223", "12223"，而不会匹配"13", "122233"。`{1,}`表示没有最大限制。

### Quiz
* 1到10个字符长度的标准英文字母 `^\w{1,10}$`
* 标准英文字母加上`'`符号 `^[\w']{1,10}$`
* 出生日期，以下格式之一：dd/mm/yyyy, dd-mm-yyyy, dd.mm.yyyy，且在01/01/1900和31/12/2099之间 `^(0[1-9]|1[012])[\\/\\.-](0[1-9]|[12]\\d|3[01])[\\/\\.-](19|20)\\d{2}$`
* 电子邮件：[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\\.[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*@(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?

### 匹配
使用`func matchesInString(_ string: String, options options: NSMatchingOptions, range range: NSRange) -> [NSTextCheckingResult]`对字符串进行匹配，返回匹配到的结果。

### 替换
对于每个捕获组捕获到的内容可以单独取出进行替换。

    let str = "Jan 15, 1995"
    
    let regex = "^(\\w{3})\\s(\\d{2}),\\s(\\d{4})"
    let replacePattern = "$0 $3 and $1 and $2"
    
    do {
      let expression = try NSRegularExpression(pattern: regex, options: .CaseInsensitive)
      let replacement = expression.stringByReplacingMatchesInString(str, options: [], range: NSRange(location: 0, length: str.characters.count), withTemplate: replacePattern)
      print(replacement) // Jan 15, 1995 1995 and Jan and 15
    } catch let error as NSError {
      print(error.localizedDescription)
    }


`$0`代表原字符串，从`$1`开始代表对应的捕获组捕获到的字符串，可以根据需要进行选取。  
使用`func replaceMatchesInString(_ string: NSMutableString, options options: NSMatchingOptions, range range: NSRange, withTemplate templ: String) -> Int`可以直接对原字符串进行替换。