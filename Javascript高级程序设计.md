# JavaScript高级程序设计
June 2, 2016 借于学校图书馆

---
## 第二章 在HTML中使用JavaScript
#### `<script>`元素
向HTML页面中插入JavaScript的主要方法就是使用`<script>`元素。  
使用`<script>`元素的方式有两种：

* 直接在页面中嵌入JavaScript代码
* 包含外部JavaScript文件

包含在`<script>`元素内部的JavaScript代码将被从上至下依次**解释**。在解释器对`<script>`元素内部的所有代码求值完毕之前，页面中的其余内容都不会被浏览器加载或显示。  
在页面中嵌入JavaScript代码时，代码的任何地方都不要出现`"</script>"`字符串，否则将被视作JavaScript代码终止。

    <script type="text/javascript">
      function sayScript() {
        alert("</script>")
      }
    </script>
    
通过将字符串分割为两部分可以解决这个问题：

    <script type="text/javascript">
      function sayScript() {
        alert("</scr" + "ipt>")
      }
    </script>
    
##### 包含外部JavaScript文件
`<script type="text/javascript" src="example.js"></script>`  
需要注意，带有scr属性的`<script>`元素不应该在标签和结束标签之间包含额外的JS代码。  
`<script>`和`<img>`元素类似，都可以包含HTML所在域外的的URL，如：  

    <script type="text/javascript" src="http://www.example.com/somefile.js"></script>

所使用的外部JS文件必须是可信任的。  
如存在多个`<script>`元素，浏览器将为按照出现的先后顺序依次进行解析。

#### 标签的位置
按照惯例，所有的`<script>`元素都应该放在页面的`<head>`元素中，如：

    <html>
      <head>
        <title>Examle</title>
        <script type="text/javascript" src="example.js"></script>
      </head>
      <body>
        <!- content here -->
      </body>
    </html>
    
这种做法目的是把所有外部文件的引用放在相同的地方，但是这样做意味着必须等到全部JavaScript代码都被下载、解析和执行完成以后，才能开始呈现页面的内容（浏览器在遇到`<body>`标签时才开始呈现内容）。对于需要很多JavaScript代码的页面来说，这样会导致呈现页面时明显的延迟，而延迟期间浏览器窗口一片空白。为了避免这种情况，现代Web应用程序一般都把JavaScript引用放在`<body>`元素中，页面的内容后面，如下：

    <html>
      <head>
        <title>Examle</title>
      </head>
      <body>
        <!- content here -->
        <script type="text/javascript" src="example.js"></script>
      </body>
    </html>
    
#### 延迟脚本
在HTML4.01中定义了defer属性，表示脚本在执行过程中不会影响页面的构造。所有，脚本会被延迟到页面解析完毕后再执行。如：

    <html>
      <head>
        <title>First page with Javascript</title>
        <script type="text/javascript" defer="defer" src="example.js"></script>
      </head>
      <body>
        Hello World!
      </body>
    </html>
    
#### 嵌入代码和外部文件
在HTML中嵌入JavaScript代码虽然没有问题，但一般认为最好的做法还是尽可能使用外部文件。使用外部文件具有以下优点：

* 可维护性：开发人员在不触及HTML标记的情况下编辑JS代码
* 可缓存：如果两个页面使用同一文件，那么这个文件只需被下载一次
* 可适应未来：HTML和XHTML包含外部文件的语法相同

## 第三章：基本概念
### 语法
#### 标识符
变量、函数、属性的名字，或函数的参数，是按照以下规则组合的一个或多个字符：

* 由字母、下划线_、数字、美元符$组成  
* 第一个字符不能是数字

#### 风格
语句结尾的分号不是必须的，但建议在任何时候都不应该省略  
即使if/for/while等只有一条语句，仍然建议使用大括号

### 变量
变量是松散类型的，可以用来保存任何类型的数据，即每个变量仅仅是一个用于保存值的占位符而已。  
定义变量使用`var`：

    var message;
    var hello = "hello world";
    var num = 3;
    
### 数据类型
基本数据类型：

* Undefined
* Null
* Boolean
* Number
* String

复杂数据类型：Object  
Object的本质是由一组无序的键值对组成的  

不支持其他任何自定义类型

#### typeof操作符
使用typeof操作符将返回下列某个字符串：

* "undefined" 值未定义
* "boolean" 布尔值
* "string" 字符串
* "number" 数值
* "object" 对象或null
* "function" 函数

例子：

    var msg = "sth";
    alert(typeof msg);  // string
    alert(typeof(msg));  // string
    alert(typeof 20);  // number 

typeof的操作数可以是变量，也可以是数值字面量。**typeof是一个操作符而不是一个函数，因此圆括号不是必须的**

