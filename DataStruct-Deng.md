# 数据结构－邓俊辉 e-book
### 使用栈实现进制转换
    let stack = Stack<String>()

    let digits = ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "A", "B", "C", "D", "E", "F"]
    
    var n = 2333 // 需要转换的10进制数组
    let base = 2 // 目标进制
    
    while n > 0 {
      let remainder = n % base
      stack.push(digits[remainder])
      n /= base
    }
    
基本流程为：

1. 给定一个大于0的十进制数字
2. 用给定数字除以目标进制，将余数对应的字符压入栈中，商赋给原数字
3. 判断原数字是否大于0，若是，重复第1步；否则此时栈从顶到底即为给定数字从高到低的目标进制表示形式

### 使用栈判断左右括号是否匹配
    func paren(str: String) -> Bool {
      let s = Stack<Int>()
      
      for i in str.characters {
        switch i {
        case "(", "[", "{": s.push(1)
        case ")", "]", "}":
          if s.pop() == nil { return false }
        default: break
        }
      }
      
      return s.isEmpty
    }
    
遍历字符串，遇到左括号就向栈中压入元素，遇到右括号就pop元素，若遍历完毕栈为空，则左右括号匹配。若栈不为空或某次pop时栈为空，说明括号不匹配。

### 使用栈计算逆波兰表达式
求"0 ! 1 + 2 3 ! 4 + ^ * 5 ! 67 - 8 9 + - -"的值  
将字符串分割为["0", "!", "1", "+", "2", "3", "!", "4", "+", "^", "*", "5", "!", "67", "-", "8", "9", "+", "-", "-"]的形式，遍历整个字符串数组。  
如果为数字，则将其push到栈中，如果为操作符，则将其所需的操作数pop，计算出结果后push到栈中。

    func eval(cals: [String]) -> Int {
      let stack = Stack<Int>()
      for cal in cals {
        if let num = Int(cal) {
          stack.push(num)
        } else {
          switch cal {
          case "+":
            stack.push(stack.pop()! + stack.pop()!)
          case "-":
            stack.push(-(stack.pop()! - stack.pop()!))
          case "*":
            stack.push(stack.pop()! * stack.pop()!)
          case "!":
            stack.push(foo(stack.pop()!))
          case "^":
            let pow = stack.pop()!
            let root = stack.pop()!
            var ans = 1
            for _ in 0..<pow {
              ans *= root
            }
            stack.push(ans)
          default:
            break
          }
          
        }
        print(stack)
      }
      
      return Int(stack.top!)
    }

如果表达式正确，则遍历结束后栈中只剩下唯一一个数字。