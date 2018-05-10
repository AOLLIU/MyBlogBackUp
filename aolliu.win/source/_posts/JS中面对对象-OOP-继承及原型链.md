---
title: JS中面对对象(OOP)继承及原型链
date: 2015-08-24 21:38:44
tags:
    - web开发
---


几大特性:封装 继承 多态 抽象


<!--more-->

## 继承

```
function Person(name,age){//person类,里面有两个属性
  this.name = name;
  this.age = age;
}

//每一个类都有一个prototype对象属性,给这个对象属性添加一个hi属性,并给hi属性赋值一个方法
Person.prototype.hi = funtion(){
  console.log("Hi,my name is" + this.name +",I am" + this.age +"years old now.");
}
//同样给这个prototype对象属性添加一个walk属性,并给这个属性赋值一个方法
Person.prototype.walk = function(){
  console.log(this.name + "is walking...");
}
//同上
Person.prototype.LEGS_NUM = 2;
Person.prototype.ARMS_NUM = 2;

//创建了一个Student类
function Student(name,age,className){
  Person.call(this,name,age);//先调用下父类的call方法,把this作为person里的this以及将name,age传进去
  this.className = className;
}

// Object.creat(对象)是创建一个空对象,这里是将person的prototype属性'赋值'给student的prototype属性,说赋值是不对的,应该说是继承给,这里用Object.creat()这种方法,而不用Student.prototype = Person.prototype;主要是如果这样的话如果给student添加属性,Person也会相应的增加这个属性,这不是我们想要的,现在这样做是不会影响Person中的属性的
Student.prototype = Object.creat(Person.prototype);
Student.prototype.constructor = Student;//这条语句没有必要写,因为本来就是Student

Student.prototype.hi = function(){
  console.log("Hi,my name is" + this.name +",I am" + this.age +"years old now and from" + this.className + ".");
}

//这里是给student.prototype添加一个learn属性,并且给他赋值一个函数对象
Student.prototype.learn = function(subject){
  console.log(this.name + "is learning" + subject + "at" + this.className)
}

//test
//创建一个实例对象wenxiaoli
var wenxiaoli = new Student('wenxiali',27,'Class 3,grade 2');

wenxiaoli.hi();//Hi,my name is wenxiaoli,I am 27 years old now,and form Class 3,grade 2

wenxiaoli.LEGS_NUM;//2

wenxiaoli.walk();//wenxiaoli is walking...
wenxiaoli.learn("math");//wenxiali is learning math,at Class 3, grade 2

```

### JS的继承是在原型链的基础上实现的,所以更深刻的理解JS里面的继承必须理解原型链的概念,对于上面的例子可以表现为下图

![Snip20161119_23.png](http://upload-images.jianshu.io/upload_images/1494773-ab26a1cef08cd778.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)