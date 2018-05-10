---
title: iOS Widget开发
date: 2018-02-27 15:50:46
tags:
    - iOS开发
    - OC
---


## widget的尺寸:
- 1.widget有两种状态:折叠和展开,不同状态下最大宽高不同
```
typedef NS_ENUM(NSInteger, NCWidgetDisplayMode) {
    NCWidgetDisplayModeCompact, // Fixed height
    NCWidgetDisplayModeExpanded, // Variable height
} NS_ENUM_AVAILABLE_IOS(10_0);
```
<!--more-->

- 2.如果你想改widget的高度，都是在下面这个方法中这么写
```
- (void)widgetActiveDisplayModeDidChange:(NCWidgetDisplayMode)activeDisplayMode withMaximumSize:(CGSize)maxSize {
    //这个意思就是折叠状态110pt，展开状态300pt。如果你折叠状态就算写120，也一样是110的高度。展开状态下，当然要取比maxSize.height小的一个值。那么maxSize这个值是多少？
    if (activeDisplayMode == NCWidgetDisplayModeCompact) {
        self.preferredContentSize = CGSizeMake(maxSize.width, 110);
    } else {
        self.preferredContentSize = CGSizeMake(maxSize.width, 300);
    }
}
```
- 3.maxSize的大小不是一个固定值,影响因素主要有:设备,系统,系统字体大小,plus上海域横屏竖屏有关.
   - 备注:
       - 1.系统字体可以在设置→通用→辅助功能→更大字体中选择,如果不开启辅助功能中的更大字体,系统字体大小一共有7档(见图统计)。如果开启辅助功能中的更大字体系统字体大小一共有11档,折叠的高度分别是这11档次(没有统计,如果需要自己另行统计)
       - 2.系统默认字体为第4档
       - 3.下图中所有单位为pt
       - 4.所有app的widget都有或多或少的展示问题.(一方面是设计问题没有考虑所有情况,另一方面苹果这个模块做的真是...)

![](http://upload-images.jianshu.io/upload_images/1494773-0961cfb78dcf035f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/1494773-1d669b7a932cb93f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 设计分析


## 好奇心(常规)

- **未展开状态**显示内容为[最新指定一篇 + 发布时间更新],文章显示形式为列表式.
- **展开状态**显示3篇内容
   - "看图:过去24小时发生了什么"为特殊文章.
     - 未展开状态显示内容为主标题和文章内容前3张新闻图
     - 展开状态重排版以模拟"banner"的形式展现,"banner"显示"你需要知道的"第一条新闻内容中的配图以及小标题.并且显示图片数量与当前页码(如果"你需要知道的"新闻总数量为5条,则图片数量显示5)
- 底部3个按钮为了方便用户进入app设置的3个快速入口

**普通折叠和展开**
![普通折叠和展开](http://upload-images.jianshu.io/upload_images/1494773-9c848db28319f4e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**看图折叠和展**
![看图折叠和展开](http://upload-images.jianshu.io/upload_images/1494773-2363ebbe5d2466c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 好奇心研究所:

- **未展开状态**: 显示研究所最新更新的文章标题,以及显示参与人数或者最新文章标识
  - 研究所最新文章标题数大于20个中文字符(超出一行),不显示剩余字符.
  - 研究所更新频率为每天两篇,上下午各一篇
  
- **展开状态重新排版**
  - 根据研究所两种互动方式进行排版
    - 第一种[我说],显示热门评论用户头像及内容,显示数量为3条.底部两个按钮:换一批,说点什么
    - 第二种[投票],显示选项配图,标题内容,和投票最多的选项内容及gif图.底部两个按钮:投它一票,其他选项
  - 交互: 底部两个按钮,无论点击哪一个都跳转app内的详情页.

**我说折叠和展开**
![我说折叠和展开](http://upload-images.jianshu.io/upload_images/1494773-22a11ae35f276abe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**投票折叠和展开**
![投票折叠和展开](http://upload-images.jianshu.io/upload_images/1494773-92bd10d578e1135d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




