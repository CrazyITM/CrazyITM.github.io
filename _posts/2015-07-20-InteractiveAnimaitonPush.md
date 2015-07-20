---
layout: post
category : Animation
tagline: "Supporting tagline"
tags : [UIViewControllerInteractiveTransitioning]
---


# 为控制器切换提供用户交互
[上篇](http://www.ll.com)文章我们讨论了如何为控制器切换添加过渡动画,这里我们将继续讨论关于如何为其添加用户交互.
1. 讨论官方提供的协议.
2. 讨论类_UIPercentDrivenInteractiveTransition_.
3. 具体实现.


## 官方提供的协议
在使用UIViewControllerAnimatedTransitioning协议的时候.官方介绍:如果想要使用拥有用户交互的动画需要创建遵循 UIViewControllerInteractiveTransitioning的对象.
### 主要方法: 
`startInteractiveTransition:`
[^1]在这个方法中可以得到上下文context,并且在更新UI后更新上下文:
* updateInteractiveTransition: 告诉上下文完成进度,
* cancelInteractiveTransition: 告诉上下文过渡取消,
* finishInteractiveTransition: 告诉上下文过渡完成.


参考: [http://www.captechconsulting.com/blogs/ios-7-tutorial-series-custom-navigation-transitions--more](http://www.captechconsulting.com/blogs/ios-7-tutorial-series-custom-navigation-transitions--more)

[^1]:	Your implementation of this method should use the data in the transitionContext parameter to configure user interactivity for the transition and then start the animations. While tracking user interactions, your event handling code should regularly call the context object’s updateInteractiveTransition: method to report on how much of the transition is now complete. If events indicate that the user has canceled the transition, call the cancelInteractiveTransition method. If events indicate that the transition has finished, call the finishInteractiveTransition method.