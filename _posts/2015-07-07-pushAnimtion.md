---
layout : post
category : Animation
tagline : "Supporting tagline"
tags :[pushAnimation ,maskLayer ,shapelayer,UIViewControllerAnimatedTransitioning,UIPercentDrivenInteractiveTransition]
---


#	自定义控制器动画切换效果

之前在微博上看到有人转发的这种效果,一直想学习,偶尔一次在一个帖子上看到实现过程,[原文][raywenderlichURL] 使用swift实现.

[raywenderlichURL]: http://www.raywenderlich.com/86521/how-to-make-a-view-controller-transition-animation-like-in-the-ping-app "原文"


**实现效果如下**
<br>
![alt text][finishid]

<!--[finishid]:../assets/pushAnimation/pushAnimation.gif "最终效果图"
-->
[finishid]:{{site.url}}/assets/pushAnimation/pushAnimation.gif "最终效果图"

~\(≧▽≦)/~ 效果太赞了 .
<br>

## 我们开始吧:


###  大致步骤:


1. **在NavgationController 的代理方法中捕获到切换上下文 然后读取目的控制器(ToViewController)的view,贴到上下文的视图上面.**

2. **计算点击的按钮的形状(shaper)  和屏幕的shaper 作为CA动画的开始值和结束值 在目的控制器(ToViewController)上面执行动画.**

3. **动画结束后移除mask,并且通知上下文切换结束了.**

4. **通过UIPercentDrivenInteractiveTransition类可以控制切换的进度,结合手势读取进度值,实现最后的自由控制效果.(该步骤实现交互型动画,篇幅原因已分篇** [传送门][interactiveURL]**)**

[interactiveURL]: http://www.raywenderlich.com/86521/how-to-make-a-view-controller-transition-animation-like-in-the-ping-app "交互型动画"


####    1.首先实现最基本的push操作


这里FirstViewController `touchBegin` 的时候push 进来一个SecondViewController ,并没有什么不同.
<br>
![alt text][easypushid]

<!--[easypushid]:../assets/pushAnimation/easyPushAnimation.gif "最基本的push"
-->

[easypushid]:{{site.url}}/assets/pushAnimation/easyPushAnimation.gif "最基本的push"

<br>




####    2.为*NavgationController*添加一个代理
* 新建一个`NavigationDelegate`类继承自`NSObject` 并接受协议 *`<UINavigationControllerDelegate>`*
* 实现代理方法:
<br>

```
	-(id <UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
                                   animationControllerForOperation:(UINavigationControllerOperation)operation
                                                fromViewController:(UIViewController *)fromVC
                                                  toViewController:(UIViewController *)toVC  ;
                                                  
```                                          

* 实现如下:

```
-(id <UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
                                   animationControllerForOperation:(UINavigationControllerOperation)operation
                                                fromViewController:(UIViewController *)fromVC
                                                  toViewController:(UIViewController *)toVC {
		   return nil;
		   }
		   
```

	1. 这里只是返回nil,所以不会有任何变化,但运行后会发现无论push还是pop都会调用该代理方法.
	2. 该代理方法要求我们返回一个id*`<UIViewControllerAnimatedTransitioning>`* 类型的对象.




####    3.创建 *id`<UIViewControllerAnimatedTransitioning>`* 吧

*	在继续之前让我们看下官方文档吧 
	
