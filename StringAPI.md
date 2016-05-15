# String
## Instance Variables
### `var capitalizedString: String { get }`  
Produce a string with the first character from each word changed to the corresponding uppercase value.  
返回字符串中每个单词的首字母大写的字符串。  
### `var characters: String.CharacterView { get }`  
A collection of Characters representing the String's extended grapheme clusters.  
返回字符串对应的CharacterView  
### `var debugDescription: String { get }`
A textual representation of self, suitable for debugging.  
返回适用于debug的字符串
### `var decomposedStringWithCanonicalMapping: String { get }`
Returns a string made by normalizing the String’s contents using Form D.  
Unicode范式D标准化
### `var decomposedStringWithCompatibilityMapping: String { get }`
Returns a string made by normalizing the String’s contents using Form KD.  
Unicode范式KD标准化
### `var endIndex: Index { get }`
The "past the end" position in self.characters.  
endIndex is not a valid argument to subscript, and is always reachable from startIndex by zero or more applications of successor().  
返回字符串结束位置的Index, Index即String.CharacterView.Index。使用`string[index]`可以获得index位置的字符，但`string[string.endIndex]`将导致错误。获取最后一个字符应使用`string[string.endIndex.predecessor()]`
### `var fastestEncoding: NSStringEncoding { get }`
[Foundation] Returns the fastest encoding to which the String may be converted without loss of information.  
**暂时意义不明**
### `var hash: Int { get }`
An unsigned integer that can be used as a hash table address.  
将字符串进行的**一种**hash操作，hash表的key
### `var hashValue: Int { get }`
Axiom: x == y implies x.hashValue == y.hashValue.  
Note: The hash value is not guaranteed to be stable across different invocations of the same program. Do not persist the hash value across program runs.  
String类对Hashable协议的实现，唯一确定一个字符串
### `var isEmpty: Bool { get }`
true iff self contains no characters.  
`"".isEmpty`返回`true`  
`var s: String? = ""; s?.isEmpty`返回`Optional(true)`  
`var s: String?; s?.isEmpty`返回`nil`
### `var localizedCapitalized(Lowercase, Uppercase)String: String { get }`
[Foundation] A capitalized representation of the String that is produced using the current locale.  
类似`capitalizedString`，判断位置信息，返回每个单词首字母大写（全小写，全大写）字符串。
### `var nulTerminatedUTF8: ContiguousArray<CodeUnit> { get }`
A contiguously-stored nul-terminated UTF-8 representation of self.  
To access the underlying memory, invoke withUnsafeBufferPointer on the ContiguousArray.
`"hello".nulTerminatedUTF8`返回`[104, 101, 108, 108, 111, 0]`  
返回了每个字母的ascii码，包括最后的空字符
### `var precomposedStringWithCanonicalMapping: String { get }`
[Foundation] Returns a string made by normalizing the String’s contents using Form C.  
Unicode范式C标准化
### `var precomposedStringWithCompatibilityMapping: String { get }`
[Foundation] Returns a string made by normalizing the String’s contents using Form KC.   
Unicode范式KC标准化 
### `var smallestEncoding: NSStringEncoding { get }`
[Foundation] Returns the smallest encoding to which the String can be converted without loss of information.  
返回没有损失的最小编码
### `var startIndex: Index { get }`
The position of the first Character in self.characters if self is non-empty; identical to endIndex otherwise.  
返回字符串首字母的Index，可以直接使用`string[string.startIndex]`获得首字母(Character)，当字符串为空时，`startIndex`与`endIndex`相同。
### `var stringByRemovingPercentEncoding: String?`
[Foundation] Returns a new string made from the String by replacing all percent encoded sequences with the matching UTF-8 characters.  
将URL编码转化为UTF-8编码，不符合编码规则将返回nil
### `var unicodeScalars: String.UnicodeScalarView { get set }`
The value of self as a collection of Unicode scalar values.  
返回该字符串的`UnicodeScalarView`，可修改。遍历`UnicodeScalarView`可以获得字符串中每个字符的`UnicodeScalar`形式

    // 汉字的转换
    extension String {
      mutating func removeCharactersInString(s: String) -> String {
        for ch in s.characters {
          for c in self.characters {
            if ch == c {
              self.removeAtIndex(self.characters.indexOf(c)!)
            }
          }
        }
        return self
      }
    }
    
    let name = "王仕杰"
    
    var uni = ""
    for i in name.unicodeScalars {
      uni += i.escape(asASCII: true)
    }
    
    let array = uni.removeCharactersInString("{}\\").componentsSeparatedByString("u").filter { !$0.isEmpty } // ["738B", "4ED5", "6770"]
    
    func foo(a: [String]) -> String {
      var ans = ""
      
      for bar in array {
        ans += String(UnicodeScalar(Int(bar, radix: 16)!))
      }
      
      return ans
    }
    
    foo(array) // "王仕杰"
    
