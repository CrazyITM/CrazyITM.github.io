---
layout: post
category: Animation
tagline: "Supporting tagline"
tags: [UITableViewCell Animation]
---

参考链接:[http://www.raywenderlich.com/49311/advanced-table-view-animations-tutorial-drop-in-cards][1]

TableViewCell将要出现的时候会调用
-tableView:willDisplayCell:forRowAtIndexPath: 方法
所以我们在该方法中添加并执行动画.
值得注意的是,如果对 tableViewCell 本身做旋转等操作 可能会影响到其他的cell,

> Rotating the actual cell would cause part of it to cover the cell above and below it, which would cause some odd visual effects, such as flickering and clipping of the cell


所以,一般会在cell上添加一个容器视图,对容器视图进行操作.

为了执行过动画的cell 不再执行动画 , 需要记录indexpath 于是新建一个NSmutableSet属性,

	@property (nonatomic,strong) NSMutableSet  * indexSet ;

-  在diplay的代码中作为判断依据 是否执行动画

	-tableView:willDisplayCell:forRowAtIndexPath:
	{// 如果包含indexPath 不再执行
	if([self.indexSet containsObject:indexPath])
	{
	return;
	}
	[self.indexSet addObject:indexPath];
	//取得容器视图
	UIView * view =[cell viewWithTag:1001];
	//设置转换矩阵
	[view .layer setTransform:self.transform];
	//设置成默认的状态
	[UIView animateWithDuration:0.25 animations:^{
	view.layer.transform =CATransform3DIdentity;
	view.layer.opacity = 1;
	}];
	}
	

 其中设置transFrom如下


	-(void)setTransform
	{
	CGFloat rotationAngleDegrees = -15;
	CGFloat rotationAngleRadians = rotationAngleDegrees (M_PI/180); //旋转15度
	CGPoint offsetPositioning = CGPointMake(-20, -20);
	CATransform3D transform =CATransform3DIdentity;
	transform =CATransform3DRotate(transform, rotationAngleRadians,0.0, 0.0, 1.0);
	transform =CATransform3DTranslate(transform,offsetPositioning.x, offsetPositioning.y, 0.0);
	_transform = transform;
	}
	

其中 角度与弧度的转换



	1度=π/180弧度( ≈0.017453弧度 ) 
	一个圆是360度，2π弧度
	例如： 
	90°＝90×π/180 ＝π/2 弧度 
	60°＝60×π/180 ＝π/3 弧度 
	45°＝45×π/180 ＝π/4 弧度 
	30°＝30×π/180 ＝π/6 弧度 
	120°＝120×π/180 ＝2π/3 弧度
	

[1]:	http://www.raywenderlich.com/49311/advanced-table-view-animations-tutorial-drop-in-cards