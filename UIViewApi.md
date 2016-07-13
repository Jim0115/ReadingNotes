# APIs of UIView
### Initializing
`init(frame frame: CGRect)`  
使用一个`CGRect`构造一个view，view的`center`和`bound`可以通过`frame`计算出来。  
子类重写此方法时必须先调用父类的init方法。

### Configuring a View’s Visual Appearance
`backgroundColor?`  
view的背景颜色，修改此属性可以产生动画。默认值是nil，表示透明的view。  
<br>
