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