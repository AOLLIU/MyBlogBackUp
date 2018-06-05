---
title: Masonry 源码学习
date: 2017-03-04 16:37:21
tags:
    - iOS开发
    - OC
---


## 1.介绍:

masonry 是用于自动布局的第三方框架

```
[_imageView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.centerX.equalTo(self.view);
	make.top.equalTo(self.view).offset(20);
	make.width.equalTo(@150);
	make.height.equalTo(@150);
}];
```

## 2.源码解析:

### 1.)`mas_makeConstraints: `
	masonry使用分类的方式为 UIKit 添加一个方法`mas_makeConstraint`, 这个方法接受了一个 block, 这个 block 有一个`MASConstraintMaker`类型的参数,

```
//添加约束
- (NSArray *)mas_makeConstraints:(void(^)(MASConstraintMaker *make))block;
//更新约束
- (NSArray *)mas_updateConstraints:(void(^)(MASConstraintMaker *make))block;
//重置约束
- (NSArray *)mas_remakeConstraints:(void(^)(MASConstraintMaker *make))block;

```
<!--more-->

### 2.)Constraint Maker Block

以`mas_makeConstraints`为例分析Masonry内部是如何为视图添加约束的,方法的实现如下:

```
- (NSArray *)mas_makeConstraints:(void(^)(MASConstraintMaker *))block {
    self.translatesAutoresizingMaskIntoConstraints = NO;
    MASConstraintMaker *constraintMaker = [[MASConstraintMaker alloc] initWithView:self];
    block(constraintMaker);
    return [constraintMaker install];
}

```

### 分析:

- 1.Masonry 是封装的苹果的 AutoLayout 框架, 我们要在为视图添加约束前将`translatesAutoresizingMaskIntoConstraints`设置为NO. 如果这个属性没有设置为NO, 那么约束不会被成功添加.
- 2.初始化一个MASConstraintMaker
- 3.将maker传入block作为属性
- 4.调用maker的install方法为视图添加约束


## 3.) MASConstraintMaker

```
- (id)initWithView:(MAS_VIEW *)view {
    self = [super init];
    if (!self) return nil;
    
    self.view = view;
    self.constraints = NSMutableArray.new;
    
    return self;
}
```
#### 分析:
- 1.在初始化`MASConstraintMaker`的实例时, 它会持有一个对应 view 的弱引用, 并初始化一个`constraints`的空可变数组用来保存之后配置属性时持有所有的约束.
- 2. maker持有一个空可变数组用来之后配置属性时持有所有的约束,maker提供了工厂方法创建MASConstraint.所有的约束(MASConstraint)都会被保存到约束数组中,最后install时将约束添加到视图上.


## 4.) Setup MASConstraintMaker

调用`block(constraintMaker)`时, 是对`constraintMaker`进行配置.

```
	make.centerX.equalTo(self.view);
	make.top.equalTo(self.view).offset(20);
	make.width.equalTo(@150);
	make.height.equalTo(@150);
```

### make.left

访问make的left,right,width,height等属性时, 会调用`constraint:addConstraintWithLayoutAttribute:`方法.

```
// MASViewConstraint.m

- (MASConstraint *)left {
    return [self addConstraintWithLayoutAttribute:NSLayoutAttributeLeft];
}

- (MASConstraint *)addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
    return [self constraint:nil addConstraintWithLayoutAttribute:layoutAttribute];
}

- (MASConstraint *)constraint:(MASConstraint *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
    MASViewAttribute *viewAttribute = [[MASViewAttribute alloc] initWithView:self.view layoutAttribute:layoutAttribute];
    MASViewConstraint *newConstraint = [[MASViewConstraint alloc] initWithFirstViewAttribute:viewAttribute];
    if ([constraint isKindOfClass:MASViewConstraint.class]) { ... }
    if (!constraint) {
        newConstraint.delegate = self;
        [self.constraints addObject:newConstraint];
    }
    return newConstraint;
}

```


#### 分析:
	
- 上列方法最终会到`constraint:addConstraintWithLayoutAttribute:`这一方法,因为在这个类中传入该方法的第一个参数一直为nil,所以省略的代码不会执行.这部分代码会先以布局属性 left 和视图本身初始化一个 MASViewAttribute 的实例,之后使用 MASViewAttribute 的实例初始化一个 constraint 并设置它的代理,加入数组,然后返回.这是我们输入 make.left 进行的全部工作, 它会返回一个 MASConstraint, 用于之后的继续配置.


### make.left.equalTo(@20)

在 make.left 返回 MASConstraint 之后, 我们会继续在这个链式的语法中调用下一个方法来指定约束的关系.

