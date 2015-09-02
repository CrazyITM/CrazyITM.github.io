---
layout: post
category: Animation
tagline: "Supporting tagline"
tags: [UICollecitonView Animation]
---


[^1]ios6的介绍中,UICollectionView闪亮登场,他不仅沿袭了tableView的优秀设计并且在几个基本方面做了延伸.其中,UICollectionView比UITableView更强大的部分是它完全灵活的布局设计(layout).本文中我们将要完成一个相当复杂的自定义的UICollectionView layout 并且通过这种方式讨论这个类的设计.

本文的例子在[这里][1].

## Layout Objects - layout 对象

**UITableView** 和** UICollectionView** 都是 数据源和代理驱动的,他们对自己承载的数据一无所知.

**UICollectionView**对抽象更深一步,他的代理控制子视图的位置大小和单独的布局.通过提供一个自定义的layout对象. 你可以实现你可以想象的几乎所有的布局. 布局继承自抽象基类UICollectionViewLayout。iOS 6有一个具体的布局实现的形式UICollectionViewFlowLayout类.

**UICollectionViewFlowLayout**类可用于实现标准的网格视图,这可能是最常见的用例集合视图。苹果不是名称UICollectionViewGridLayout来命名是很机智的,即使这是我们大多数人如何看待它。flow layout ,更通用一点,更好的描述了类的功能:它的布局方式是一个接一个的cell,当需要时,会插入新的行或列。通过定制滚动的方向、大小和间距的cell,一个流布局还可以布局cell在一个行或列。事实上,UITableView布局可以被认为是一个特例的流布局

在你考虑编写自己的UICollectionViewLayout子类之前,你应该问问你自己是否你能将脑海里的布局装换为UICollectionViewFlowLayout类。类非常可定制,也可以被子类本身进行进一步定制查看集合视图编程指南中的 [_Knowing When to Subclass the Flow Layout_][2] 的提示。


## Cells and Other Views - collectionView 上的视图

为了适应任意的布局,collectionViews设置了一个类似于tableView 但是比TableView 更加灵活的视图层次.通常情况下,主要的内容被可以任意分组的cell呈现,CollectionView的cell必须是UICollectionViewCell的子类.除了cell之外,collectionView 还管理其他的两种视图supplementary views和decoration views.

**Supplementary views** collection view 里的Supplementary views 就像tableView 的section的headerView 和footerView,用来显示改section的信息.和cell一样,他们的内容被代理所驱动.和tableView中的header和footer不同的是, Supplementary views的数量和布局完全被layout控制.

**Decoration views**  Decoration views纯粹被当做装饰物,他们被layout对象管理和拥有.并且内容也不会从数据源中取得.当一个layout对象指出他需要一个decoration view的时候collectionView会毫不犹豫的创造它并将使用layout对象的属性进行操作.任何对这个视图的自定义都没有默认值[^2].

Supplementary View 和 decoration View 一定要是UICollectionReusableView的子类. 无论哪个View都要在使用前在collectionView上注册.以便能够在对重用池调用出列方法的时候,得到一个实例对象.如果你使用了 Interface Builder 你可以直接吧cell 拖到collectionView 里面来代替注册操作. Supplementary View 和decoration View 在使用UICollectionViewFlowLayout(流布局) 的使用也可以使用这种方法.如果不是UICollectionViewFlowLayout(流布局),就得在代码里面进行注册操作,注册方法为:` registerClass:… orregisterNib:…`.viewdidLoad 中调用他就是一个不错的选择.


………


[http://www.objc.io/issues/3-views/collection-view-layouts/][3]


A custom collection view layout is also a nice step toward a lighter view controlleras your view controller does not contain any layout code. Combine this with a separate datasource class as explained in Chris’ article and the view controller for a collection view will hardly contain any code at all.


[^1]:	Introduced in iOS 6, UICollectionView is the new star among view classes in UIKit. It shares its API design with UITableView but extends the latter in a few fundamental ways. The most powerful feature of UICollectionView and the point where it significantly exceeds UITableView’s capabilities is its completely flexible layout architecture. In this article, we will implement a fairly complex custom collection view layout and discuss important aspects of the class’s design along the way.

[^2]:	感谢 彩色控≡(qq:1686856718)  技术支持

[1]:	https://github.com/objcio/issue-3-collection-view-layouts
[2]:	http://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/UsingtheFlowLayout/UsingtheFlowLayout.html#//apple_ref/doc/uid/TP40012334-CH3-SW4
[3]:	http://www.objc.io/issues/3-views/collection-view-layouts/