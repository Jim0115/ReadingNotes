## Apr 24 优酷土豆
### 字符串解码 输入"a2b3q2f4" 输出"aabbbqqffff"

    let c = "a2j5b3k6b3n4m6v7c1"
    
    func decode(code: String) -> String {
      var ans = ""
      
      guard !code.isEmpty else { return "" }
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
    
## April 28, 2016 leetcode
### 二叉树的深度

    /**
     * Definition for a binary tree node.
     * public class TreeNode {
     *     public var val: Int
     *     public var left: TreeNode?
     *     public var right: TreeNode?
     *     public init(_ val: Int) {
     *         self.val = val
     *         self.left = nil
     *         self.right = nil
     *     }
     * }
     */
     
    class Solution {
      func maxDepth(root: TreeNode?) -> Int {
        if root == nil {
          return 0
        } else {
          return 1 + max(maxDepth(root!.left), maxDepth(root!.right))
        }
      }
    }
    
判断当前节点是非存在，不存在则返回0，存在则返回1 + 左右子树中深度更大的

### 翻转二叉树，即交换二叉树的左右子树

    func invertTree(root: TreeNode?) -> TreeNode? {
      let tmp = root?.left
      root?.left = root?.right
      root?.right = tmp
      if root?.left != nil {
        invertTree(root?.left)
      }
      if root?.right != nil {
        invertTree(root?.right)
      }
      return root
    }
    
    // 标准答案
    public TreeNode invertTree(TreeNode root) {
        if (root == null) {
            return null;
        }
        TreeNode right = invertTree(root.right);
        TreeNode left = invertTree(root.left);
        root.left = right;
        root.right = left;
        return root;
    }
    
    // 修改后的答案
    func invertTree(root: TreeNode?) -> TreeNode? {
      if root == nil { return nil }
      
      let tmp = root?.left
      root?.left = root?.right
      root?.right = tmp
      
      invertTree(root?.left)
      invertTree(root?.right)
      
      return root
    }
    
对于空节点返回空，否则交换其左右子树，递归操作其左右子树。

### move zeros 将一个数组中所有的0移动到数组末尾
    func moveZeroes(inout nums: [Int]) {
      var i = 0
      while nums[i] != 0 {
        if i == nums.count - 1 {
          return
        }
        i += 1
      }
      var j = i
      while j < nums.count {
        if nums[j] != 0 {
          nums[i] = nums[j]
          nums[j] = 0
          i += 1
        }
        j += 1
      }
    }
使用两个指针，指针1指向数组头部第一个0，指针2指向指针1后第一个不为0的元素。之后，在数组范围内指针2++，遇到不为0的元素就交换两个指针的元素，之后指针1++。
## April 29, 2016 leetcode
### 判断两个二叉树是否相同
    func isSameTree(p: TreeNode?, _ q: TreeNode?) -> Bool {
      if (p == nil && q == nil) { return true }
      if p?.val != q?.val {
        return false
      } else {
        return isSameTree(p?.left, q?.left) && isSameTree(p?.right, q?.right)
      }
    }
    
首先判断两个节点是否相同：此时同为空或值相同则相同，不同为空则不同，同不为空但值不同则不同。之后递归的对节点的左右子树进行对比，节点相同且左右子树都相同则两个树相同。

### 判断一个字符串是否是另一字符串乱序重拍后的结果

    func isAnagram(s: String, _ t: String) -> Bool {
      if s.characters.count != t.characters.count { return false }
      for c in s.characters {
        if !t.characters.contains(c) { return false }
      }
      return true
    }
    // 错误答案
只判断了一个字符串中的所有元素是否全部包含在另一字符串中。如"aab"和"abb"就会出现错误结果。

    func isAnagram(s: String, _ t: String) -> Bool {
      if s.characters.count != t.characters.count { return false }
      return s.characters.sort() == t.characters.sort()
    }
首先判断两字符串长度，不等则返回false。之后对两字符串进行排序，判断是否相等。  
本题假设字符串只包含26个字母，则可以对所有字符进行hash，计算出每个字符出现的次数后对比。

### Excel 列字母转换为数字
    A -> 1
    B -> 2
    C -> 3
    ...
    Z -> 26
    AA -> 27
    AB -> 28 
    
类似于进制转换，将26进制转换为10进制。

    func titleToNumber(s: String) -> Int {
      let str = s.uppercaseString
      var ans = 0
      var times: Double = 0
      
      for char in str.unicodeScalars.reverse() {
        ans += (Int(char.value) - 64) * Int(pow(Double(26), times))
        times += 1
      }
      
      return ans
    }

将字符串转换为字符数组并倒序，遍历该字符数组，从最低位开始算起，每高一位代表26^n