### `var utf8（16）: String.UTF8View { get }`
A UTF-8(16) encoding of self.

## Subscripts
### `subscript(i: Index) -> Character { get }`
Access the Character at position.  
Requires: position is a valid position in self.characters and position != endIndex.
使用index作为下标可以从字符串中获得字符
### `subscript(subRange: Range<Index>) -> String { get }`
Access the characters in the given subRange.  
Complexity: O(1) unless bridging from Objective-C requires an O(N) conversion.
使用Range结构体作为下标可以从字符串中取出子字符串

## Class Methods
### `static func availableStringEncodings() -> [NSStringEncoding]`
[Foundation] Returns an Array of the encodings string objects support in the application’s environment.  
返回所有支持的编码
### `static func defaultCStringEncoding() -> NSStringEncoding`
[Foundation] Returns the C-string encoding assumed for any method accepting a C string as an argument.  
返回默认的C语音字符串编码
### `static func fromCString(cs: UnsafePointer<CChar>) -> String?`
Creates a new String by copying the nul-terminated UTF-8 data referenced by a CString. 
Returns nil if the CString is NULL or if it contains ill-formed UTF-8 code unit sequences.  
将C语言字符串转换为String
### `static func localizedStringWithFormat(format: String, _ arguments: CVarArgType...) -> String`
[Foundation] Returns a string created by using a given format string as a template into which the remaining argument values are substituted according to the user's default locale.
`String.localizedStringWithFormat("%@ %ld %lu", "hello", 3, 20)  // hello 3 20`

## Instance Methods
### `mutating func append(c: Character)`
将一个字符追加到字符串末尾，字符串必须为可变的
### `mutating func append(x: UnicodeScalar)`
将一个Unicode字符追加到字符串末尾，字符串必须为可变的
### `mutating func appendContentsOf(other: String)`
Append the elements of other to self.  
将`other`字符串追加到字符串的末尾
### `mutating func appendContentsOf<S : SequenceType where S.Generator.Element == Character>(newElements: S)`
泛形的追加方法，对任意元素为Character的SequenceType成立
### `func cStringUsingEncoding(encoding: NSStringEncoding) -> [CChar]?`
[Foundation] Returns a representation of the String as a C string using a given encoding.  
`"hello".cStringUsingEncoding(NSUTF8StringEncoding) // [104, 101, 108, 108, 111, 0]`  
`"hello\0".cStringUsingEncoding(NSUTF8StringEncoding) // [104, 101, 108, 108, 111, 0]`    
对于末尾没有`\0`的字符串，在转换为C语言类型字符串时会自动添加`\0`，但对已有的字符串不会重复添加。  
使用`String.fromCString([CChar])`可以转换回String
### `func canBeConvertedToEncoding(encoding: NSStringEncoding) -> Bool`
[Foundation] Returns a Boolean value that indicates whether the String can be converted to a given encoding without loss of information.  
判断字符串是否能无损转换为指定的编码
### `func capitalizedStringWithLocale(locale: NSLocale?) -> String`
[Foundation] Returns a capitalized representation of the String using the specified locale.  
根据输入的位置输出大小写转换后的String
### `func caseInsensitiveCompare(aString: String) -> NSComparisonResult`
[Foundation] Returns the result of invoking `compare:options:` with `NSCaseInsensitiveSearch` as the only option.  
大小写不敏感的比较方法，返回枚举类型`NSComparisonResult`
  
     enum NSComparisonResult : Int {
        case OrderedAscending
        case OrderedSame
        case OrderedDescending
     }