```
⚠️: 请以英文为准,中文为笔者即兴翻译,不做质量保证 
		
		Adopt the UIViewControllerAnimatedTransitioning protocol in objects that implement the animations for a custom view controller transition. The methods in this protocollet you define an animator object, which creates the animations for transitioning a view controller on or off screen in a fixed amount of time. The animations you create using this protocol must not be interactive. To create interactive transitions, you must 
		combine your animator object with another object that controls the timing of your 
		animations. 
		
		遵循了这个协议的对象晋升为一个动画者,他创造出一个动画,这个动画执行时间固定,作用是 过渡控制器在屏幕上的的切换.你创建的这个动画者是没有交互的,想要创建有交互的 你得结合另一个对象(后面讲到)来控制你的动画的进度.
		
		In your animator object, implement the transitionDuration: method to specify the duration of your transition and implement the animateTransition: method to create 	the animations themselves. Information about the objects involved in the transition is passed to your animateTransition: method in the form of a context object. Use the information provided by that object to move the target view controller’s view on or off screen over the specified duration.
		
		你创建的那个动画者,通过实现transitionDuration:方法来指定过渡期间;  通过实现animateTransition:方法来创建动画。过渡动画所需要的对象通过animateTransition:方法的上下
		文对象传递给你。使用上下文提供给你的信息对象实现那些切换动画吧(别忘了有固定时间的)。

		Create your animator object from a transitioning delegate—an object that conforms to
		the UIViewControllerTransitioningDelegate protocol. When presenting a view
		controller, st the presentation style to UIModalPresentationCustom and assign your
		transitioning delegate to the view controller’s transitioningDelegate property. The 
		view controller retrieves your animator object from the transitioning delegate and 
		uses it to perform the animations. You can provide separate animator objects for 
		presenting and dismissing the view controller.
		
		这段话的意思是在遵循了<UIViewControllerTransitioningDelegate>协议的对象,将会在协议的实现中创建
		这个动画者,来实现对应的过渡动画. (注:bro 我们看到返回值就知道 我们应该创建这个动画者了 我们遵循的是
		UINavigationControllerDelegate协议,对吧)
		
		To add user interaction to a view controller transition, you must use an animator 
		object together with an interactive animator object—a custom object that adopts the 
		UIViewControllerInteractiveTransitioning protocol. For more on defining interactive 
		transitions, see UIViewControllerInteractiveTransitioning Protocol Reference.
		
		想要使用有交互的动画,您必须使用另一个对象,它必须遵循UIViewControllerInteractiveTransitioning
		协议.不懂请参考UIViewControllerInteractiveTransitioning协议。
		
关于两个代理方法:
		- (void)animateTransition:(id<UIViewControllerContextTransitioning>)transitionContext

		UIKit calls this method when presenting or dismissing a view controller. Use this
		method to configure the animations associated with your custom transition. You can
		 use view-based animations or Core Animation to configure your animations.
		 
		 在控制器切换的时候UIKit会调用这个代理方法,你的过渡效果将在这里实现,你可以使用基础动画或者CA动画来
		 配置你的动画者,
		 
		All animations must take place in the view specified by the containerView property 
		of transitionContext. Add the view being presented (or revealed if the transition 
		involves dismissing a view controller) to the container view’s hierarchy and set up 
		any animations you want to make that view move into position. If you want to draw to 
		the screen directly without a view, use this method to configure a CADisplayLink 
		object instead.
		
	　　所有的动画都必须发生在上下文(transitionContext)的containerView上。把将要显示的视图添加到
	　　contanerView的视图层次上(如果涉及到dismiss一个控制器,将要显示的就是下面的那个),并设置你想要的位移
	　　动画。如果你不想使用View,而是直接屏幕上绘制那么你得在这个方法里使用CADisplayLink.

		You can retrieve the view controllers involved in the transition from the
		viewControllerForKey: method of transitionContext. For more information about the 
		information provided by the context object, see 
		UIViewControllerContextTransitioning Protocol Reference.
		
		你可以通过viewControllerForKey: 方法来获取相关的控制器

```
 * 开始coding..


CommonAnimator.h

```
#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>
//CommonAnimator 类 接受了 <CommonAnimator> 协议
@interface CommonAnimator : NSObject<CommonAnimator>
@end

```

CommonAnimator.m

```
#import "CommonAnimator.h"

static  NSTimeInterval const  duringTime = 0.5;

@implementation CommonAnimator

- (NSTimeInterval)transitionDuration:(id <UIViewControllerContextTransitioning>)transitionContext
{//返回 动画持续的时间
    return duringTime;
}
- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext
{

//1. 读取上下文中的 containerView
  UIView * containerView =   transitionContext.containerView;
        
   UIViewController * toViewController =[transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];

//2. 把将要显示的视图添加到containerView上
    [containerView addSubview:toViewController.view];
    
   dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(duringTime * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    //3. 动画结束后通知上下文 过渡已经完成
        [transitionContext completeTransition:YES];
    });
    
}


@end


```


NavigationDelegate.h

```
#import "NavigationDelegate.h"
#import "CommonAnimator.h"
@implementation NavigationDelegate


- (id <UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
                                   animationControllerForOperation:(UINavigationControllerOperation)operation
                                                fromViewController:(UIViewController *)fromVC
                                                  toViewController:(UIViewController *)toVC
{
//navgationdelegate 代理方法中 将CommonAnimator对象返回
        CommonAnimator * animator =[[CommonAnimator alloc]init];
        return animator;
}



@end
```
* 运行后发现没有任何动画效果. ok,我们返回 `CommonAnimator` 继续添加动画

