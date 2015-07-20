---
layout: post
category : Animation
tagline: "Supporting tagline"
tags : [UIViewControllerInteractiveTransitioning]
---


# 为控制器切换提供用户交互
[上篇](http://www.ll.com)文章我们讨论了如何为控制器切换添加过渡动画,这里我们将继续讨论关于如何为其添加用户交互.
1.   讨论官方提供的协议.
2.   讨论类*UIPercentDrivenInteractiveTransition*.
3.   具体实现.


## 官方提供的协议
在使用UIViewControllerAnimatedTransitioning协议的时候.官方介绍:如果想要使用拥有用户交互的动画需要创建遵循 UIViewControllerInteractiveTransitioning的对象.

### 主要方法: 
`startInteractiveTransition:`
[^1]在这个方法中可以得到上下文context,并且在更新UI后更新上下文:
- updateInteractiveTransition: 告诉上下文完成进度,
- cancelInteractiveTransition: 告诉上下文过渡取消,
- finishInteractiveTransition: 告诉上下文过渡完成.

## 类*UIPercentDrivenInteractiveTransition*
苹果官方提供了一个类* UIPercentDrivenInteractiveTransition* 实现了上述协议的方法,使用简单.
- updateInteractiveTransition
- cancelInteractiveTransition
- finishInteractiveTransition
这几个方法与上面的名字类似,他内部除了通知上下文响应操作之外还帮我们更新了UI.

## 具体实现
苹果提供了一些建议:动画者和UIViewControllerAnimatedTransitioning 的实现者可以使用相同的类[^2], 于是我们修改动画执行者的父类

```
`@interface Cyclo_MaskAnimatior : UIPercentDrivenInteractiveTransition
```
`
并且在navgationController的代理方法中返回Animator

```
`(id \<UIViewControllerInteractiveTransitioning\>)navigationController:(UINavigationController *)navigationController
  interactionControllerForAnimationController:(id \<UIViewControllerAnimatedTransitioning\>) animationController

 return [self.animator isKindOfClass:[Cyclo_MaskAnimatior class](#)]?self.animator:nil;
 }
```
`
现在添加用户交互:以手势为例,将当前显示的View作为手势的控制器,发送消息给动画者,这里发送消息的顺序是 firstViewController - navgationCtontroller - navgationCtontroller.delegate - Animator

手势的部分代码如下:


	-(void)paned:(UIPanGestureRecognizer *)gestureRecognizer
	{
	
	NSLog(@"%f",[gestureRecognizer velocityInView:self.navigationController.view].x);
	
	AninationNavigationController * animaitonNav =(AninationNavigationController *)self.navigationController;
	
	switch (gestureRecognizer.state) {
	case UIGestureRecognizerStateBegan:
	{
	 
	touchDirection =([gestureRecognizer velocityInView:self.view].x >0);
	
	CGPoint translation =[gestureRecognizer  locationInView:self.view];
	
	id obj = [NSValue valueWithCGPoint:translation];
	
	SecondViewController *secondVC =[[SecondViewController alloc]init];
	
	[animaitonNav setAnimationType:TransformType_cyclo Context:obj];
	
	[animaitonNav pushViewController:secondVC animated:YES];
	
	break;
	}
	case UIGestureRecognizerStateChanged:
	{
	
	CGPoint translation =[gestureRecognizer  translationInView:self.navigationController.view];
	
	CGFloat completionProgress =(fabs(-translation.x))/CGRectGetWidth(self.navigationController.view.bounds);
	
	[animaitonNav updateInteractiveTransition:completionProgress];
	
	break;
	}
	case UIGestureRecognizerStateEnded:
	{
	if(  ([gestureRecognizer velocityInView:self.navigationController.view].x<0) != touchDirection)
	{
	[animaitonNav finishInteractiveTransition];
	}else
	{
	[animaitonNav cancelInteractiveTransition];
	}
	break;
	}
	
	default:
	{
	[animaitonNav cancelInteractiveTransition];
	break;
	}
	}
	}
	



参考: [http://www.captechconsulting.com/blogs/ios-7-tutorial-series-custom-navigation-transitions--more](http://www.captechconsulting.com/blogs/ios-7-tutorial-series-custom-navigation-transitions--more)

[^1]:	Your implementation of this method should use the data in the transitionContext parameter to configure user interactivity for the transition and then start the animations. While tracking user interactions, your event handling code should regularly call the context object’s updateInteractiveTransition: method to report on how much of the transition is now complete. If events indicate that the user has canceled the transition, call the cancelInteractiveTransition method. If events indicate that the transition has finished, call the finishInteractiveTransition method.

[^2]:	The transition delegate and the transition animator can, if you wish, be defined within a single custom class, but the class must adopt both protocols.