### `func commonPrefixWithString(aString: String, options: NSStringCompareOptions) -> String`
[Foundation] Returns a string containing characters the String and a given string have in common, starting from the beginning of each up to the first characters that aren’t equivalent.  
返回两个字符串从头开始相同的部分。若无相同部分则返回`""`
### `func compare(aString: String, options mask: NSStringCompareOptions = default, range: Range<Index>? = default, locale: NSLocale? = default) -> NSComparisonResult`
[Foundation] Compares the string using the specified options and returns the lexical ordering for the range.  
字符串比较的万能方法
### `func completePathIntoString(outputName: UnsafeMutablePointer<String> = default, caseSensitive: Bool, matchesIntoArray: UnsafeMutablePointer<[String]> = default, filterTypes: [String]? = default) -> Int`
[Foundation] Interprets the String as a path in the file system and attempts to perform filename completion, returning a numeric value that indicates whether a match was possible, and by reference the longest path that matches the String. Returns the actual number of matching paths.  
补全路径 **意义不明**
### `func componentsSeparatedByCharactersInSet(separator: NSCharacterSet) -> [String]`
[Foundation] Returns an array containing substrings from the String that have been divided by characters in a given set.  
使用一个字符集中所有字符去分割字符串，字符集通常使用`NSCharacterSet`中的static属性。
### `func componentsSeparatedByString(separator: String) -> [String]`
[Foundation] Returns an array containing substrings from the String that have been divided by a given separator.  
使用参数字符串作为**分割标准**去分割原字符串，返回经过分割后的字符串数组。  
`"aloha".componentsSeparatedByString("lo") // ["a", "ha"]`
### `func containsString(other: String) -> Bool`
[Foundation] Returns true iff other is non-empty and contained within self by case-sensitive, non-literal search.  
Equivalent to self.rangeOfString(other) != nil  
判断是非包含参数字符串。  
`"believe".containsString("lie") // true`
### `func dataUsingEncoding(encoding: NSStringEncoding, allowLossyConversion: Bool = default) -> NSData?`
[Foundation] Returns an NSData object containing a representation of the String encoded using a given encoding.  
使用指定的编码将字符串对象转换为NSdata，若字符串不符合编码要求将返回nil。  
`"东北大学".dataUsingEncoding(NSUTF8StringEncoding)  // <e4b89ce5 8c97e5a4 a7e5ada6>`
`"NEU".dataUsingEncoding(NSASCIIStringEncoding) // <4e4555>`  
`"东北大学".dataUsingEncoding(NSASCIIStringEncoding) // nil`
### `func enumerateLines(body: (line: String, inout stop: Bool) -> ())`
[Foundation] Enumerates all the lines in a string.
`line`为每一行的字符串，`stop`为控制条件，在closure中将`stop`设为true将终止遍历。使用`\r`, `\n`, `\r\n`均被视为一次换行。
### `func enumerateLinguisticTagsInRange(range: Range<Index>, scheme tagScheme: String, options opts: NSLinguisticTaggerOptions, orthography: NSOrthography?, _ body: (String, Range<Index>, Range<Index>, inout Bool) -> ())`
[Foundation] Performs linguistic analysis on the specified string by enumerating the specific range of the string, providing the Block with the located tags.  
**意义不明**
### `func enumerateSubstringsInRange(range: Range<Index>, options opts: NSStringEnumerationOptions, _ body: (substring: String?, substringRange: Range<Index>, enclosingRange: Range<Index>, inout Bool) -> ())`
[Foundation] Enumerates the substrings of the specified type in the specified range of the string.  
根据`NSStringEnumerationOptions`中的条件在`range`中遍历字符串，在closure中处理。

    struct NSStringEnumerationOptions : OptionSetType {
        init(rawValue rawValue: UInt)
        static var ByLines: NSStringEnumerationOptions { get }
        static var ByParagraphs: NSStringEnumerationOptions { get }
        static var ByComposedCharacterSequences: NSStringEnumerationOptions { get }
        static var ByWords: NSStringEnumerationOptions { get }
        static var BySentences: NSStringEnumerationOptions { get }
        static var Reverse: NSStringEnumerationOptions { get }
        static var SubstringNotRequired: NSStringEnumerationOptions { get }
        static var Localized: NSStringEnumerationOptions { get }
    }
    
### `func getBytes(inout buffer: [UInt8], maxLength maxBufferCount: Int, usedLength usedBufferCount: UnsafeMutablePointer<Int>, encoding: NSStringEncoding, options: NSStringEncodingConversionOptions, range: Range<Index>, remainingRange leftover: UnsafeMutablePointer<Range<Index>>) -> Bool`
[Foundation] Writes the given range of characters into buffer in a given encoding, without any allocations. Does not NULL-terminate.

buffer: A buffer into which to store the bytes from the receiver. The returned bytes are not NUL-terminated.

maxBufferCount: The maximum number of bytes to write to buffer.

usedBufferCount: The number of bytes used from buffer. Pass nil if you do not need this value.

encoding: The encoding to use for the returned bytes.

