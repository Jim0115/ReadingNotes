# Class 8 @ April 22, 2016
## Animation
    
    + animateWithDuration:delay:options:animations:completion: // of UIView
    
只能作用于frame, transform and alpha

### UIViewAnimationOptions
`BeginFromCurrentState` // interrupt others, in-progress animations of there properties  
`AllowUserInteraction` // allow gestures to get processed while animation is in progress  
`LayoutSubviews` // animate the relayout of subviews along with a parent's animation  动画过程中保证子视图跟随运动  
`repeat` // repeat indefinitely  
`Autoreverse` // play animation forwards, then backwards  
`OverrideInheritedDuration` // if not set, use duration of any in-progress animation  
`OverrideInheritedCurve` // if not set, just interpolate between current and end state image  
`CurveEaseInEaseOut` // slower at the beginning, normal throughout, then slow at end  
`CurseEaseIn` // slower at the beginning, but then constant through the rest  
`CurveLinear` // same speed throughout

---
### 修改view的其他属性时使用

    + (void)transitionWithView:(UIView *)view 
                      duration:(NSTimeInterval)duration 
                       options:(UIViewAnimationOptions)options
                    animations:(void (^)(void))animations 
                    completion:(void (^)(BOOL finished))completion
                    
#### Options
`UIViewAnimationOptionsTransitionFlipFrom{Left, Right, Top, Bottom}` // flip view over  
`UIViewAnimationOptionsTransitionCrossDissolve` // dissolving from old to new state  
`UIViewAnimationOptionsTransitionCurl{Up, Down}` // curling up or down 卷起  
  
将在屏幕外渲染新视图，然后通过动画进行视图切换

### 使用动画过渡到新View

    + (void)transitionFromView:(UIView *)fromView
                        toView:(UIView *)toView
                      duration:(NSTimeInterval)duration
                       options:(UIViewAnimationOptions)options
                    completion:(void (^)(BOOL finished))completion