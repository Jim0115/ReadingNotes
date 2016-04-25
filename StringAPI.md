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
