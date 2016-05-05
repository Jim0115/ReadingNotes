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

## April 30, 2016 leetcode
### 判断数组中是否存在重复数字。
    func containsDuplicate(nums: [Int]) -> Bool {
      return Set(nums).count != nums.count
    }
将数组元素放入集中，判断集中元素数量和数组中元素数量是否相同。

    func containsDuplicate(nums: [Int]) -> Bool {
      if nums.count == 0 { return false }
      let sortedNums = nums.sort()
      for i in 0..<sortedNums.count - 1 {
        if sortedNums[i] == sortedNums[i + 1] {
          return true
        }
      }
      return false
    }
基于排序的解法，对数组进行排序，判断是否有相等的相邻元素。

### 找到一个数组中存在的数量超过一半的元素
    func majorityElement(nums: [Int]) -> Int {
      let sortedNums = nums.sort()
      return sortedNums[sortedNums.count / 2]
    }
排序数组，从中找出中位数即为数量超过一半的元素

### 去除一个有序链表中重复的元素
    func deleteDuplicates(head: ListNode?) -> ListNode? {
      var curr = head
      while curr != nil {
        if curr?.val == curr?.next?.val {
          curr?.next = curr?.next?.next
          curr = curr?.next
        } else {
          curr = curr?.next
        }
      }
      return head
    }
从链表头节点开始，判断当前节点值是否等于下一节点值：是，则当前节点的下一节点换为原下一节点的下一节点，继续判断；否，当前节点等于当前节点下一节点，直到当前节点为空。

### 判断一个数是否为3的n次方
    func isPowerOfThree(n: Int) -> Bool {
      var num = 1
      while num <= n {
        if num == n { return true }
        num *= 3
      }
      return false
    }
从num = 1开始进行计算，若num > n 返回false，否则判断num == n，若是，返回true，否则num *= 3。
### ugly number
如果一个数的质因数只有2, 3, 5，则这个数为ugly number。1被视为ugly number。

    func isUgly(num: Int) -> Bool {
      if num == 0 { return false }
      var n = num
      while (n % 2 == 0) {
        n = n / 2
      }
      while (n % 3 == 0) {
        n = n / 3
      }
      while (n % 5 == 0) {
        n = n / 5
      }
      return n == 1
    }
判断一个数的因数中是否有2,3,5，若是，则将其除以2,3,5，判断最后输出的结果是否为1。

## May 1, 2016 leetcode
### Happy Number
将一个数所有位上的数字进行平方求和，若最终收敛为1，则该数字为一个Happy Number。否则将在几个数字间循环。

    func isHappy(n: Int) -> Bool {
      var ans = sum(n)
      var nums = Set<Int>()
      while true {
        if ans == 1 {
          return true
        } else if !nums.contains(ans) {
          nums.insert(ans)
          ans = sum(ans)
        } else {
          return false
        }
        
      }
      return false
    }
    
    func sum(n: Int) -> Int {
      return n.description.characters.reduce(0) {
        $0 + Int(pow(Double(String($1))!, 2.0))
      }
    }

创建一个集合用于存储所有已经出现过的数字，每次操作后进行判断，若结果为1，则返回true。结果不为1，判断集合中是否存在该数字，若存在，则产生循环，返回false；不存在，则将数字存入集合中，继续进行操作。

### Best Time to Buy and Sell Stock
给定数组表示第i天的股票价格，若只允许一次买入和卖出，计算所获得的最大收益。（卖出必须在买入之后，所以不能单纯的用数组中最大值减去最小值）  
使用动态规划方法求解。  

    func maxProfit(prices: [Int]) -> Int {
      if prices.isEmpty { return 0 }
      var maxProfit = 0
      var currentMin = prices[0]
      for index in 0..<prices.count {
        currentMin = min(currentMin, prices[index])
        maxProfit = max(maxProfit, prices[index] - currentMin)
      }
      return maxProfit
    }
    
两个变量分别记录当前的最低价格和最大收益。遍历数组，判断当天股价是否小于最低价，若是，则赋值给最低价。判断当前天数卖出是否能获得最大收入，若是，更新最大收入。最终的最大收入即为计算结果。