#### Undefined类型
Undefined类型只有一个值，即undefined。在使用var声明变量但并未对其进行初始化时，这个变量的值就是undefined。

    var msg;
    alert(msg); // "undefined"
    
包含undefined值的变量与未定义的变量存在不同，如：

    var msg;
    alert(msg); // "undefined"
    
    alert(age); // error
    
对于尚未声明的变量，只能执行一个操作，即使用typeof检测其类型
    
    var msg;
    alert(typeof msg);  // "undefined"
    
    alert(typeof age);  // "undefined"
    
#### Null类型
Null类型是第二个只有一个值的类型，这个特殊值为null。null值表示一个空对象指针，这也是`typeof null`返回"object"的原因。

    var person = null;
    alert(typeof person); // object
    
如果定义的对象准备在将来保存一个对象，那么最好将该变量初始化为null。这样，只要直接检查null值旧可以知道相应的变量是否已经保存了一个对象的引用。

    if (car != null) {
      // do sth with cat
    }

undefined值是派生自null值的，所以
    
    alert(undefined == null) // true
    
#### Boolean类型
Boolean类型只有两个字面值，true和false。不能保证true等于1，false等于0。  
虽然Boolean类型的值只有两个，但所有类型的值都有与这两个Boolean值等价的值。要将一个值转换为对应的Boolean值，可以调用转型函数Boolean()

    var msg = "hello";
    alert(Boolean(msg));  // true
    
可以对任何类型的值调用`Boolean()`函数，而且总会返回一个Boolean值。至于返回的值是true还是false，取决于转换值的数据类型及实际值。

|  数据类型 | 转换为true的值 | 转换为false的值 |
| ------------ | ------------- | ------------ |
| Boolean | true  | false |
| String | 任何非空字符串  | 空字符串 |
| Number | 任何非零数字，包括无穷大 | 0和NaN|
|Object|任何对象|null|
|Undefined||undefined|

控制流语句将自动执行Boolean转换。在执行这样的转换时，确切的知道使用的是什么变量至关重要。

    var msg = "hello";
    if (msg) {
      alert(msg);
    }

#### Number类型
    var intNum = 10;
    alert(intNum); // "10"
    
    var octalNum1 = 070;
    var octalNum2 = 088;
    alert(octalNum1); // "56"
    alert(octalNum2); // "88"
    
    var hexNum1 = 0xA;
    var hexNum2 = 0x1f;
    alert(hexNum1); // 10
    alert(hexNum2); // 31
    
0开头的数字表示8进制，后面的数字必须为0-7，否则将忽略开头的0解析为10进制。  
0x开头的数字表示10进制，后跟0-9,A-F。A-F大小写无限制。
##### 浮点数值
浮点数值表示该数值中必须包含一个小数点，并且小数点后必须至少有一位数字。虽然小数点前可以没有整数，但不推荐这种做法。

    var floatNum1 = 1.1;
    var floatNum2 = 0.1;
    var floatNum3 = .1; // not recommand
    
由于保存浮点数所需内存空间为保存整数的两倍，所以如果小数点后没有跟任何数字或小数点后数字为0，则该值会被保存为整数。

##### 数值范围
最大数值`Number.MAX_VALUE`，通常是1.7976931348623157e+308。  
最小数值`Number.MIN_VALUE`，通常是5e-324。  
超出这个范围的数字自动转换为`Infinity`和`-Infinity`。将无法继续参与计算。  
`isFinity()`函数判断数值是否为有穷的。

##### NaN
表示原本要返回数值的操作数未返回数值的情况。任何数值除以0会返回NaN，因此不会影响其他代码的执行。  
任何涉及NaN的操作都会返回NaN。NaN与任何数都不相等，包括NaN本身。
使用`isNaN()`函数判断是否“不是数值”。

    alert(isNaN(NaN)); // true
    alert(isNaN(10)); // false 
    alert(isNaN("10")); // false
    alert(isNaN("Blue")); // true
    alert(isNaN(true)); // false
    alert(isNaN("true")); // true
    alert(isNaN("0xaa")); // false
    
##### 数值转换
有3个函数可以将非数值转换为数值:`Number()`,`parseInt()`和`parseFloat()`

`Number()`: 

* true -> 1 / false -> 0
* 数字不转换
* null -> 0
* undefined -> NaN
* 字符串
    * 字符串只包含数字，转换为10进制数值，忽略前导0 `"011" -> 11 / "123" -> 123`
    * 有效的浮点格式转换为浮点数 `"1.1" -> 1.1`
    * 有效的16进制格式，转换为对应的10进制 `"0xf" -> 15`
    * 空字符串 -> 0
    * 其他字符串 -> NaN