```
- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext
{
  UIView * containerView =   transitionContext.containerView;
    
//    UIViewController * fromViewController =[transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
    
    UIViewController * toViewController =[transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];

    [containerView addSubview:toViewController.view];
    
    //目的View
    UIView * toView =toViewController.view;
    
    toView.alpha = 0;
     //最基本的动画
    [UIView animateWithDuration:duringTime animations:^{
        toView.alpha = 1;
    } completion:^(BOOL finished) {
        [transitionContext completeTransition:YES];
    }];
    
}

```
![alt text][baseAnimationID]

<!--[baseAnimationID]:../assets/pushAnimation/基本动画切换.gif "基本动画切换"
-->
[baseAnimationID]:{{site.url}}/assets/pushAnimation/基本动画切换.gif "基本动画切换"



* 动画有了,我现在想实现一个效果,当touchBegin的时候从touch的位置开始向外辐射

	新建一个Cyclo_MaskAnimatior类,他的beginPosition 属性用来储存开始的位置
	
	
	Cyclo_MaskAnimatior
	
```
#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>
@interface Cyclo_MaskAnimatior : NSObject<UIViewControllerAnimatedTransitioning>


+(Cyclo_MaskAnimatior *)animatiorWithBeginPosition:(CGPoint )beginPosition;

@end









#import "Cyclo_MaskAnimatior.h"

@interface Cyclo_MaskAnimatior ()

@property (nonatomic,assign) CGPoint   beginPosition ;

@property (nonatomic,strong) CAShapeLayer  * maskLayer ;

@property (nonatomic,strong) id <UIViewControllerContextTransitioning>   transitionContext ;


@end

static  NSTimeInterval const  duringTime = 0.5;

static  NSString * const  PathKey = @"path";


@implementation Cyclo_MaskAnimatior


+(Cyclo_MaskAnimatior *)animatiorWithBeginPosition:(CGPoint )beginPosition
{
    Cyclo_MaskAnimatior * animator =[[Cyclo_MaskAnimatior alloc]init];
    
    animator .beginPosition = beginPosition;
    
    return animator;
}

- (NSTimeInterval)transitionDuration:(id <UIViewControllerContextTransitioning>)transitionContext
{
    return duringTime;
}

- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext
{
    self.transitionContext = transitionContext;
    
    UIView * containerView =   transitionContext.containerView;
    
    UIViewController * toViewController =[transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
    
    [containerView addSubview:toViewController.view];
    
    //目的View
    UIView * toView =toViewController.view;
    
    
    //1.  原始的path
    UIBezierPath * circleMaskPathInitial =[UIBezierPath bezierPathWithOvalInRect:CGRectInset(CGRectMake(self.beginPosition.x, self.beginPosition.y, 0, 0), -50,-50)];
    
    //获取距离左右边距的最大值
    CGFloat distanceH =( self.beginPosition.x > (toView.frame.size.width - self.beginPosition.x))?self.beginPosition.x : (toView.frame.size.width - self.beginPosition.x);
    //获取距离上下边距的最大值
    CGFloat distanceV =( self.beginPosition.y > (toView.frame.size.height -  self.beginPosition.y))?self.beginPosition.y : (toView.frame.size.height -  self.beginPosition.y);
    //获取最小圆的半径
    double radius = sqrt((distanceH*distanceH) + (distanceV*distanceV));
    
    //2. 最终的path
    UIBezierPath * circleMaskPathFinal =[UIBezierPath bezierPathWithOvalInRect:CGRectInset(CGRectMake( self.beginPosition.x, self.beginPosition.y, 0, 0), -radius,-radius)];
    
    
    CABasicAnimation * pathAnimation =[CABasicAnimation animationWithKeyPath:PathKey];
    
    pathAnimation . fromValue = (__bridge id)(circleMaskPathInitial.CGPath);
    
    pathAnimation . toValue = (__bridge id)(circleMaskPathFinal.CGPath);
    
    pathAnimation.duration = duringTime;
    
    pathAnimation.removedOnCompletion = YES;
    
    pathAnimation.delegate = self;
    
    toView.layer.mask = self.maskLayer;
    
    [self.maskLayer setValue:pathAnimation.toValue forKey:PathKey];
    
    [self.maskLayer addAnimation:pathAnimation forKey:PathKey];

}

//动画结束后调用
- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag
{
        if (self.transitionContext) {
        //动画 执行结束后  上下文是否要完成 的决定条件
        // 上下文是非被取消
        [self.transitionContext completeTransition:(![self.transitionContext transitionWasCancelled])];

        UIViewController * toViewController =[self.transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
        
        toViewController.view.layer.mask = nil;
    }
}





#pragma mark - setter && getter

-(CAShapeLayer *)maskLayer
{
    if (_maskLayer) {
        return _maskLayer;
    }
    _maskLayer =[CAShapeLayer new];
    return _maskLayer;
}

@end


```

