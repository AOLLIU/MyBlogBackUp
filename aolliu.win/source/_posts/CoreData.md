---
title: CoreData
date: 2015-3-20 21:20:35
tags:
    - iOS开发
    - OC
---

# Core Data
- 是对SQLite的封装,会自动生成SQLite语句,性能不是太好

### Core Data中3个重要的组成部分
- NSManagerObject:实体对象(1个类对应一张表,1个对象对应表中的1条记录)
- NSPersistenStoreCoordinator:存储器,决定了你的数据存储在什么地方(SQLite/XML/其他文件)
- NSManageObjectContext:操作数据库

<!--more-->

### Core Data的基本使用
- 1.新建一个项目,创建一个data model,并且在model中添加字段

![Snip20160623_4.png](http://upload-images.jianshu.io/upload_images/1494773-22c7af6bcda9c8d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 2.再创建一个NSManagerObject subclass

![Snip20160623_5.png](http://upload-images.jianshu.io/upload_images/1494773-ba19d7e8e076f80a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 3.在storyboard中托四个按钮分别做增删改查
- 4.导入头文件

```objc
#import <CoreData/CoreData.h>
#import "Entity.h"
```
- 5.在控制器中viewdidload方法中
 - 1.创建上下文对象,用来操作数据库 NSManageObjectContext
 - 2.创建模型对象 NSManageObjectModel
 - 3.初始化一个持久化对象,需要表明数据持久化到哪里,指出数据库的路径,告诉数据库存放到那个位置 NSPersistenStoreCoordinator
 - 4.将数据持久化
 - 5.将上下文保存
 
```objc
@interface ViewController ()
{
    NSManagedObjectContext *_context;
}
@end

@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    // 创建上下文对象
    _context = [[NSManagedObjectContext alloc]init];
    
    // 创建模型对象
    NSManagedObjectModel *model = [NSManagedObjectModel mergedModelFromBundles:nil];
    
   // 初始化一个持久化对象
    NSPersistentStoreCoordinator *storeCoor = [[NSPersistentStoreCoordinator alloc]initWithManagedObjectModel:model];
    // 需要表明数据持久化到哪里
    NSError *error = nil;
    
    // 文档路径
    NSString *doc = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
    
    // 数据库的路径
    NSString *path = [doc stringByAppendingPathComponent:@"wenxiaoli.sqlite"];
    
    NSLog(@"path = %@" , path);
    
    // url 告诉数据库存放到哪个位置
    [storeCoor addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:[NSURL fileURLWithPath:path] options:nil error:&error];
    if (error) {
        NSLog(@"数据库持久化失败");
    }
    
    // 将数据持久化
    _context.persistentStoreCoordinator = storeCoor;
    
    NSError *saveError = nil;
    // 将上下文保存
    [_context save:&saveError];
    if (saveError) {
        NSLog(@"上下文保存失败");
    }
    
}
```

 - 6增.
 
```objc
// 增
- (IBAction)addMember:(id)sender {
    for (int i = 0 ; i < 15; i ++) {
        Entity *entity = [NSEntityDescription insertNewObjectForEntityForName:@"Entity" inManagedObjectContext:_context];
        entity.name = [NSString stringWithFormat:@"zhangsan%d",i];
        entity.height = @(178 + i);
        entity.age = [NSDate date];
        [_context save:nil];
    }
}
```
- 7.查
 - 1.获取数据对象
 - 2.添加过滤条件
 - 3.排序
 - 4.查询结果
 - 5.打印结果

```objc
// 查
- (IBAction)searchMember:(id)sender {
    // 获取数据对象   获取到Entity里面的数据
    NSFetchRequest *request = [NSFetchRequest fetchRequestWithEntityName:@"Entity"];
    // 过滤条件\
        注意点  zhangsan要写在外面
    // 过滤查询
    // request.predicate = [NSPredicate predicateWithFormat:@"name ==%@",@"zhangsan"];
    
    // 模糊查询
//    request.predicate = [NSPredicate predicateWithFormat:@"name BEGINSWITH%@",@"lisi"];
//    request.predicate = [NSPredicate predicateWithFormat:@"name ENDSWITH%@",@"2"];
    request.predicate = [NSPredicate predicateWithFormat:@"name CONTAINS%@",@"s"];

    
    // 排序  YES 是升序
    NSSortDescriptor *sort = [NSSortDescriptor sortDescriptorWithKey:@"height" ascending:YES];
    request.sortDescriptors = @[sort];
    
    // 查询结果\
        数组里面存放都是Entity这个类型的对象
    NSArray *arr = [_context executeFetchRequest:request error:nil];
    
    for (Entity *e in arr) {
        NSLog(@"%@ %@",e.name,e.height);
    }
    
    
}

```

- 8.删,先查后删

```objc
// 删
- (IBAction)deleteMember:(id)sender {
    // 获取数据对象   获取到Entity里面的数据
    NSFetchRequest *request = [NSFetchRequest fetchRequestWithEntityName:@"Entity"];
    // 过滤条件\
    注意点  zhangsan要写在外面
    request.predicate = [NSPredicate predicateWithFormat:@"name ==%@",@"zhangsan"];
    
    // 排序  YES 是升序
    NSSortDescriptor *sort = [NSSortDescriptor sortDescriptorWithKey:@"height" ascending:YES];
    request.sortDescriptors = @[sort];
    
    // 查询结果\
    数组里面存放都是Entity这个类型的对象
    NSArray *arr = [_context executeFetchRequest:request error:nil];
    
    for (Entity *e in arr) {
        [_context deleteObject:e];
    }
}
```
- 8.改(先查后改)

```objc
// 改(update)
- (IBAction)updateMember:(id)sender {
    // 获取数据对象   获取到Entity里面的数据
    NSFetchRequest *request = [NSFetchRequest fetchRequestWithEntityName:@"Entity"];
    // 过滤条件\
    注意点  zhangsan要写在外面
    request.predicate = [NSPredicate predicateWithFormat:@"name ==%@",@"zhangsan2"];
    
    // 排序  YES 是升序
    NSSortDescriptor *sort = [NSSortDescriptor sortDescriptorWithKey:@"height" ascending:YES];
    request.sortDescriptors = @[sort];
    
    // 查询结果\
    数组里面存放都是Entity这个类型的对象
    NSArray *arr = [_context executeFetchRequest:request error:nil];
    
    for (Entity *e in arr) {
        e.name = @"lisi2";
        e.name = @"lisi2";
        
        // 保存更新
        [_context save:nil];
    }
}
```