	let celebrities: [String?] = ["Michael Jackson", nil, "Michael Caine", nil, "Michael Jordan"]
	
	for case let name? in celebrities {
	    name/
	}


	let words = ["1989", "Fearless", "Red"]	let input = "My favorite album is Fearless"	words.contains(where: input.contains)
	// 实际的意义是遍历`words`中的单词，判断`input`中是否含有此单词
	
	words.contains { (String) -> Bool in
    	
	}
	
	input.contains(other: String)
	
不能throw的函数是可以throws的子类型，即在任何需要可throw函数的地方都可以使用非throw的函数  
throws 表示一种能力，可以抛出错误，但不是必须抛出。  
rethrows用于参数可能throws的情况，表示函数是否throw取决于参数。这样在某些情况下不需要使用try-catch

函数式编程的5条原则：

- 函数是一等数据类型，可以像Int或String一样被创建，复制，传递。
- 由于函数是一等数据类型，其可以被用作其他函数的参数。
- 为了允许函数能以不同方式被重用，函数对于相同的输入必须有相同的输出，同时不产生side effect。
- 由于函数对相同的输入一定有相同的输出，应该偏向使用不可变的数据类型，而不是使用函数去改变变量。
- 由于函数不会产生side effect且变量都是不可变的，可以减少对程序状态的跟踪。

使用POP的一个好处是可以基于struct去做，而OOP必须基于类实现。  
协议只允许添加方法和计算属性，即不能通过协议添加`状态`。