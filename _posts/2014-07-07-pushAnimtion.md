---
layout: post
category : Animation
tagline: "Supporting tagline"
tags : [pushAnimation ,maskLayer ,shapelayer]
---
#自定义控制器动画切换效果

之前在微博上看到有人转发的这种效果,一直想学习,偶尔一次在一个帖子上看到实现过程,[原文][raywenderlichURL] 使用swift实现.

[raywenderlichURL]: http://25.io/mou/ "Markdown editor on Mac OS X"


**实现效果如下**
<br>
![alt text][id]

[id]:{{site.url}}/assets/pushAnimation/pushAnimation.gif "效果图1"

~\(≧▽≦)/~ 效果太赞了 

##让我们开始吧:
**1.shape layer,maskLayer,CA动画的使用**(关于[layer的使用][LayerURL])

[LayerURL]:layer.html

**2. UIViewControllerAnimatedTransitioning协议和类UIPercentDrivenInteractiveTransition的使用**

**3.其他的小技巧(比如使用xib中如何实现圆角button)**

如果你上述知识不熟悉可以自行点开链接去补充下.这里不做详细介绍

##大致步骤:
1. **在NavgationController 的代理方法中捕获到切换上下文 然后读取目的控制器(ToViewController)的view,贴到上下文的视图上面.**

2. **计算点击的按钮的形状(shaper)  和屏幕的shaper 作为CA动画的开始值和结束值 在目的控制器(ToViewController)上面执行动画.**

3. **动画结束后移除mask,并且通知上下文切换结束了.**

4. **通过UIPercentDrivenInteractiveTransition类可以控制切换的进度,结合手势读取进度值,实现最后的自由控制效果.**


```
Fenced code blocks are like Stardard
Markdown’s regular code blocks, except that
they’re not indented and instead rely on a
start and end fence lines to delimit the code
block.
```
<br/>
原文网址-[-http://www.raywenderlich.com/86521/how-to-make-a-view-controller-transition-animation-like-in-the-ping-app](http://)(本文在此基础修改)

