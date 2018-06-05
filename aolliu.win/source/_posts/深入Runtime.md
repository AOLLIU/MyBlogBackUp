---
title: 深入Runtime
date: 2016-04-15 19:07:41
tags:
	- iOS开发
	- OC

---


## 1.简介

- runtime 是一个运行时库，主要使用C和汇编写的库，为C添加了面向对象的能力并创造了Objective-C，并且拥有消息分发，消息转发等功能。开发者在编码过程中，可以给任意一个对象发送消息，在编译阶段只是确定了要向接收者发送这条消息，而接受者将要如何响应和处理这条消息，那就要看运行时来决定了。

- C语言中，在编译期，函数的调用就会决定调用哪个函数。而OC的函数，属于动态调用过程，在编译期并不能决定真正调用哪个函数，只有在真正运行时才会根据函数的名称找到对应的函数来调用。

- 开发者只需要编写 OC 代码即可，Runtime系统自动在幕后把我们写的源代码在编译阶段转换成运行时代码，在运行时确定对应的数据结构和调用具体哪个方法

<!--more-->

## 2.基本知识

要深入了解Runtime机制,就不得不知道NSObject类:

```
typedef struct objc_class *Class;
typedef struct objc_object *id;

@interface Object { 
    Class isa; 
}

@interface NSObject <NSObject> {
    Class isa  OBJC_ISA_AVAILABILITY;
}

struct objc_object {
private:
    isa_t isa;
}

struct objc_class : objc_object {
    // Class ISA;
    Class superclass;
    cache_t cache; 
    class_data_bits_t bits;
}

union isa_t 
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }
    Class cls;
    uintptr_t bits;
}

```

**分析:**

- 从上面源码我们可以看出所有的对象都会包含一个isa_t类型的结构体。
- 在objc_class中，除了isa之外，还有3个成员变量，一个是父类的指针，一个是方法缓存，最后一个这个类的实例方法链表(数据区域)。

**方法调用**
- 当一个对象的实例方法被调用的时候，会通过isa找到相应的类，然后在该类的class_data_bits_t中去查找方法。class_data_bits_t是指向了类对象的**数据区域**。在该数据区域内查找相应方法的对应实现。
- 但是在我们调用类方法的时候，类对象的isa里面是什么呢？这里为了和对象查找方法的机制一致，引入了元类(meta-class)的概念。meta-class存储着一个类的所有类方法。每个类都会有一个单独的meta-class.

- 在引入元类之后，类对象和对象查找方法的机制就完全统一了。

  - 对象的实例方法调用时，通过对象的 isa 在类中获取方法的实现。
  - 类对象的类方法调用时，通过类的 isa 在元类中获取方法的实现。

**下图描述了对象,类,元类的关系**

