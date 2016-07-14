# APIs of UIView
### Initializing
`init(frame frame: CGRect)`  
使用一个`CGRect`构造一个view，view的`center`和`bound`可以通过`frame`计算出来。  
子类重写此方法时必须先调用父类的init方法。

### Configuring a View’s Visual Appearance
`@NSCopying var backgroundColor: UIColor?`  
view的背景颜色，animatable。默认值是nil，表示透明的view。  

`var hidden: Bool`  
隐藏view，该属性为false时view仍然可能是隐藏的，原因是该视图的直接或间接父视图是隐藏的。  

`var alpha: CGFloat`  
0.0 - 1.0的浮点数，表示视图的透明度，默认为1.0。0.0表示完全透明，1.0表示完全不透明。此属性只作用于当前视图而对当前视图的子视图无影响。

`var opaque: Bool`  
此属性提示绘图系统如何绘制该view。若为true，表示视图完全不透明，绘图系统会进行优化一些绘制选项以提升性能。若为false，绘图系统会将此view与其他内容进行拼接。默认值为true。  
一个不透明的view要求其所有子view的alpha值为1.0且填满bounds，如果view是不透明的但其子view没有填满bounds或部分/全部子view是透明的，将产生不可预测的结果。如果view部分或全部透明，必须设置此选项为false  
只有自定义的`UIView`的子类需要设置此属性。此属性对于系统提供的视图如`UILabel`,`UIButton`,`UITableViewCell`等无意义。

`var tintColor: UIColor!`  
色调。默认为nil。该属性为nil的视图会使用其距离最近父视图的`tintColor`，若在其视图结构中没有设置`tintColor`，会使用系统默认的浅蓝色。

`var tintAdjustmentMode: UIViewTintAdjustmentMode`

    enum UIViewTintAdjustmentMode : Int {
        case Automatic // 与父视图相同
        case Normal // 正常
        case Dimmed // 灰暗 类似于enable = false状态
    }
`tintColor`的调整模式

`var clipsToBounds: Bool`  
true表示超出该视图可见bounds的部分将被裁剪第掉。默认值为false

`var clearsContextBeforeDrawing: Bool`
在绘制前是否自动清除bounds的内容。如果为true，则会在下一次绘制前清空上一次绘制的内容