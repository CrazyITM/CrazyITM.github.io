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

[1]:	http://www.raywenderlich.com/49311/advanced-table-view-animations-tutorial-drop-in-cards