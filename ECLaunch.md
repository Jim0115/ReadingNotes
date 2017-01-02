# ECLaunch
`ECLaunch`的优势在于，避免了繁琐的`vc1.navigationController?.pushViewController(vc2, animated: true)`写法。在调用时无需考虑当前VC所在的层级，直接调用`ECLaunch.launchViewController(vc)`即刻实现所需的效果。  
在 iOS 8.0之后，系统在`UIViewController`中引入了`show`, `showDetail`两方法，同样可以实现从未知层级直接跳转。  
不同之处在于，系统方式必须在一个UIViewController对象上调用`show`方法。在项目中，ECLaunch多与JLRoute搭配使用。在JLRoute的handler中，只有类而没有具体的对象。

## 代码
### UIViewController Extension
	extension UIViewController {
	  func ecl_frontViewController() -> UIViewController? {
	    var frontVC: UIViewController?
	    
	    if presentedViewController != nil {
	      frontVC = presentedViewController
	    } else if let tabbarVC = self as? UITabBarController {
	      frontVC = tabbarVC.selectedViewController
	    } else if let naviVC = self as? UINavigationController {
	      frontVC = naviVC.topViewController
	    }
	    
	    return frontVC
	  }
	}
对当前VC进行判断，分为三种情况：  

- 存在被present的vc
- 自身是tabbarVC
- 自身是navigationVC

针对三种情况寻找其不同的frontVC。  

### UIWindow Extension

	extension UIWindow {
	  func ecl_visiableViewController() -> UIViewController? {
	    var visiableVC: UIViewController?
	    var frontVC = rootViewController
	    
	    while frontVC != nil {
	      visiableVC = frontVC
	      frontVC = frontVC?.ecl_frontViewController()
	    }
	    
	    return visiableVC
	  }
	}
基于 UIViewController Extension 实现，对一个window从其`rootViewController`开始，迭代寻找其前一个VC。

### ECLaunch 中的私有方法
    private class func standardLaunchViewController(_ vcToLaunch: UIViewController, inWindow window: UIWindow, navigationCreater block: ECLNavigationViewControllerCreateBlock) {
      let visiableVC = window.ecl_visiableViewController()
    
      if vcToLaunch is UINavigationController {
        visiableVC?.present(vcToLaunch, animated: true, completion: nil)
      } else {
        if let navi = visiableVC?.navigationController {
          navi.pushViewController(vcToLaunch, animated: true)
        } else if let navi = visiableVC as? UINavigationController {
          navi.pushViewController(vcToLaunch, animated: true)
        } else {
          let navi = block()
          navi.viewControllers = [vcToLaunch]
          visiableVC?.present(navi, animated: true, completion: nil)
        }
      }
    }
  
通过参数`window`找到visiableVC。首先判断需要launch的VC是否为`UINavigationController`，若是，直接在visiableVC上present。之后，判断visiableVC是否存在navigationController，若是，使用navi直接push参数`vcToLaunch`。再之后，判断visiable自身是否为`UINavigationController`，若是，使用其push参数`vcToLaunch`。最后，即结构中不存在`UINavigationController`时，使用参数`navigationCreater`创建一个`UINavigationController`。
	
	private class func nestedPresentViewController(_ vcToLaunch: UIViewController, inWindow window: UIWindow, navigationCreater block: ECLNavigationViewControllerCreateBlock) {
	  let visiableVC = window.ecl_visiableViewController()
	  
	  if vcToLaunch is UINavigationController {
	    visiableVC?.present(vcToLaunch, animated: true, completion: nil)
	  } else {
	    let navi = block()
	    navi.viewControllers = [vcToLaunch]
	    visiableVC?.present(navi, animated: true, completion: nil)
	  }
	}
	
与上一个方法类似。区别在于不考虑最靠前的VC为navigationVC的情况。即不会push，一定是present。  

	private class func present(_ vcToPresent: UIViewController, inWindow window: UIWindow) {
	  let visiableVC = window.ecl_visiableViewController()
	  visiableVC?.present(vcToPresent, animated: true, completion: nil)
	}
同上，但考虑的因素更少。直接present参数。  

