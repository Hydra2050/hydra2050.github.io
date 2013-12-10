---
layout: post
title: "关于UITableView的一个有趣的问题"
date: 2013-12-10 23:03:49 +0800
comments: true
categories: 
---

##起因

今天碰到了一个很有趣的问题，大概现象是这个样子的：在一个可以动态增加、删除tablviewcell的页面，每个cell上有一个输入框，当离开页面的时候，记录cell个数、输入框内容，下次进入自动填充显示。

发现之前在ios6上运行完全没有问题，但是在ios7上，一旦删除过一条后，退出页面，再进来，第一个行的输入框记录的内容会变成删除的那个cell的输入框内容。

<p style="color:red" > 另外一个值得注意的地方是，cell个数很少，没有超过tableview的高度。</p>

难道又是ios7兼容的问题，原来一直都没有问题啊？

好吧，开始查看代码，单步调试，看看到底是哪一步逻辑出错。

##哪里出了问题

在退出页面，保持内容的时候，大概有这样一段代码逻辑：

	for(UIView* v in self.tableView.subviews)
    {
        if([v isKindOfClass:[UITableViewCell class]])
        {
           	UITextField* textField = cell.myTextField;
           	// 取出textField.text 根据cell的 indexPath值 保存到data model
        }
    }
  
单步调试的时候，发现tableview上竟然存在一个为nil的cell。不错，就是我们删除的那个cell。程序内并没有非空检测，结果取它的的indexPath.row ，得到的值为零。这样就将它的textField内容保存了下来。

##删掉的cell为什么还在

这是该到我们的利器大显身手了——[Reveal](http://revealapp.com/)。
我写了一个demo，最初有6个cell，功能是点击cell，就让cell的个数减一。当我删除五行后，在Reveal中可以看到：

![Reveal截图](./images/tvst.png)

是的，那个cell还在！还在tableview上，只不过被隐藏掉了。
打印tableview上的cell：

	<UITableViewCell: 0x755be20; frame = (0 220; 320 44); text = '5'; hidden = YES; autoresize = W; layer = <CALayer: 0x755bf50>>
	<UITableViewCell: 0x755b680; frame = (0 176; 320 44); text = '4'; hidden = YES; autoresize = W; layer = <CALayer: 0x755b480>>
	<UITableViewCell: 0x755aee0; frame = (0 132; 320 44); text = '3'; hidden = YES; autoresize = W; layer = <CALayer: 0x755ae10>>
	<UITableViewCell: 0x755a870; frame = (0 88; 320 44); text = '2'; hidden = YES; autoresize = W; layer = <CALayer: 0x755a730>>
	<UITableViewCell: 0x7559f60; frame = (0 44; 320 44); text = '1'; hidden = YES; autoresize = W; layer = <CALayer: 0x755a090>>
	<UITableViewCell: 0x7556f70; frame = (0 0; 320 44); text = '0'; hidden = YES; autoresize = W; layer = <CALayer: 0x75570f0>>

可以看到，还是存在6个cell对象。

##好吧，那为什么当初在ios6上没有问题呢

ios6上运行没有问题，不代表代码逻辑没有问题，只不过问题被巧妙的掩藏过去了。

问题的关键在于查找到的顺序：在ios7中，被删除的cell会最后被遍历，会覆盖row为0的内容；而在ios6中，被删除的cell最先被遍历，虽然会先写入错误的内容，但是后面会被真正的第0行内容覆盖，所以才看起来没有问题。

终于，又解决了一个有趣的bug。

##总结


这种遍历的方式本来就有问题，应该根据indexPath，遍历查找cell。另外程序缺少异常情况的处理，cell为空时，没有做异常处理。对UITableView的机制缺乏了解，通过这个bug又有了新的认识。

Reveal乃居家旅行必备神器。

也许很多时候，我们埋下了陷阱，由于某种巧合，表面上安然无事。但是终究有一天会暴漏出来。要不断提高代码质量。

（ps：如果没有埋下这个错误，也许就没有了今天解决问题的乐趣，对这一块还是不了解。就像那句话说的：无知让我感到痛苦，无知为我带来快乐。）