options: A mask to specify options to use for converting the receiver’s contents to encoding (if conversion

is necessary).

range: The range of characters in the receiver to get.

leftover: The remaining range. Pass nil If you do not need this value.

Returns: true iff some characters were converted.

Note: Conversion stops when the buffer fills or when the conversion isn't possible due to the chosen encoding.

Note: will get a maximum of min(buffer.count, maxLength) bytes.  
将字符串信息写入buffer中

### `func getCString(inout buffer: [CChar], maxLength: Int, encoding: NSStringEncoding) -> Bool`
[Foundation] Converts the String’s content to a given encoding and stores them in a buffer.  
Note: will store a maximum of min(buffer.count, maxLength) bytes.  
将字符串写入指定的buffer中  
**在Swift的函数命名规则中，get开头的方法表示将通过修改某个传入的参数获取值**
### `func hasPrefix(prefix: String) -> Bool`
Returns true iff self begins with prefix.  
判断一个字符串是否有指定前缀。 `"hello".hasPrefix("he") // true`

### `func hasSuffix(prefix: String) -> Bool`
Returns true iff self begins with suffix.  
判断一个字符串是否有指定后缀。 `"hello".hasPrefix("lo") // true`
### `mutating func insert(newElement: Character, atIndex i: Index)`
Insert newElement at index i.  
Invalidates all indices with respect to self.  
在指定位置插入字符
### `mutating func insertContentsOf<S : CollectionType where S.Generator.Element == Character>(newElements: S, at i: Index)`
Insert newElements at index i.  
Invalidates all indices with respect to self.  
在指定位置插入元素为Character的集合类型

    var name = "University"
    name.insertContentsOf("Peking ".characters, at: name.startIndex)  // Peking University
    
### `func lengthOfBytesUsingEncoding(encoding: NSStringEncoding) -> Int`
[Foundation] Returns the number of bytes required to store the String in a given encoding.  
返回字符串在指定编码下的字节长度。对于无法转换成指定编码的字符串返回0；
    
    "东北大学".lengthOfBytesUsingEncoding(NSASCIIStringEncoding) // 0
    "东北大学".lengthOfBytesUsingEncoding(NSUTF32StringEncoding) // 32
    "东北大学".lengthOfBytesUsingEncoding(NSUTF8StringEncoding) // 8
    "东北大学".lengthOfBytesUsingEncoding(NSUnicodeStringEncoding) // 8
    
### `func lineRangeForRange(aRange: Range<Index>) -> Range<Index>`
[Foundation] Returns the range of characters representing the line or lines containing a given range.
返回输入range中字符串所在行的range
### `func linguisticTagsInRange(range: Range<Index>, scheme tagScheme: String, options opts: NSLinguisticTaggerOptions = default, orthography: NSOrthography? = default, tokenRanges: UnsafeMutablePointer<[Range<Index>]> = default) -> [String]`
[Foundation] Returns an array of linguistic tags for the specified range and requested tags within the receiving string.

### `func localizedCaseInsensitiveCompare(aString: String) -> NSComparisonResult`
[Foundation] Compares the string and a given string using a case-insensitive, localized, comparison.

### `func propertyList() -> AnyObject`
[Foundation] Parses the String as a text representation of a property list, returning an NSString, NSData, NSArray, or NSDictionary object, according to the topmost element.
### `mutating func removeAll(keepCapacity keepCapacity: Bool = default)`
Remove all characters.

Invalidates all indices with respect to self.

keepCapacity: If true, prevents the release of allocated storage, which can be a useful optimization when self is going to be grown again.  
清空字符串。

### `mutating func removeAtIndex(i: Index) -> Character`
Remove and return the element at index i.  
Invalidates all indices with respect to self.  
移除指定index位置的字符并返回。

### `mutating func removeRange(subRange: Range<Index>)`
Remove the indicated subRange of characters.  
Invalidates all indices with respect to self.  
移除指定Range内的字符串

### `mutating func replaceRange(subRange: Range<Index>, with newElements: String)`
Replace the given subRange of elements with newElements.  
Invalidates all indices with respect to self.  
将指定range内的字符串替换为新字符串

### `mutating func reserveCapacity(n: Int)`
pre-allocate storage for the given capacity  
为字符串预先分配内存空间

### `func stringByAddingPercentEncodingWithAllowedCharacters(allowedCharacters: NSCharacterSet) -> String?`
[Foundation] Returns a new string made from the String by replacing all characters not in the specified set with percent encoded characters.  
根据允许的字符集将string中不符合的字符进行URL编码