### 合并两个有序链表
    func mergeTwoLists(l1: ListNode?, _ l2: ListNode?) -> ListNode? {
      if l1 == nil { return l2 }
      if l2 == nil { return l1 }
      
      var n1 = l1
      var n2 = l2
      
      let h = ListNode(0)
      var head = h
      
      while (n1 != nil || n2 != nil) {
        let compare = (n1?.val ?? Int.max) < (n2?.val ?? Int.max)
        head.next = compare ? n1 : n2
        
        head = head.next!
        if compare {
          n1 = n1?.next
        } else {
          n2 = n2?.next
        }
      }
      
      return h.next
    }
**可选值进行比较时，nil被视为最小的值**  
循环解法：某个队列为空，就返回另一个队列。对两个链表的头节点进行比较，较小的视为下一节点。循环直到两链表遍历完毕。

    func mergeTwoLists(l1: ListNode?, _ l2: ListNode?) -> ListNode? {
      if l1 == nil { return l2 }
      if l2 == nil { return l1 }
      
      var ret: ListNode? = nil
      
      if (l1?.val < l2?.val)
      {
        ret = l1;
        ret?.next = mergeTwoLists(l1?.next, l2);
      }
      else
      {
        ret = l2;
        ret?.next = mergeTwoLists(l1, l2?.next);
      }
      
      return ret;
    }
递归解法：某个队列为空，就返回另一个队列。比较两个头节点的值，较小的视为下一节点。对剩余的两个队列继续调用合并方法。

## May 2, 2016 leetcode
### 成对交换链表中节点。
    func swapPairs(head: ListNode?) -> ListNode? {
      if head?.next == nil { return head }
      var n1 = head
      var n2 = n1?.next
      let h = n2
      
      while n2 != nil {
        n1?.next = n2?.next
        n2?.next = n1
        
        let tmp = n1
        n1 = n1?.next
        n2 = n1?.next
        if n2 != nil {
          tmp?.next = n2
        }
      }
      return h
    }
使用两个指针进行指向头节点和头节点的下一节点，调整next指针指向的节点，需要注意与后面一对节点之间的指向关系。
### 对一个字符串中的元音字母倒序排列
    func isVowel(c: Character) -> Bool {
      return "aeiouAEIOU".characters.contains(c)
    }
    
    func toCharArray(s: String) -> [Character] {
      var v = [Character]()
      for c in s.characters {
        v.append(c)
      }
      return v
    }
    
    func toString(chars: [Character]) -> String {
      var s = ""
      for c in chars {
        s.append(c)
      }
      return s
    }
    
    func reverseVowels(s: String) -> String {
      if s.isEmpty { return s }
      var chars = toCharArray(s)
      
      var start = 0
      var end = chars.count - 1
      
      while true {
        while !isVowel(chars[start]) && start < end {
          start += 1
        }
        while !isVowel(chars[end]) && end > start {
          end -= 1
        }
        
        if start >= end {
          return toString(chars)
        }
        
        let tmp = chars[start]
        chars[start] = chars[end]
        chars[end] = tmp
        
        start += 1
        end -= 1
      }
    }
    
由于string.characterView不能修改，于是将字符串转化成字符数组进行操作。从首尾两个方向遍历字符数组，找到元音字母后进行交换。有几种特殊输入情况：空字符串，不包含元音字母的字符串，大小写元音字母，需要注意。

## May 3, 2016 leetcode
### 判断平衡二叉树
    func depth(root: TreeNode?) -> Int {
      if root == nil {
        return 0
      } else {
        return 1 + max(depth(root?.left), depth(root?.right))
      }
    }
    
    func isBalancedNode(node: TreeNode?) -> Bool {
      return abs(depth(node?.left) - depth(node?.right)) <= 1
    }
    
    func isBalanced(root: TreeNode?) -> Bool {
      if root == nil { return true }
      return isBalancedNode(root) && isBalanced(root?.left) && isBalanced(root?.right)
    }
    
判断一个二叉树是平衡二叉树，从根节点起，计算每个节点左右子树深度之差，判断是否大于1，若是，返回false，否则判断左右子树是否为平衡二叉树。

### 二叉树的一种广度遍历
将二叉树：

        3
       / \
      9  20
        /  \
       15   7
转化为如下形式：

    [
      [15,7],
      [9,20],
      [3]
    ]