```
- (MASConstraint * (^)(id attr))equalTo;
- (MASConstraint * (^)(id attr))greaterThanOrEqualTo;
- (MASConstraint * (^)(id attr))lessThanOrEqualTo;

```
### 分析:

- 这三个方法是在 MASViewConstraint 的父类, MASConstraint 中定义的.MASConstraint 是一个抽象类, 很多的方法都必须在子类中覆写的. Masonry 中有两个 MASConstraint 的子类,分别是 MASViewConstraint 和 MASCompositeConstraint. 后者是一些约束的集合. 

先来看一下这三个方法是怎么实现的:

```
// MASConstraint.m

- (MASConstraint * (^)(id))equalTo {
    return ^id(id attribute) {
        return self.equalToWithRelation(attribute, NSLayoutRelationEqual);
    };
}

```
	该方法会执行`self.equalToWithRelation`, 这个方法是定义在子类中的, 父类没有提供这个方法的具体实现.

```
- (MASConstraint * (^)(id, NSLayoutRelation))equalToWithRelation { MASMethodNotImplemented(); }
```

因为我们为 equalTo 提供了参数 attribute 和布局关系 NSLayoutRelationEqual, 这两个参数会传递到 equalToWithRelation中,设置 constraint 的布局关系和 secondViewAttribute 属性, 为 maker 的 install 做准备.


```
// MASViewConstraint.m

- (MASConstraint * (^)(id, NSLayoutRelation))equalToWithRelation {
    return ^id(id attribute, NSLayoutRelation relation) {
        if ([attribute isKindOfClass:NSArray.class]) { ... } 
        else {
            ...
            self.layoutRelation = relation;
            self.secondViewAttribute = attribute;
            return self;
        }
    };
}

```

	注意:`setSecondViewAttribute:`方法, 它并不只是一个简单的 setter 方法, 它会根据你传入的值的种类赋值

```
// MASConstraintMaker.m

- (void)setSecondViewAttribute:(id)secondViewAttribute {
    if ([secondViewAttribute isKindOfClass:NSValue.class]) {
        [self setLayoutConstantWithValue:secondViewAttribute];
    } else if ([secondViewAttribute isKindOfClass:MAS_VIEW.class]) {
        _secondViewAttribute = [[MASViewAttribute alloc] initWithView:secondViewAttribute layoutAttribute:self.firstViewAttribute.layoutAttribute];
    } else if ([secondViewAttribute isKindOfClass:MASViewAttribute.class]) {
        _secondViewAttribute = secondViewAttribute;
    } else {
        NSAssert(NO, @"attempting to add unsupported attribute: %@", secondViewAttribute);
    }
}

```

- 第一种情况对应的就是:
	`make.left.equalTo(@40);`传入 NSValue 的时,会直接设置 constraint 的 offset, centerOffset, sizeOffset, 或者 insets

- 第二种情况一般会直接传入一个视图:
	`make.left.equalTo(view);`这时,就会初始化一个 layoutAttribute 属性与 firstViewArribute 相同的 MASViewAttribute, 上面的代码就会使视图与 view 左对齐.

- 第三种情况会传入一个视图的MASViewAttribute:
	`make.left.equalTo(view.mas_right);`使用这种方法时,一般是因为约束的方向不同,这行代码会使视图的左侧与view的右侧对其

	上面我们完成了对一个约束的配置,然后可以使用相同的语法完成对一个视图上所有约束进行配置, 最后到了install方法


### 5.)Install MASConstraintMaker

在`mas_makeConstraints:`方法的最后会调用`[constraintMaker install]`方法设置所有存储在 self.constraints 数组中的所有约束.


```
- (NSArray *)install {
    if (self.removeExisting) {
        NSArray *installedConstraints = [MASViewConstraint installedConstraintsForView:self.view];
        for (MASConstraint *constraint in installedConstraints) {
            [constraint uninstall];
        }
    }
    NSArray *constraints = self.constraints.copy;
    for (MASConstraint *constraint in constraints) {
        constraint.updateExisting = self.updateExisting;
        [constraint install];
    }
    [self.constraints removeAllObjects];
    return constraints;
}
```


这个方法会先判断当前的视图的约束是否应该要被 uninstall,如果我们调用 mas_remakeConstraints: 方法, 视图中原啦的约束就会全部被 uninstall.

```
- (NSArray *)mas_remakeConstraints:(void(^)(MASConstraintMaker *make))block {
    self.translatesAutoresizingMaskIntoConstraints = NO;
    MASConstraintMaker *constraintMaker = [[MASConstraintMaker alloc] initWithView:self];
    constraintMaker.removeExisting = YES;
    block(constraintMaker);
    return [constraintMaker install];
}
```

然后遍历 constraints 数组, 调用 install 方法.


