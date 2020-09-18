---
layout:     post
title:      "自定义ViewController的跳转动画"
subtitle:   " \"试试写个与众不同的跳转动画\""
date:       2018-01-31 09:35:00
author:     "Leezw"
header-img: "img/post-bg-2015.jpg"
tags:
    - iOS
---

在学习iOS动画时看到，别人炫酷的动画效果一直是我学习的动力，在学习如何自定义后发现其实并没那么难，一些渐变，翻转，平移等在UIView上能实现的动画都可以放到跳转动画里，做完例子你也可以，分享一个github上别人做的炫酷的动画效果：[github地址](https://github.com/shu223/AnimatedTransitionGallery)
![https://github.com/shu223/AnimatedTransitionGallery](http://upload-images.jianshu.io/upload_images/1640181-8f6e494722bd2429.gif?imageMogr2/auto-orient/strip)


## 原理

当我们进行视图切换的时候，无论是push，pop还是通过PresentModal和dismiss时，我们可以把跳转动画分割为三个部分，
1：第一个视图控制器显示。
2：临时视图控制器  显示动画效果（并不是真的存在这个控制器，其实只有一个containerView）
3：显示第二个视图控制器。
用形象一点的说法，我们可以将第二部分也看做是一个临时的视图控制器，他执行一段关于UIView的动画，我们只要让动画的开始界面与控制器1一样，结束界面与视图控制器2一样。当动画执行结束，执行到第三步时，把临时的视图控制器释放。这样只要稍微实现一些UIView，CALayer的动画，
具体的实现方法以下图的效果为例子：
![跳转动画Demo.gif](
http://upload-images.jianshu.io/upload_images/1640181-fcf8d0ee3b7b7c32.gif?imageMogr2/auto-orient/strip)

## 用到的协议及方法

###### @protocol UIViewControllerAnimatedTransitioning

实现这个协议的对象就相当于是之前说到的临时视图控制器，协议中主要用到的两个方法是：

    //这个方法中我们只要返回页面跳转动画的执行时间
    -(NSTimeInterval)transitionDuration:(id < UIViewControllerContextTransitioning >)transitionContext;
    //在这个方法中实现具体动画
    -(void)animateTransition:(id < UIViewControllerContextTransitioning >)transitionContext;

第二个方法是跳转动画核心内容，方法中的参数transitionContext是当前临时控制器界面的上下文。通过上下文，我们先拿到临时视图控制器的背景视图containerView，他已经有一个子视图，即fromViewController.view。我们要做的是，先获得toViewController

    UIViewController *toVC = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
然后把当前视图加到内容视图上

    [[transitionContext containerView] addSubview:toVC.view];
剩下的部分就可以发挥我们的系统核心动画的学习水平，比如修改toVC.view的frame，从下方旋转弹出等等。而要实现Demo里的效果，我们需要CAShapeLayer来给toView做遮挡，然后从修改ShapeLayer.path属性，从一个小圆变成一个囊括全屏的大圆，小圆大圆我们通过UIBezierPath画出来，对于ShapeLayer和UIBezierPath不熟悉的同学可以先去补习一下，这里具体的使用也不复杂，也可以直接看下面的代码学习。

    //应用程序的屏幕高度
    #define kWindowH   [UIScreen mainScreen].bounds.size.height
    //应用程序的屏幕宽度
    #define kWindowW    [UIScreen mainScreen].bounds.size.width
  #  
    _circleCenterRect = CGRectMake(120, 320, 50, 50);
    //圆圈1--小圆
    UIBezierPath *smallCircleBP =  [UIBezierPath bezierPathWithOvalInRect:_circleCenterRect];
    //圆圈2--大圆
    //以_circleCenterRect的中心为圆心
    CGFloat centerX = _circleCenterRect.origin.x+_circleCenterRect.size.width/2;
    CGFloat centerY = _circleCenterRect.origin.y+_circleCenterRect.size.height/2;
    //找出圆心到页面4个角 最长的距离 作为半径
    CGFloat r1 = (kWindowW-centerX)>centerX?(kWindowW-centerX):centerX;
    CGFloat r2 = (kWindowW-centerY)>centerY?(kWindowW-centerY):centerY;
    CGFloat radius = sqrt((r1 * r1) + (r2 * r2));
    UIBezierPath *bigCircleBP = [UIBezierPath bezierPathWithOvalInRect:CGRectInset(_circleCenterRect, -radius, -radius)];
画完圆后通过CABasicAnimation执行：

    //设置maskLayer
    CAShapeLayer *maskLayer = [CAShapeLayer layer];//将它的 path 指定为最终的 path 来避免在动画完成后会回弹
    toVC.view.layer.mask = maskLayer;
    maskLayer.path = bigCircleBP.CGPath;
    //执行动画
    CABasicAnimation *maskLayerAnimation = [CABasicAnimation animationWithKeyPath:@"path"];
    maskLayerAnimation.fromValue = (__bridge id)(smallCircleBP.CGPath);
    maskLayerAnimation.toValue = (__bridge id)((bigCircleBP.CGPath));
    maskLayerAnimation.duration = [self transitionDuration:transitionContext];
    maskLayerAnimation.delegate = self;
    [maskLayer addAnimation:maskLayerAnimation forKey:@"path"];
返回时候的动画效果和上面的代码基本相同，只用将大圆和小圆的顺序对调，完成了跳转动画最核心的部分后就是怎么使用这个对象了，无论是navigation和模态跳转，系统都给我们提供了回调方法，我们只需要实现方法，然后把我们之前写好的动画对象返回就可以了。由于给视图添加了shapelayer，可以在CABasicAnimation的代理方法里实现。

    - (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag{
        //告诉 系统这个 transition 完成
        [self.transitionContext completeTransition:![self.transitionContext transitionWasCancelled]];
        //清除 fromVC 的 mask
        [self.transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey].view.layer.mask = nil;
        [self.transitionContext viewControllerForKey:UITransitionContextToViewControllerKey].view.layer.mask = nil;
    }

###### @protocol UINavigationControllerDelegate

这是修改Navigation跳转动画需要实现的协议，当我们通过pop或者push进行跳转时，都会调用下面的方法(协议的其他方法由于暂时用不到，所以就不介绍了)，我们可以看到返回值是id <UIViewControllerAnimatedTransitioning>，就是我们之前编写的动画对象。
    - (nullable id <UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
                                   animationControllerForOperation:(UINavigationControllerOperation)operation
                                                fromViewController:(UIViewController *)fromVC
                                                  toViewController:(UIViewController *)toVC;
方法中的operation参数是一个枚举类型，我们可以判断它的值是 UINavigationControllerOperationPush还是
    UINavigationControllerOperationPop来区别当前的跳转方式。

###### @protocol UIViewControllerTransitioningDelegate

这是修改模态跳转动画需要实现的协议，我们进行跳转PresentModal和dismiss时会分别调用两个方法：

    //PresentModal跳转回调方法
    - (nullable id <UIViewControllerAnimatedTransitioning>)animationControllerForPresentedController:(UIViewController *)presented presentingController:(UIViewController *)presenting sourceController:(UIViewController *)source;
    //dismiss跳转回调方法
    - (nullable id <UIViewControllerAnimatedTransitioning>)animationControllerForDismissedController:(UIViewController *)dismissed;
这两个方法的返回值和上面navigation回调方法的返回值一样，我们只需要初始化我们的实现了UIViewControllerTransitioningDelegate协议的动画对象就可以了。具体的实现以及效果可以查看文章末尾附上的Demo。

###### navigation跳转需要注意的地方

在使用Navigation跳转的时候，我们希望视图1跳转到视图2和视图2返回视图1有自定义动画时，我们只用在视图1中将设置视图1的navigation的delagate。其次delegate的设置最好放在viewWillAppear中。

demo所实现的最终效果只出现在点击按钮时的跳转动画，而不包括像navigation侧滑返回那样的手势动画。如何怕侧滑效果不统一，我们可以先把侧滑返回的效果关闭：

    self.navigationController.interactivePopGestureRecognizer.enabled = NO; 
>// TODO：手势跳转动画实现方法（坑 待填）

[Demo地址](https://github.com/Baymax0/MySampleDemo/tree/master)

[bestswifter 的swift版教程](http://www.jianshu.com/p/ea0132738057)


---


