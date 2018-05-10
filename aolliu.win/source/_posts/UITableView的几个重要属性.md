---
title: UITableView的几个重要属性
date: 2014-9-4 10:56:34
tags:
    - iOS开发
    - OC
---

**一.什么是tableView的内容(content)?**
     内容包括以下3个部分:除这三个之外都不是tableview的内容,例如给tableview添加个子控件UIView,这个view就不是tableview的内容
 - 1.Cell
 - 2.tableHeaderView\tableFooterView
 - 3.sectionHeader\sectionFooter

**二.contentSize.height**
 内容的高度

<!--more-->

**三.contentOffset.y**
 内容的偏移量(frame的顶部 - content的顶部)

**四.contentInset**
 内容周围的间距(内边距)
*注意*:是内容四周的间距

**五.Frame**
 - 1.frame.size.height :  矩形框的高度
 - 2.frame : 以父控件内容的左上角为坐标原点
注意:子控件添加都是添加到父控件的内容上

**具体情况分析如下:**
- 什么也没有的情况
![](http://upload-images.jianshu.io/upload_images/1494773-5338aa63d22bbe09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 只有headerView和footerView的情况
![](http://upload-images.jianshu.io/upload_images/1494773-22742912fcd6edfa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 只有cell的情况
![](http://upload-images.jianshu.io/upload_images/1494773-a127382cd07f4c57.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 有cell和contentinset的情况
![](http://upload-images.jianshu.io/upload_images/1494773-7d7f53548169090a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 有cell和headerView和footerView情况
![](http://upload-images.jianshu.io/upload_images/1494773-5426fffe8715b584.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 有cell和headerView和footerView也有contentinset
![](http://upload-images.jianshu.io/upload_images/1494773-424cabf47c4a7d92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

***下面将会引入子控件,注意子控件是相对于父控件的内容来添加的***
- 有cell,有content inset,有子控件
![](http://upload-images.jianshu.io/upload_images/1494773-dc6f66d042070eb9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 有cell,有footerView/headerview,有子控件
![](http://upload-images.jianshu.io/upload_images/1494773-163ca951b3829cc6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 有cell,有footerView/headview,有contentinset,有子控件
![](http://upload-images.jianshu.io/upload_images/1494773-c1de373d25dd8537.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
