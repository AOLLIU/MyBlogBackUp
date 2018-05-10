---
title: 3DTouch
date: 2015-11-20 20:35:50
tags:
  - iOS开发
  - OC
---

# 3D Touch的三大模块
- 在我们app中使用3D Touch 功能,主要分为以下3个模块,在下面会一一介绍

## 1.Home Screen Quick Actions
- 通过主屏幕的icon,我们可以使用3D Touch呼出一个菜单,进行快速定位应用功能模块
- iOS9为我们提供了两种屏幕标签,静态标签和动态标签
 - 静态标签:静态标签是我们在项目的配置plist文件中配置的标签，在用户安装程序后就可以使用，并且排序会在动态标签的前面。
 - 动态标签:动态标签是我们在程序中，通过代码添加的，与之相关的类，主要有三个：
   - UIApplicationShortcutItem 创建3DTouch标签的类
   - UIMutableApplicationShortcutItem 创建可变的3DTouch标签的类
   - UIApplicationShortcutIcon 创建标签中图片Icon的类
- 具体实现:

<!--more-->

### APPIcon深按弹窗
- 1.appdelegate方法中实现设置和监听

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
//    设置图标长按时,弹出的样式
//    iconWithTypedef: 图标的类型 这里可以跳到头文件中看icon是可以自己通过图片生成的
    UIApplicationShortcutIcon * icon0 = [UIApplicationShortcutIcon iconWithType:UIApplicationShortcutIconTypeAdd];
    UIApplicationShortcutIcon * icon1 = [UIApplicationShortcutIcon iconWithType:UIApplicationShortcutIconTypeLove];
    
    /*
     第一个参数:type,类型,一个字符串,可以随便写,到后面通过监听类型来做对应的事情
     第二个参数:标题
     第三个参数:子标题
     第四个参数:就是图标,可以自定义,也可以使用系统的
     第五个参数:就是想传出去的信息
     */
    UIApplicationShortcutItem * item0 = [[UIApplicationShortcutItem alloc]initWithType:@"type0" localizedTitle:@"添加" localizedSubtitle:@"真的添加" icon:icon0 userInfo:@{@"wenxiaoli" : @"你叫什么名字"}];
    
    UIApplicationShortcutItem *item1 = [[UIApplicationShortcutItem alloc]initWithType:@"type2" localizedTitle:@"删除" localizedSubtitle:@"真的删除" icon:icon1 userInfo:@{@"hahaha" : @"管你什么事"}];
    
    application.shortcutItems = @[item0,item1];
    
    return YES;
}


//监听弹窗的点击
- (void)application:(UIApplication *)application performActionForShortcutItem:(UIApplicationShortcutItem *)shortcutItem completionHandler:(void (^)(BOOL))completionHandler{
    
    if ([shortcutItem.type isEqualToString:@"type1"]) {
        NSLog(@"%@ ",shortcutItem.userInfo[@"liuwei"]);
    }else{
    
        NSLog(@"%@ ",shortcutItem.userInfo[@"liuwei"]);
    }
}
```

## peak and pop
- 这个功能是一套全新的用户交互机制,在使用会有3个交互阶段
 - 1.提示用户这里用3D Touch的交互,会使交互控件周围模糊
 - 2.继续深按,会出现预览试图
 - 3.通过试图上的交互控件进一步交互
 
### 1.判断当前设备是否支持3DTouch,注册3DTouch

```objc
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{

    static NSString * cellID = @"cell";
    
    LVHeroCell * cell = [tableView dequeueReusableCellWithIdentifier:cellID];
    
//    1.判断是否支持3DTouch
    if (self.traitCollection.forceTouchCapability == UIForceTouchCapabilityAvailable) {
//        注册cell,支持3DTouch,并设置预览代理
        [self registerForPreviewingWithDelegate:self sourceView:cell];
    }
    cell.heroModel = self.heroes[indexPath.row];

    return cell;
}
```

### 2.遵守协议实现代理方法 

```objc

#pragma mark - 预览代理
//轻按,中按时调用
- (UIViewController *)previewingContext:(id<UIViewControllerPreviewing>)previewingContext viewControllerForLocation:(CGPoint)location{

//    1.获取sourceView
    LVHeroCell * sourceView = (LVHeroCell *)[previewingContext sourceView];
    
//    2.设置弹出的位置
    [previewingContext setSourceRect:sourceView.bounds];
    
//    3.设置弹框的view
    LVDetailViewController *detailVC = [[LVDetailViewController alloc]init];
    detailVC.preferredContentSize = CGSizeMake(0, 500);
    detailVC.heroModel = sourceView.heroModel;
    __weak typeof(detailVC) weakDetailVc = detailVC;
    detailVC.toDetailVcBlock = ^(){
        [self.navigationController pushViewController:weakDetailVc animated:YES];
    };
    return detailVC;
}

//弹窗出现后继续重按时调用
- (void)previewingContext:(id<UIViewControllerPreviewing>)previewingContext commitViewController:(UIViewController *)viewControllerToCommit{

    [self showViewController:viewControllerToCommit sender:self];
}

```
### 3.设置控制器在弹窗时候,下面输出的数组,在需要弹出的控制器中

```objc
//设置控制器在弹窗的情况,下面输出的数组
- (NSArray<id<UIPreviewActionItem>> *)previewActionItems{

    UIPreviewAction *action1 = [UIPreviewAction actionWithTitle:@"进入" style:UIPreviewActionStyleDefault handler:^(UIPreviewAction * _Nonnull action, UIViewController * _Nonnull previewViewController) {
        
//        因为当前控制器不能自己push自己,那么就得是他的父控制器知道什么时候push他,给她定义一个block属性,在父控制器中拿到block属性设置,在这里调用block
        LVDetailViewController *detailVc = (LVDetailViewController *)previewViewController;
        detailVc.toDetailVcBlock();
//        
//        LVDetailViewController *vc = [[LVDetailViewController alloc]init];
//        [self presentViewController:vc animated:true completion:nil];
        
    }];
    UIPreviewAction *action2 = [UIPreviewAction actionWithTitle:@"取消" style:UIPreviewActionStyleDestructive handler:^(UIPreviewAction * _Nonnull action, UIViewController * _Nonnull previewViewController) {
        NSLog(@"取消");
    }];

    
    return @[action1,action2];

}
```

## Force properties :新的交互参数:力度,通过检测力度值,来做对应的交互处理

- 暂未总结

