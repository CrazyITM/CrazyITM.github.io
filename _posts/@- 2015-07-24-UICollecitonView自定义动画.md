---
layout: post
category: Animation
tagline: "Supporting tagline"
tags: [UICollecitonView Animation]
---


[^1]ios6的介绍中,UICollectionView闪亮登场,他不仅沿袭了tableView的优秀设计并且在几个基本方面做了延伸.其中,UICollectionView比UITableView更强大的部分是它完全灵活的布局设计(layout).本文中我们将要完成一个相当复杂的自定义的UICollectionView layout 并且通过这种方式讨论这个类的设计.

本文的例子在[这里][1].

## Layout Objects

[^2]_UITableView_ 和 _UICollectionView_ 都是 数据源和代理驱动的,他们对自己承载的数据一无所知.

[^3]_UICollectionView_对抽象更深一步,他的代理控制子视图的位置大小和单独的布局.通过提供一个自定义的layout对象. 你可以实现你可以想象的几乎所有的布局. 布局继承自抽象基类UICollectionViewLayout。iOS 6有一个具体的布局实现的形式UICollectionViewFlowLayout类.

[^4]_UICollectionViewFlowLayout_类可用于实现标准的网格视图,这可能是最常见的用例集合视图。苹果不是名称UICollectionViewGridLayout来命名是很机智的,即使这是我们大多数人如何看待它。flow layout ,更通用一点,更好的描述了类的功能:它构建布局cell接着cell,当需要时插入行或列。通过定制滚动的方向、大小和间距的cell,一个流布局还可以布局cell在一个行或列。事实上,UITableView布局可以被认为是一个特例的流布局

[^5]在你考虑编写自己的UICollectionViewLayout子类之前,你应该问问你自己是否你能将脑海里的布局装换为UICollectionViewFlowLayout类。类非常可定制,也可以被子类本身进行进一步定制查看集合视图编程指南中的 [_Knowing When to Subclass the Flow Layout_][2] 的提示。


## Cells and Other Views

[^6]为了适应任意的布局,collectionViews设置了一个类似于tableView 但是比TableView 更加灵活的视图层次.通常情况下,主要的内容被可以任意分组的cell呈现,CollectionView的cell必须是UICollectionViewCell的子类.除了cell之外,collectionView 还管理其他的两种视图supplementary views和decoration views.

**Supplementary views** [^7]collection view 里的Supplementary views 就像tableView 的section的headerView 和footerView,用来显示改sectio的信息.和cell一样,他们的内容被代理所驱动.和tableView中的header和footer不同的是, Supplementary views的数量和布局完全被layout控制.

**Decoration views**  [^8]纯粹被当做装饰物,他们被layout对象管理和拥有.并且内容也不会从数据源中取得.当一个layout对象指出他需要一个decoration view的时候collectionView会毫不犹豫的创造它并将使用layout对象的属性进行操作.针对这个view内容的自定义都哈哈哈不会翻译.(Any customization of the view’s contents is not intended);

[^1]:	Introduced in iOS 6, UICollectionView is the new star among view classes in UIKit. It shares its API design with UITableView but extends the latter in a few fundamental ways. The most powerful feature of UICollectionView and the point where it significantly exceeds UITableView’s capabilities is its completely flexible layout architecture. In this article, we will implement a fairly complex custom collection view layout and discuss important aspects of the class’s design along the way.

[^2]:	Both UITableView and UICollectionView are data-source- and delegate-driven. They act as dumb containers for the collection of subviews they are displaying, knowing nothing about their actual contents.

[^3]:		UICollectionView takes the abstraction one step further. It delegates the control over its subviews’ positions, sizes, and appearances to a separate layout object. By providing a custom layout object, you can achieve pretty much any layout you can imagine. Layouts inherit from the abstract base classUICollectionViewLayout. iOS 6 comes with one concrete layout implementation in the form of the UICollectionViewFlowLayout class.

[^4]:	A flow layout can be used to implement a standard grid view, which is probably the most common use case for a collection view. Apple was smart enough to not actually name the class UICollectionViewGridLayout, even if that is how most of us think about it. The more generic term, flow layout, describes the class’s capabilities much better: it builds its layout by placing cell after cell, inserting line or column breaks when needed. By customizing the scroll direction, sizing, and spacing of the cells, a flow layout can also layout cells in a single row or column. In fact, UITableView’s layout can be thought of as a special case of flow layout.

[^5]:	Before you consider writing your own UICollectionViewLayout subclass, you should always ask yourself if you can achieve the layout you have in mind withUICollectionViewFlowLayout. The class is remarkably customizable and can also be subclassed itself for further customization. See Knowing When to Subclass the Flow Layout in the Collection View Programming Guide for tips.

[^6]:	To accommodate arbitrary layouts, collection views set up a view hierarchy that is similar to, but more flexible than, that of a table view. As usual, your main content is displayed in cells, which can optionally be grouped into sections. Collection view cells must be subclasses of UICollectionViewCell. In addition to cells, collection views manage two more kinds of views: supplementary views and decoration views.

[^7]:	Supplementary views in a collection view correspond to a table view’s section header and footer views in that they display information about a section. Like cells, their contents are driven by the data source object. Unlike their usage in table views, however, supplementary views are not bound to being header or footer views; their number and placement are entirely controlled by the layout.

[^8]:	Decoration views act as pure ornamentation. They are owned and managed entirely by the layout object and do not get their contents from the data source. When a layout object specifies that it requires a decoration view, the collection view creates it automatically and applies the layout attributes provided by the layout object. Any customization of the view’s contents is not intended

[1]:	https://github.com/objcio/issue-3-collection-view-layouts
[2]:	http://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/UsingtheFlowLayout/UsingtheFlowLayout.html#//apple_ref/doc/uid/TP40012334-CH3-SW4