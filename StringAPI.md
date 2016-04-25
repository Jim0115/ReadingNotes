# String
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
返回字符串结束位置的Index, Index即String.CharacterView.Index。使用`string[index]`可以获得index位置的字符，但`string[string.endIndex]`将导致错误。获取最后一个字符使用`string[string.endIndex.predecessor()]`