代码：

    func levelOrderBottom(root: TreeNode?) -> [[Int]] {
      if root == nil { return [] }
      
      var ans = [[Int]]()
      
      var children = [TreeNode?](arrayLiteral: root)
      var noNil: [TreeNode] {
        return children.flatMap { $0 }
      }
      
      while noNil.count > 0 {
        var tmp = [Int]()
        var tmpNode = [TreeNode?]()
        for v in noNil {
          tmp.append(v.val)
          tmpNode.append(v.left)
          tmpNode.append(v.right)
        }
        children = tmpNode
        ans.append(tmp)
      }
      return ans.reverse()
    }
本质上是二叉树的广度遍历，将每层结果保存在一个数组中，整个树构成一个二维数组。

## May 4, 2016 leetcode
### Plus One
input: [9, 9, 9]   
output: [1, 0, 0, 0]

    func plusOne(digits: [Int]) -> [Int] {
      if digits.count == 0 { return [] }
      var ans = digits
      var index = ans.endIndex - 1
      if index == 0 && ans[index] == 9 {
        ans[index] = 0
        ans.insert(1, atIndex: 0)
        return ans
      }
      while addOne(&ans[index]) {
        index -= 1
        if index == 0 && ans[index] == 9 {
          ans[index] = 0
          ans.insert(1, atIndex: 0)
          return ans
        }
      }
      return ans
    }
    
    func addOne(inout num: Int) -> Bool {
      if num != 9 {
        num += 1
        return false
      } else {
        num = 0
        return true
      }
    }
    
使用一个辅助函数addOne，判断数字是否为9，若是，置0，返回true（进位），否则直接将数字+1，返回false（不进位）。  
从末位开始对数字+1，如果进位，则对其前一位进行+1操作。直到某一位不为9时，直接将其+1，循环结束。如果到达数组首位，则将数组前插入一个1而其他位均为0。

### 移除数组中所有给定的值，返回完成后数组长度

    func removeElement(inout nums: [Int], _ val: Int) -> Int {
      nums = nums.flatMap { return $0 == val ? nil : $0 }
      return nums.count
    }
在flatMap的过程中另开了与nums大小相同的内存空间。

### 判断一个二叉树是否关于中轴对称
    func isSymmetric(root: TreeNode?) -> Bool {
      if root == nil { return true }
      return isMirrored(root?.left, right: root?.right)
    }
    
    func isMirrored(left: TreeNode?, right: TreeNode?) -> Bool {
      if left == nil && right == nil {
        return true
      } else if left?.val != right?.val {
        return false
      }
      return isMirrored(left?.right, right: right?.left) && isMirrored(left?.left, right: right?.right)
    }
    
使用辅助函数，判断两棵树是否是对称的。如果两个根节点都为空，则返回true；如果两个根节点值不同，返回false。不满足这两条，判断左子树的左子树和右子树的右子树以及左子树的右子树和右子树的左子树是否都满足对称。

### 对一个有序数组进行去重
    func removeDuplicates(inout nums: [Int]) -> Int {
      if nums.isEmpty { return 0 }
      var i = 0
      for j in 1..<nums.count {
        if nums[i] != nums[j] {
          i += 1
          nums[i] = nums[j]
        }
      }
      return i + 1
    }
使用两个索引，i指向数组头部，j从数组的第二个开始。让j走到数组尾部，如果j处的值和i处的值不同，则将i++，将j处的值赋值到i处，继续步骤直到j到达数组尾部。  
此时所有的不重复的数字都集中在数组的前 i + 1 个位置上，所以数组长度为 i + 1。

## May 5, 2016 leetcode
### 杨辉三角
For example, given numRows = 5,
Return

    [
         [1],
        [1,1],
       [1,2,1],
      [1,3,3,1],
     [1,4,6,4,1]
    ]
    
代码：

    func generate(numRows: Int) -> [[Int]] {
      if numRows == 0 { return [] }
      var ans = [[1]]
      for _ in 1..<numRows {
        ans.append(nextRow(ans.last!))
      }
      return ans
    }
    
    func nextRow(row: [Int]) -> [Int] {
      var ans = Array<Int>(count: row.count + 1, repeatedValue: 1)
      for i in 0..<ans.count where i != 0 && i != ans.count - 1 {
        ans[i] = row[i - 1] + row[i]
      }
      return ans
    }
特殊情况0，返回空数组。初始化返回数组为[[1]]，使用辅助函数`nextRow(row: [Int]) -> [Int]`，给出当前输入行的下一行。循环n - 1次，每次对nextRow输入当前数组中最后一个元素，将返回结果追加到当前数组中。返回当前数组。