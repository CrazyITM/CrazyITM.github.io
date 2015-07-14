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

[id]:{{site.url}}/assets/pushAnimation/pushAnimation.gif "最终效果图""
<!--[id]:../assets/pushAnimation/pushAnimation.gif "最终效果图"
-->
~\(≧▽≦)/~ 效果太赞了 

##我们开始吧:

下面是一些须知:

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

---

####1.首先实现最基本的push操作


这里FirstViewController `touchBegin` 的时候push 进来一个SecondViewController ,并没有什么不同.
<br>
![alt text][easypushid]

[easypushid]:{{site.url}}/assets/pushAnimation/easyPushAnimation.gif "最基本的push"
<!--[easypushid]:../assets/pushAnimation/easyPushAnimation.gif "最基本的push"
-->

<br>

####2.为*NavgationController*添加一个代理
* 新建一个`NavigationDelegate`类继承自`NSObject` 并接受协议 *`<UINavigationControllerDelegate>`*
* 实现代理方法:
				
		- (id <UIViewControllerAnimatedTransitioning>)navigationController(UINavigationController *)navigationController animationControllerForOperation:(UINavigationControllerOperation)operation fromViewController:(UIViewController *)fromVC toViewController:(UIViewController *)toVC  NS_AVAILABLE_IOS(7_0);

* 实现如下:


		- (id <UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController animationControllerForOperation:(UINavigationControllerOperation)operation fromViewController:(UIViewController *)fromVC toViewController:(UIViewController *)toVC{
		   return nil;
		   }

	1. 这里只是返回nil,所以不会有任何变化,但运行后会发现无论push还是pop都会调用该代理方法.
	2. 该代理方法要求我们返回一个id*`<UIViewControllerAnimatedTransitioning>`* 类型的对象.




####3.创建 *id`<UIViewControllerAnimatedTransitioning>`* 吧

*	在继续之前让我们看下官方文档吧 
	
		⚠️: 请以英文为准,中文为笔者即兴翻译,不做质量保证 
		
		Adopt the UIViewControllerAnimatedTransitioning protocol in objects that implement the animations for a custom view controller transition. The methods in this protocol let you define an animator object, which creates the animations for transitioning a view controller on or off screen in a fixed amount of time. The animations you create using this protocol must not be interactive. To create interactive transitions, you must combine your animator object with another object that controls the timing of your animations. 
		
		遵循了这个协议的对象晋升为一个动画者,他创造出一个动画,这个动画执行时间固定,作用是 过渡控制器在屏幕上的的切换.你创建的这个动画者是没有交互的,想要创建有交互的 你得结合另一个对象(后面讲到)来控制你的动画的进度.
		
		In your animator object, implement the transitionDuration: method to specify the duration of your transition and implement the animateTransition: method to create the animations themselves. Information about the objects involved in the transition is passed to your animateTransition: method in the form of a context object. Use the information provided by that object to move the target view controller’s view on or off screen over the specified duration.
		
		你创建的那个动画者,通过实现transitionDuration:方法来指定过渡期间;  
		通过实现animateTransition:方法来创建动画。过渡动画所需要的对象通过animateTransition:方法的上下文对象传递给你。使用上下文提供给你的信息对象实现那些切换动画吧(别忘了有固定时间的)。

		Create your animator object from a transitioning delegate—an object that conforms to the UIViewControllerTransitioningDelegate protocol. When presenting a view controller, st the presentation style to UIModalPresentationCustom and assign your transitioning delegate to the view controller’s transitioningDelegate property. The view controller retrieves your animator object from the transitioning delegate and uses it to perform the animations. You can provide separate animator objects for presenting and dismissing the view controller.
		
		这段话的意思是在遵循了<UIViewControllerTransitioningDelegate>协议的对象,将会在协议的实现中创建这个动画者,来实现对应的过渡动画. (注:bro 我们看到返回值就知道 我们应该创建这个动画者了 我们遵循的是UINavigationControllerDelegate协议,对吧)
		
		To add user interaction to a view controller transition, you must use an animator object together with an interactive animator object—a custom object that adopts the UIViewControllerInteractiveTransitioning protocol. For more on defining interactive transitions, see UIViewControllerInteractiveTransitioning Protocol Reference.
		
		想要使用有交互的动画,您必须使用另一个对象,它必须遵循UIViewControllerInteractiveTransitioning协议.不懂请参考UIViewControllerInteractiveTransitioning协议。










 
原文网址-[-http://www.raywenderlich.com/86521/how-to-make-a-view-controller-transition-animation-like-in-the-ping-app](http://www.raywenderlich.com/86521/how-to-make-a-view-controller-transition-animation-like-in-the-ping-app)(本文在此基础修改)