此时我们拥有两个Animator,我觉得应该在切换控制器的时候应该能够选择Animator,下面对导航控制器和他的代理进行修改

```
#import <UIKit/UIKit.h>
#import "NavigationDelegate.h"

@interface AninationNavigationController : UINavigationController

-(void)pushViewController:(UIViewController *)viewController WithType:(TransformType )type Context:(id)context;


-(UIViewController *)popViewControllerWithType:(TransformType)type Context:(id)context;


@end


```

```

#import "AninationNavigationController.h"
@interface AninationNavigationController ()
@property (nonatomic,strong) NavigationDelegate   * navgationDelegate ;

@end

@implementation AninationNavigationController


-(void)pushViewController:(UIViewController *)viewController WithType:(TransformType )type Context:(id)context
{
    self.navgationDelegate.context = context;
    self.navgationDelegate.transformType = type;
    [self pushViewController:viewController animated:YES];
    
}

-(UIViewController *)popViewControllerWithType:(TransformType)type Context:(id)context
{
    self.navgationDelegate.context = context;
    self.navgationDelegate.transformType = type;
    return [self popViewControllerAnimated:YES];
}

-(instancetype)initWithRootViewController:(UIViewController *)rootViewController
{
    self =[super initWithRootViewController:rootViewController];
    
    if (self) {
        _navgationDelegate =[[NavigationDelegate alloc]init];
        self.delegate = _navgationDelegate;
    }
    
    return self;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}


@end


```

```
#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>

typedef NS_ENUM(NSInteger, TransformType)
{
    TransformType_cyclo= 1,
    TransformType_common=2,
};


@interface NavigationDelegate : NSObject<UINavigationControllerDelegate>


@property (nonatomic,strong) id  context ;

@property (nonatomic,assign) TransformType  transformType ;


@end

```


```
#import "NavigationDelegate.h"
#import "CommonAnimator.h"
#import "Cyclo_MaskAnimatior.h"

@interface NavigationDelegate ()

@end

@implementation NavigationDelegate


- (id <UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
                                   animationControllerForOperation:(UINavigationControllerOperation)operation
                                                fromViewController:(UIViewController *)fromVC
                                                  toViewController:(UIViewController *)toVC
{
    
    
    switch (self.transformType) {
        case TransformType_cyclo:
        {
            CGPoint point = [self.context CGPointValue];
            Cyclo_MaskAnimatior * animator =[Cyclo_MaskAnimatior animatiorWithBeginPosition:point];
            return animator;
        }
            break;
        case TransformType_common:
        {
            CommonAnimator * animator =[[CommonAnimator alloc]init];
            return animator;
        }
        default:
            break;
    }
    
    return nil;
}



@end

```

使用如下:

```
@implementation FirstViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor =[UIColor redColor];
    // Do any additional setup after loading the view.
}

-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    SecondViewController *secondVC =[[SecondViewController alloc]init];

    
    UITouch * touch = [touches anyObject];
    
     CGPoint touchPoint =   [touch locationInView:self.view];

    id obj = [NSValue valueWithCGPoint:touchPoint];

    AninationNavigationController * animaitonNav =(AninationNavigationController *)self.navigationController;
    
    [animaitonNav pushViewController:secondVC WithType:TransformType_cyclo Context:obj];
}

```

```
#import "SecondViewController.h"
#import "AninationNavigationController.h"
@interface SecondViewController ()

@end

@implementation SecondViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor =[UIColor blueColor];
    // Do any additional setup after loading the view.
}


-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    AninationNavigationController * aninationNAv =(AninationNavigationController *) self.navigationController;
    
    [aninationNAv popViewControllerWithType:TransformType_common Context:nil];
}


@end

```
效果如下:

![alt text][CAAnimationID]

<!--[CAAnimationID]:../assets/pushAnimation/CA动画切换.gif "CA动画切换"
-->
[CAAnimationID]:{{site.url}}/assets/pushAnimation/CA动画切换.gif "CA动画切换"




参考-[http://www.raywenderlich.com/86521/how-to-make-a-view-controller-transition-animation-like-in-the-ping-app](http://www.raywenderlich.com/86521/how-to-make-a-view-controller-transition-animation-like-in-the-ping-app)