* 对象：调用`valueOf()`方法，按照上述规则进行判断。若结果为NaN，则调用`toString()`方法，再次进行转换，返回结果
 ---
    alert(Number("Hello World!")); // NaN
    alert(Number("")); // 0
    alert(Number("000011")); // 11
    alert(Number(true)); // 1
    
`parseInt()`:  
忽略字符前面的空格，知道找到第一个非空字符。如果第一个非空字符不是负号或数字字符，parseInt()返回NaN，对空字符串同样返回NaN。如果第一个字符是数字字符，则继续解析第二个字符，直到遇到非数字字符或字符串结束。parseInt()同样可以识别出各种进制的数字。

    alert(parseInt("1234blue")); // 1234
    alert(parseInt("")); // NaN
    alert(parseInt("0xa")); // 10
    alert(parseInt(22.5)); // 22
    alert(parseInt("070", 8)); // 56
    alert(parseInt("70")); // 70
    alert(parseInt("0xfqwer")); // 15
    
推荐在任何情况下都明确指定基数，避免错误解析。  
<br/>
`parseFloat()`:  
与parseInt()类似，也是从第一个字符开始解析，直到字符串末尾或遇到无效字符。不同的是，parseFloat()遇到的一个小数点是有效的，将被视为浮点数中的小数点。另外，parseFloat()只解析十进制值。最后，没有小数点或小数点后都是0，会返回整数。

    alert(parseFloat("1234blue")); // 1234
    alert(parseFloat("0xa")); // 0    
    alert(parseFloat("22.5")); // 22.5
    alert(parseFloat("22.34.5")); // 22.34
    alert(parseFloat("0908.5")); // 908.5
    alert(parseFloat("3e5")); // 300000
    
#### String类型
字符串的长度可以通过其`length`属性获得。  
##### 不可变 
字符串都是不可变的。要改变一个字符串，首先要销毁原字符串，在用新字符串去填充该变量。

    var str = "Java";
    str += "Script";
    
创建"Java" -> 创建"Script" -> 创建"JavaScript" -> 销毁"Java"和"Script"  
##### 转换为字符串
使用`toString()`方法获得对应的字符串，除null和undefined外都有一个`toString()`方法。  
数值的`toString()`方法可以传递一个参数，表示转换的目标进制。范围是2-36。  
<br/>
使用`String()`方法转换为字符串：

* 如果有`toString()`方法，调用该方法
* 如果为null，返回"null"
* 如果为undefined，返回"undefined"

### 函数
    function functionName(arg1, arg2, ... , argN) {
        statements
    }
函数在定义时不必指定返回值。实际上，任何函数在任何时候都可以通过return返回值。  
函数在执行return语句之后停止并自动退出。  
不带有返回值的return语句返回undefined。  
>推荐的做法是要么让函数始终都返回一个值，要么永远都不要返回值。否则会给调试带来不便。

#### 参数
函数不介意传递进来多少参数，也不在乎传递进来的参数是什么类型。也就是说，即使定义的函数只接受两个参数，在调用这个函数时也未必一定要传递两个参数。可以这样做的原因是，函数的参数在内部是用一个数组来表示的。函数接收到的始终是这个数组，而不关心数组包含哪些参数。在函数内部可以通过`arguments`访问这个参数数组，从中获取参数。  
其实，`arguments`只是与数组类似（并不是Array的实例）。可以使用方括号访问每一个元素。使用`length`获取参数数量。  
不能使用for-in遍历`arguments`：

    function sayHello(name, message) {
      for (var arg in arguments) {
        alert(arg); // 0 1 2 3 4 5 6 7 8
      }
    }
    
    sayHello("Jack", "how's going.",34,5,1,7,5,4,3);
    
判断参数的个数可以一定程度上做到重载：

    function doAdd() {
      if (arguments.length == 1) {
        alert(arguments[0] + 10);
      } else if (arguments.length == 0) {
        alert("Nothing input");
      } else {
        var ans = 0;
        for (var i = 0; i < arguments.length; i++) {
          ans += arguments[i];
        }
        alert(ans);
      }
    }
    
    doAdd();
    doAdd(1);
    doAdd(1,2,3);
    
最后，没有传递值的命名参数将自动被赋予undefined，这和声明变量但没有初始化相同。  
所有参数都是值，无法通过引用传递参数。

#### 没有重载
函数参数由包含0到多个值的数组表示的，所以函数没有签名，无法实现函数重载。  
如果定义了两个名字相同的函数，则后定义的函数覆盖原先定义的函数。

## 第四章：变量、作用域和内存问题
#### 传递参数
Javascript中所有参数的传递都是值类型。也就是，函数内参数是对传递参数的复制。所以，在函数内部对参数进行的任何操作对外部的变量都是没有影响的。

    function changeName(p) {
      p.name = "Hi";
    }
    
    var person = new Object();
    
    changeName(person);
    
    alert(person.name); // Hi