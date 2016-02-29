---
layout: post
title: "类的外部初始化"
date: 2015-12-14T14:35:39+08:00
categories: iOS
---

当在进行类的设计的时候，遇到传值的问题的时候，比如下述问题，我们通过VC1获取了用户的姓名，要向VC2进行传递。现在的一般做法是在定义VC2的时候，在头文件中暴漏name变量。

~~~
@interface B : UIViewController
@property (strong) NSString* name;
@end
~~~

这种做法，封装性很差，任何持有VC2实例的地方都能够修改这个name值，导致一些很奇怪的逻辑。

其实这种情况应当属于外部初始化的典型应用。更好的方式就是我们就把name当成对象初始化必须的一个变量，需要对其进行初始化，那么就应当提供相应的函数来进行初始化。这样可以保持比较好的封装性。

建议以后采取这样的方式

~~~
// .h
@interface VC2 ： UIViewController
- （instancetype） init UNAVAILABLE;
-   （instancetype）initWithName:(NSString*)name;
@end

//.m
@interface VC2 ： UIViewController ()
{
     NSString* _name;
}
@end
@implatation VC2: UIViewController
-   （instancetype）initWithName:(NSString*)name
{
     self = [super init];
     if(!self) return self;
     _name = name;
     return self;
}
@end

~~~

在.h文件中进行变量声明的时候，如果不需要外部多次修改的变量，就不要暴漏了，做成私有变量，如果该变量初始化时所需的，那么就写成初始化函数哈。
