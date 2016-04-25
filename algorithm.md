## Apr 24 优酷土豆
### 字符串解码 输入"a2b3q2f4" 输出"aabbbqqffff"

let c = "a2j5b3k6b3n4m6v7c1"
    
    func decode(code: String) -> String {
      var ans = ""
      
      guard code.characters.count != 0 else { return "" }
      guard code.characters.count % 2 == 0 else { return "illegal input" }
      
      var range = code.startIndex ... code.startIndex.advancedBy(1)
      
       while range.endIndex <= code.endIndex {
        let sub = code.substringWithRange(range)
        
        for _ in 0..<Int(String(sub.characters.last!))! {
          ans.append(sub.characters.first!)
        }
        
        range.startIndex = range.startIndex.advancedBy(2)
        if range.endIndex == code.endIndex {
          return ans
        } else {
          range.endIndex = range.endIndex.advancedBy(2)
        }
      }
      
      return ans
    }