![Snip20180524_2.png](https://upload-images.jianshu.io/upload_images/1494773-1dfa3b3b1d8d200b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**分析:**

- 1.Root class (class)其实就是NSObject，NSObject是没有超类的，所以Root class(class)的superclass指向nil。
- 2.每个Class都有一个isa指针指向唯一的Meta class
- 3.Root class(meta)的superclass指向Root class(class)，也就是NSObject，形成一个回路。
- 4.每个Meta class的isa指针都指向Root class (meta)。


### 1.)isa_t (isa指针)
内部实现这里不详细描述
### 2.)cache_t

```
struct cache_t {
    struct bucket_t *_buckets;
    mask_t _mask;
    mask_t _occupied;
}

typedef unsigned int uint32_t;
typedef uint32_t mask_t;

typedef unsigned long  uintptr_t;
typedef uintptr_t cache_key_t;

struct bucket_t {
private:
    cache_key_t _key;
    IMP _imp;
}
```

**分析:**

- cache_t中存储了两个unsigned int的变量和一个bucket_t的结构体。
  - mask：分配用来缓存bucket的总数。
  - occupied：表明目前实际占用的缓存bucket的个数。
  - bucket_t的结构体中存储了一个unsigned long和一个IMP。IMP是一个函数指针，指向了一个方法的具体实现。
  
- cache_t中的bucket_t *_buckets其实就是一个散列表，用来存储Method的链表。

- Cache的作用主要是为了优化方法调用的性能。
  - 当对象receiver调用方法message时，首先根据对象receiver的isa指针查找到它对应的类.
  - 然后在类的methodLists中搜索方法，如果没有找到，就使用super_class指针到父类中的methodLists查找，一旦找到就调用方法。
  - 如果没有找到，有可能消息转发，也可能忽略它。但这样查找方式效率太低，因为往往一个类大概只有20%的方法经常被调用，占总调用次数的80%。
  - 所以使用Cache来缓存经常调用的方法，当调用方法时，优先在Cache查找，如果没有找到，再到methodLists查找。

### 3.)class_data_bits_t

```
struct class_data_bits_t {

    // Values are the FAST_ flags above.
    uintptr_t bits;
}

struct class_rw_t {
    uint32_t flags;
    uint32_t version;

    const class_ro_t *ro;

    method_array_t methods;
    property_array_t properties;
    protocol_array_t protocols;

    Class firstSubclass;
    Class nextSiblingClass;

    char *demangledName;
}

struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize;
#ifdef __LP64__
    uint32_t reserved;
#endif

    const uint8_t * ivarLayout;

    const char * name;
    method_list_t * baseMethodList;
    protocol_list_t * baseProtocols;
    const ivar_list_t * ivars;

    const uint8_t * weakIvarLayout;
    property_list_t *baseProperties;

    method_list_t *baseMethods() const {
        return baseMethodList;
    }
};
```

**分析:**

- Objc的类的属性、方法、以及遵循的协议在obj 2.0的版本之后都放在class_rw_t中。class_ro_t是一个指向常量的指针，存储来编译器决定了的属性、方法和遵守协议。备注: rw-readwrite，ro-readonly

**具体步骤:**

- 1,在编译期类的结构中的 class_data_bits_t *data指向的是一个 class_ro_t *指针
- 2,在运行时调用realizeClass方法，会做以下3件事情：

  - 从 class_data_bits_t调用 data方法，将结果从 class_rw_t强制转换为 class_ro_t指针
  - 初始化一个 class_rw_t结构体
  - 设置结构体 ro的值以及 flag

- 3,最后调用methodizeClass方法，把类里面的属性，协议，方法都加载进来。

方法method定义如下:

```
struct method_t {
    SEL name;
    const char *types;
    IMP imp;

    struct SortBySELAddress :
        public std::binary_function<const method_t&,
                                    const method_t&, bool>
    {
        bool operator() (const method_t& lhs,
                         const method_t& rhs)
        { return lhs.name < rhs.name; }
    };
};
```

- 里面包含3个成员变量。SEL是方法的名字name。types是Type Encoding类型编码
- IMP是一个函数指针，指向的是函数的具体实现。在runtime中消息传递和转发的目的就是为了找到IMP，并执行函数。



## 3.消息发送与转发

- 在 Objective-C中的“方法调用”其实应该叫做消息传递，`[object message]`会被编译器翻译为`objc_msgSend(object, @selector(message))`它有两个参数，第一个是 object ，既方法调用者，第二个参数称为选择子 SEL或者叫做方法选择器

### 消息发送阶段

- 1.objc_msgSend() 方法会先从缓存表里，查找是否有该 SEL 对应的 IMP，有的话算命中缓存，直接通过函数指针 IMP ，找到方法的具体实现函数，执行。
- 2.当然缓存表里可能并不会命中，则此时会根据类对象的 class_data_bits_t 指针找到数据区域，数据区域里用链表存放着类的实例方法
- 3.objc_msgSend() 会在类对象的方法链表里按链表顺序去匹配 SEL，匹配成功则停止，并将此方法加入到类对象的 _buckets 里缓存起来。如果没找到则会通过类对象的 superclass 指针找到其父类，去父类的方法列表里寻找（也会从父类的方法缓存列表开始）。
- 4.如果继续没有找到会一直向父类寻找，直到遇见 NSObject，NSObject 的 superclass 指向 nil。也就意味着寻找结束，并没有找到实现方法。（如果这个过程找到了，也同样会在 object 的类对象的 _buckets 里缓存起来）。
- 5.如果父类找到NSObject还没有找到,就进入了方法决议（method resolve），首先判断当前 object 的类对象是否实现了 resolveInstanceMethod: 方法，如果实现的话，会调用resolveInstanceMethod：方法，这个时候我们可以在resolveInstanceMethod：方法里动态的添加该SEL对应的方法。之后会重新执行查找方法实现的流程.
- 6.如果依旧没找到方法，或者没有实现 resolveInstanceMethod: 方法，Runtime 还有另一套机制，消息转发。

### 消息转发阶段


- 1.调用 forwardingTargetForSelector: 方法，尝试找到一个能响应该消息的对象。如果获取到，则直接转发给它。如果返回了 nil，继续下面的动作。

- 2.调用 methodSignatureForSelector: 方法，尝试获得一个方法签名。如果获取不到，则直接调用 doesNotRecognizeSelector 抛出异常。

- 3.调用 forwardInvocation: 方法，将第 2 步获取到的方法签名包装成 Invocation 传入，如何处理就在这里面了。

**备注:**
- 以上三个方法都可以通过在 object 的类对象里实现， forwardingTargetForSelector: 可以通过对参数 SEL 的判断，返回一个可以响应该消息的对象。这样则会重新从该对象开始执行查找方法实现的流程，找到了也同样会在 object 的类对象的 _buckets 里缓存起来。

- 而 2，3 方法则一般是配套使用，实现 methodSignatureForSelector: 方法根据参数 SEL ，做相应处理，返回 NSMethodSignature (方法签名) 对象，NSMethodSignature 对象会被包装成 NSInvocation 对象，forwardInvocation: 方法里就可以对 NSInvocation 进行处理了。

## 4.runtime使用

### (1)    实现多继承(假)
  - 一个类可以做到继承多个类的效果，只需要在`forwardingTargetForSelector`将消息转发给正确的类对象就可以模拟多继承的效果。
  
  - 消息转发提供了许多类似于多继承的特性，但是他们之间有一个很大的不同： 
    - 多继承：合并了不同的行为特征在一个单独的对象中，会得到一个重量级多层面的对象。   
    - 消息转发：将各个功能分散到不同的对象中，得到的一些轻量级的对象，这些对象通过消息通过消息转发联合起来。
  
### (2)    Method Swizzling
  - Method Swizzing是发生在运行时的，主要用于在运行时将两个Method进行交换.
  
  - Method Swizzling也是iOS中AOP(面相切面编程)的一种实现方式，我们可以利用苹果这一特性来实现AOP编程。Method Swizzling本质上就是对IMP和SEL进行交换。
  - 一般我们使用都是新建一个分类，在分类中进行Method Swizzling方法的交换
  
  - **注意**:
    Swizzling应该总在+load中执行,Objective-C在运行时会自动调用类的两个方法+load和+initialize。+load会在类初始加载时调用， +initialize方法是以懒加载的方式被调用的，如果程序一直没有给某个类或它的子类发送消息，那么这个类的 +initialize方法是永远不会被调用的。所以Swizzling要是写在+initialize方法中，是有可能永远都不被执行。更多关于load,initialize可以看[这里](http://aolliu.win/2015/09/01/load%E6%96%B9%E6%B3%95%E5%92%8Cinitialize%E6%96%B9%E6%B3%95/)

#### Method Swizzling使用场景

- 1.实现AOP
- 2.实现埋点统计
- 3.实现异常保护



### (3)    AOP 面向切面编程
- 类似记录日志、身份验证、缓存等事务非常琐碎，与业务逻辑无关，很多地方都有，又很难抽象出一个模块，这种程序设计问题，有一个名字叫横向关注点(Cross-cutting concern)，AOP作用就是分离横向关注点(Cross-cutting concern)来提高模块复用性，它可以在既有的代码添加一些额外的行为(记录日志、身份验证、缓存)而无需修改代码。

#### AOP工作原理:

**1,)前景介绍**
- 1.在objc_msgSend函数查找IMP的过程中，如果在父类也没有找到相应的IMP，那么就会开始执行`_class_resolveMethod`方法，如果不是元类，就执行`_class_resolveInstanceMethod`，如果是元类，执行`_class_resolveClassMethod`。在这个方法中，允许开发者动态增加方法实现。

- 2.如果`_class_resolveMethod`无法处理，会开始选择备援接受者接受消息，这个时候就到了`forwardingTargetForSelector`方法。如果该方法返回非nil的对象，则使用该对象作为新的消息接收者。`forwardingTargetForSelector`这种方法属于单纯的转发，无法对消息的参数和返回值进行处理。

- 3.最后到了完整转发阶段。Runtime系统会向对象发送`methodSignatureForSelector:`消息，并取到返回的方法签名用于生成NSInvocation对象。为接下来的完整的消息转发生成一个`NSMethodSignature`对象`。NSMethodSignature` 对象会被包装成` NSInvocation` 对象，`forwardInvocation: `方法里就可以对 `NSInvocation` 进行处理了。

```
// 为目标对象中被调用的方法返回一个NSMethodSignature实例
// 运行时系统要求在执行标准转发时实现这个方法
- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel{
    return [self.proxyTarget methodSignatureForSelector:sel];
}
```
- 4.对象需要创建一个NSInvocation对象，把消息调用的全部细节封装进去，包括selector, target, arguments 等参数，还能够对返回结果进行处理。

**2,)AOP原理**
AOP的多数操作就是在forwardInvocation中完成的。一般会分为2个阶段，一个是Intercepter注册阶段，一个是Intercepter执行阶段。

