---
layout: post
category : Animation
tagline: "Supporting tagline"
tags : [UIColor ,Generaic RGB ,XIb Color]
---


# 为控制器切换提供用户交互
[上篇]文章我们讨论了如何为控制器切换添加过渡动画,这里我们将继续讨论关于如何为其添加用户交互.
1. 讨论官方提供的协议.
2. 讨论类_UIPercentDrivenInteractiveTransition_.
3. 具体实现.


## 官方提供的协议
在使用::UIViewControllerAnimatedTransitioning::协议的时候.官方介绍:如果想要使用拥有用户交互的动画需要创建遵循:: UIViewControllerInteractiveTransitioning::的对象.
### 主要方法: 
`startInteractiveTransition:`
(fn)在这个方法中可以得到上下文context,并且在更新UI后更新上下文:
* updateInteractiveTransition: 告诉上下文完成进度,
* cancelInteractiveTransition: 告诉上下文过渡取消,
* finishInteractiveTransition: 告诉上下文过渡完成.


参考: [http://www.captechconsulting.com/blogs/ios-7-tutorial-series-custom-navigation-transitions--more]