### `func stringByAppendingFormat(format: String, _ arguments: CVarArgType...) -> String`
[Foundation] Returns a string made by appending to the String a string constructed from a given format string and the following arguments.  
类似OC中方法，在Swift中较少使用，有多种替代方法。

### `func stringByAppendingString(aString: String) -> String`
[Foundation] Returns a new string made by appending a given string to the String.  
等价于 `self + aString`

### `func stringByApplyingTransform(transform: String, reverse: Bool) -> String?`
    let NSStringTransformLatinToKatakana: String // 字母转片假名
    let NSStringTransformLatinToHiragana: String // 字母转平假名
    let NSStringTransformLatinToHangul: String // 字母转韩语
    let NSStringTransformLatinToArabic: String // 字母转阿拉伯语
    let NSStringTransformLatinToHebrew: String // 字母转希伯来语
    let NSStringTransformLatinToThai: String // 字母转泰语
    let NSStringTransformLatinToCyrillic: String // 字母转西里尔字母（俄语）
    let NSStringTransformToLatin: String // 转换为字母（带音调）
    let NSStringTransformMandarinToLatin: String // 汉语转字母（带音调）不能reverse
    let NSStringTransformHiraganaToKatakana: String // 平假名转片假名
    let NSStringTransformFullwidthToHalfwidth: String // 全宽转半宽
    let NSStringTransformToXMLHex: String 
    let NSStringTransformToUnicodeName: String // 转换为Unicode名
    let NSStringTransformStripCombiningMarks: String // 移除combining mark
    let NSStringTransformStripDiacritics: String // 移除变音符
对字符串进行上述操作返回操作后的字符串，可能为空。reverse表示进行transform的逆操作，部分逆操作始终返回nil。

一些常用的其他方式：

    "HELLO WORLD".stringByApplyingTransform("Lower", reverse: false) // hello world
    
    "HELLO WORLD".stringByApplyingTransform("[AEIOU] Lower", reverse: false) // HeLLo WoRLD
    
    "上海".stringByApplyingTransform("Any-Latin; Latin-ASCII; Lower", reverse: false) // shang hai
    
    "\"Make it so,\" said Picard.".stringByApplyingTransform("[:Punctuation:] remove", reverse: false) // Make it so said Picard
    
    "5 plus 6 equals 11 👍!".stringByApplyingTransform("[:^Letter:] remove", reverse: false) // plusequals
    
    "\"How's it going?\"".stringByApplyingTransform("Publishing", reverse: false) // “How’s it going?”
    
    
### `func stringByFoldingWithOptions(options: NSStringCompareOptions, locale: NSLocale?) -> String`
[Foundation] Returns a string with the given character folding options applied.

### `func stringByPaddingToLength(newLength: Int, withString padString: String, startingAtIndex padIndex: Int) -> String`
[Foundation] Returns a new string formed from the String by either removing characters from the end, or by appending as many occurrences as necessary of a given pad string.  
使用`padString`将字符串填充到`newLength`长度，从`padIndex`位置开始填充。  
`"123".stringByPaddingToLength(20, withString: "hello", startingAtIndex: 2) // 123llohellohellohell`

### `func stringByReplacingCharactersInRange(range: Range<Index>, withString replacement: String) -> String`
[Foundation] Returns a new string in which the characters in a specified range of the String are replaced by a given string.  
使用`replacement`替换`range`内的字符串。

### `func stringByReplacingOccurrencesOfString(target: String, withString replacement: String, options: NSStringCompareOptions = default, range searchRange: Range<Index>? = default) -> String`
[Foundation] Returns a new string in which all occurrences of a target string in a specified range of the String are replaced by another given string.  
在`range`内使用`replacement`替换`target`。

### `func stringByTrimmingCharactersInSet(set: NSCharacterSet) -> String`
[Foundation] Returns a new string made by removing from both ends of the String characters contained in a given character set.  
去掉字符串两端的字符集中的字符

### `func substringFromIndex(index: Index) -> String`
从头到当前index的子字符串。

### `func substringToIndex(index: Index) -> String`
从index到字符串末尾的子字符串，不包括index所在位置字符。

### `func substringWithRange(aRange: Range<Index>) -> String`
获得`range`内的子字符串

### `mutating func write(other: String)`
Append other to this stream.  
实现于`OutputStreamType`协议

### `func writeTo<Target : OutputStreamType>(inout target: Target)`
Write a textual representation of self into target.
将`self`写入到实现了`OutputStreamType`的`target`中。