### MASViewConstraint install

```
- (void)install {
    if (self.hasBeenInstalled) {
        return;
    }
    
    MAS_VIEW *firstLayoutItem = self.firstViewAttribute.item;
    NSLayoutAttribute firstLayoutAttribute = self.firstViewAttribute.layoutAttribute;
    MAS_VIEW *secondLayoutItem = self.secondViewAttribute.item;
    NSLayoutAttribute secondLayoutAttribute = self.secondViewAttribute.layoutAttribute;

    // alignment attributes must have a secondViewAttribute
    // therefore we assume that is refering to superview
    // eg make.left.equalTo(@10)
    if (!self.firstViewAttribute.isSizeAttribute && !self.secondViewAttribute) {
        secondLayoutItem = self.firstViewAttribute.view.superview;
        secondLayoutAttribute = firstLayoutAttribute;
    }
    
    MASLayoutConstraint *layoutConstraint
        = [MASLayoutConstraint constraintWithItem:firstLayoutItem
                                        attribute:firstLayoutAttribute
                                        relatedBy:self.layoutRelation
                                           toItem:secondLayoutItem
                                        attribute:secondLayoutAttribute
                                       multiplier:self.layoutMultiplier
                                         constant:self.layoutConstant];
    
    layoutConstraint.priority = self.layoutPriority;
    layoutConstraint.mas_key = self.mas_key;
    
    if (self.secondViewAttribute.view) {
        MAS_VIEW *closestCommonSuperview = [self.firstViewAttribute.view mas_closestCommonSuperview:self.secondViewAttribute.view];
        NSAssert(closestCommonSuperview,
                 @"couldn't find a common superview for %@ and %@",
                 self.firstViewAttribute.view, self.secondViewAttribute.view);
        self.installedView = closestCommonSuperview;
    } else if (self.firstViewAttribute.isSizeAttribute) {
        self.installedView = self.firstViewAttribute.view;
    } else {
        self.installedView = self.firstViewAttribute.view.superview;
    }


    MASLayoutConstraint *existingConstraint = nil;
    if (self.updateExisting) {
        existingConstraint = [self layoutConstraintSimilarTo:layoutConstraint];
    }
    if (existingConstraint) {
        // just update the constant
        existingConstraint.constant = layoutConstraint.constant;
        self.layoutConstraint = existingConstraint;
    } else {
        [self.installedView addConstraint:layoutConstraint];
        self.layoutConstraint = layoutConstraint;
        [firstLayoutItem.mas_installedConstraints addObject:self];
    }
}
```

**方法分析:**
	
MASViewConstraint 的 install 方法是最后为当前视图添加约束方法,首先这个方法会先获取即将用于初始化 NSLayoutConstraint 的子类的几个属性.

```
// MASViewConstraint.m
	// 这里的item == view
    MAS_VIEW *firstLayoutItem = self.firstViewAttribute.item;
    NSLayoutAttribute firstLayoutAttribute = self.firstViewAttribute.layoutAttribute;
    MAS_VIEW *secondLayoutItem = self.secondViewAttribute.item;
    NSLayoutAttribute secondLayoutAttribute = self.secondViewAttribute.layoutAttribute;
```

Masonry 之后会判断当前即将添加的约束是否是 size 类型的约束

```
// MASViewConstraint.m

if (!self.firstViewAttribute.isSizeAttribute && !self.secondViewAttribute) {
   secondLayoutItem = firstLayoutItem.superview;
   secondLayoutAttribute = firstLayoutAttribute;
}

```

如果不是 size 类型并且没有提供第二个 viewAttribute,会自动将约束添加到 superview 上.
`make.left.equalTo(@10);`等价于`make.left.equalTo(superView.mas_left).with.offset(10);`

	然后就会初始化 NSLayoutConstraint 的子类 MASLayoutConstraint:

```
// MASViewConstraint.m

MASLayoutConstraint *layoutConstraint
   = [MASLayoutConstraint constraintWithItem:firstLayoutItem
                                   attribute:firstLayoutAttribute
                                   relatedBy:self.layoutRelation
                                      toItem:secondLayoutItem
                                   attribute:secondLayoutAttribute
                                  multiplier:self.layoutMultiplier
                                    constant:self.layoutConstant];
layoutConstraint.priority = self.layoutPriority;   

```

然后会寻找 firstLayoutItem 和 secondLayoutItem 两个视图的公共 superview