#### 2-1). Intercepter注册阶段

![Snip20180605_6.png](https://upload-images.jianshu.io/upload_images/1494773-c1a302bc1ad030ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 首先会把类里面的某个要切片的方法的IMP加入到Aspect中，类方法里面如果有forwardingTargetForSelector:的IMP，也要加入到Aspect中。

![Snip20180605_7.png](https://upload-images.jianshu.io/upload_images/1494773-0836d46e6097ca1b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 然后对类的切片方法和forwardingTargetForSelector:的IMP进行替换。两者的IMP相应的替换为objc_msgForward()方法和hook过的forwardingTargetForSelector:。这样主要的Intercepter注册就完成了。

#### 2-2).Intercepter执行
![Snip20180605_8.png](https://upload-images.jianshu.io/upload_images/1494773-f0f19db342090f54.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 当执行func()方法的时候，会去查找它的IMP，现在它的IMP已经被我们替换为了objc_msgForward()方法，于是开始查找备援转发对象。

- 查找备援接受者调用forwardingTargetForSelector:这个方法，由于这里是被我们hook过的，所以IMP指向的是hook过的forwardingTargetForSelector:方法。这里我们会返回Aspect的target，即选取Aspect作为备援接受者。

- 有了备援接受者之后，就会重新objc_msgSend，从消息发送阶段重头开始。

- objc_msgSend找不到指定的IMP，再进行_class_resolveMethod，这里也没有找到，forwardingTargetForSelector:这里也不做处理，接着就会methodSignatureForSelector。在methodSignatureForSelector方法中创建一个NSInvocation对象，传递给最终的forwardInvocation方法。

- Aspect里面的forwardInvocation方法会干所有切面的事情。这里转发逻辑就完全由我们自定义了。Intercepter注册的时候我们也加入了原来方法中的method()和forwardingTargetForSelector:方法的IMP，这里我们可以在forwardInvocation方法中去执行这些IMP。在执行这些IMP的前后都可以任意的插入任何IMP以达到切面的目的。

更好的理解面向切面编程可以看[这个项目](https://github.com/steipete/Aspects)

### (4)    Associated Object关联对象
- 1.为现有的类添加私有变量 
- 2.为现有的类添加公有属性 
- 3.为KVO创建一个关联的观察者。

### (5)    动态的增加方法
- 在消息发送阶段，如果在父类中也没有找到相应的IMP，就会执行resolveInstanceMethod方法。在这个方法里面，我们可以动态的给类对象或者实例对象动态的增加方法。

### (6)    NSCoding的自动归档和自动解档
- runtime实现归档解档，我们循环依次找到每个成员变量的名称，然后利用KVC读取和赋值就可以完成encodeWithCoder和initWithCoder了。

### (7)    字典和模型互相转换

#### 7-1).字典转模型
- 1.调用 class_getProperty 方法获取当前 Model 的所有属性。
- 2.调用 property_copyAttributeList 获取属性列表。
- 3.根据属性名称生成 setter 方法。
- 4.使用 objc_msgSend 调用 setter 方法为 Model 的属性赋值（或者 KVC）

#### 7-2).模型转字典

- 1.调用 class_copyPropertyList 方法获取当前 Model 的所有属性。
- 2.调用 property_getName 获取属性名称。
- 3.根据属性名称生成 getter 方法。
- 4.使用 objc_msgSend 调用 getter 方法获取属性值（或者 KVC）






