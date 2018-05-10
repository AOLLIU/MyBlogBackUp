---
title: UIButton
date: 2014-8-22 19:35:20
tags:
    - iOS开发
    - OC
---

### 说说UIButton的几种常见的状态:

#### 高亮状态(UIControlStateHighlighted):
- 就是用户长按住按钮不松开,或者在代码中设置了按钮的属性highlighted = YES;

**如何知道按钮是不是高亮状态?**
 - 其实本质是一个方法-(BOOL)isHighlighted这个方法是本质,看他返回什么,如果你自定义button,重写了这个方法设置了返回时YES,一定是高亮状态,如果设置了为NO,那么就算在外面设置了highlighted = YES;也不是高亮状态.

<!--more-->

**在开发中有时候我们需要取消按钮的高亮状态,可以通过两个方法来实现:**

- 1.重写set highlight方法 ,在这个方法中什么也不做,覆盖系统的做法
- 2,重写is highlight方法返回NO

#### 不可用(UIControlStateDisabled ):

- 设置按钮属性enable = NO; 按钮不会接受点击的事件
enabled = NO; 状态变为disabled
userinteractionEnabled = NO;按钮不能点击了 但是不会改变按钮的状态

#### 选中状态(UIControlStateSelected):

- 设置按钮的属性selected = YES; 按钮可以接受点击事件

#### 普通状态(UIControlStateNormal):最值得注意

- 看是这个状态没有什么,再普通不过了,但是却不是这样的,如果你同时设置了按钮的状态为选中状态和不可用状态,其实按钮不会按你设置的那样真的选中再不可用,而是普通状态(UIControlStateNormal),所以UIControlStateNormal是除开前面3中以外的状态 比如两个状态同时设置