```
// View+MASAdditions.m

- (instancetype)mas_closestCommonSuperview:(MAS_VIEW *)view {
    MAS_VIEW *closestCommonSuperview = nil;

    MAS_VIEW *secondViewSuperview = view;
    while (!closestCommonSuperview && secondViewSuperview) {
        MAS_VIEW *firstViewSuperview = self;
        while (!closestCommonSuperview && firstViewSuperview) {
            if (secondViewSuperview == firstViewSuperview) {
                closestCommonSuperview = secondViewSuperview;
            }
            firstViewSuperview = firstViewSuperview.superview;
        }
        secondViewSuperview = secondViewSuperview.superview;
    }
    return closestCommonSuperview;
}
```

如果update当前的约束就会找到原来的约束,替换为新的约束.

```
- (NSArray *)mas_updateConstraints:(void(^)(MASConstraintMaker *))block {
    self.translatesAutoresizingMaskIntoConstraints = NO;
    MASConstraintMaker *constraintMaker = [[MASConstraintMaker alloc] initWithView:self];
    constraintMaker.updateExisting = YES;
    block(constraintMaker);
    return [constraintMaker install];
}
```

```
// MASViewConstraint.m

MASLayoutConstraint *existingConstraint = nil;
if (self.updateExisting) {
   existingConstraint = [self layoutConstraintSimilarTo:layoutConstraint];
}
if (existingConstraint) {
   // just update the constant
   existingConstraint.constant = layoutConstraint.constant;
   self.layoutConstraint = existingConstraint;
} else {
   [self.installedView addConstraint:layoutConstraint];
   self.layoutConstraint = layoutConstraint;
}
    
[firstLayoutItem.mas_installedConstraints addObject:self];

```

如果原来的 view 中不存在可以update的约束,或者没有调用 mas_updateConstraint: 方法, 那么就会在上一步寻找到的 installedView (公共父view)上面添加约束.

```
	[self.installedView addConstraint:layoutConstraint];

```

### 6.)其他

make.left.equal(view).with.offset(30)

有时候为了使代码更加的易懂.会使与 with , and等连接语句,这两个方法都会直接返回 MASConstraint, 方法本身不做任何的修改.

而 offset 方法其实是修改 layoutConstraint 中的常量, self.layoutConstant 在初始化时会被设置为 0, 我们可以通过修改 offset 属性来改变它.

```
// MASViewConstraint.m

- (void)setOffset:(CGFloat)offset {
    self.layoutConstant = offset;
}

```


## MASCompositeConstraint

MASCompositeConstraint 是一些 MASConstraint 的集合,它可以同时为一个视图来添加多个约束.通过 make 直接调用 edges size center 时,会产生一个 MASCompositeConstraint 的实例, 这个实例会初始化所有对应的单独的约束.

```
// MASConstraintMaker.m

- (MASConstraint *)edges {
    return [self addConstraintWithAttributes:MASAttributeTop | MASAttributeLeft | MASAttributeRight | MASAttributeBottom];
}

- (MASConstraint *)size {
    return [self addConstraintWithAttributes:MASAttributeWidth | MASAttributeHeight];
}

- (MASConstraint *)center {
    return [self addConstraintWithAttributes:MASAttributeCenterX | MASAttributeCenterY];
}

```

这些属性都会调用 addConstraintWithAttributes: 方法,生成多个属于 MASCompositeConstraint 的实例.

```
// MASConstraintMaker.m

NSMutableArray *children = [NSMutableArray arrayWithCapacity:attributes.count];
    
for (MASViewAttribute *a in attributes) {
   [children addObject:[[MASViewConstraint alloc] initWithFirstViewAttribute:a]];
}
    
MASCompositeConstraint *constraint = [[MASCompositeConstraint alloc] initWithChildren:children];
constraint.delegate = self;
[self.constraints addObject:constraint];
return constraint;

```

## mas_equalTo

这个宏将 C 和 Objective-C语言中的一些基本数据结构比如说 double CGPointCGSize 这些值用 NSValue 进行包装.


## 总结:
Masonry 是如何为视图添加约束的?

- Masonry使用分类的方式为 UIKit 添加一个方法 mas_makeConstraint,这个方法接受了一个 block, 这个 block 有一个 MASConstraintMaker 类型的参数,这个maker会持有一个约束的数组, 这里保存着所有将被加入到视图中的约束.

- 我们通过链式的语法配置 maker, 设置它的 left right 等属性, 比如说 make.left.equalTo(view),其实这个 left equalTo还有像 with offset 之类的方法都会返回一个 MASConstraint 的实例, 所以在这里才可以用链式的语法.

- 在配置结束后, 首先会调用 maker 的 install 方法, 而这个 maker的 install 方法会遍历其持有的约束数组, 对其中的每一个约束发送 install 消息. 在这里就会使用到在上一步中配置的属性,初始化 NSLayoutConstraint 的子类 MASLayoutConstraint并添加到合适的视图上.




