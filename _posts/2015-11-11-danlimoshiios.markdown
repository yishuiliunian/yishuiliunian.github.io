---
layout: post
title: "iOS设计模式反思之单例模式的进化"
date: 2015-11-11T15:25:12+08:00
categories: iOS
---



#单例模式

什么是单例模式？ 单例模式想一个大独裁者，他规定在他的国度里面，所有数据的访问和请求都得经过他，甚至你要调用相关的函数也得经过它。学术一点就是，单例模式，为某一类需求和数据提供了统一的程序接口。主要的实现技术就是，确保全局只有一个对象的实例存在。举个例子把，比如NSNotificationCenter 中的 defaultCenter 负责全局的消息分发、NSFileManager 的 defaultManager 统一负责物理文件的管理、NSUserDefaults 的 standardUserDefaults 统一管理用户的配置文件……不一而足。在整个iOS框架中，可以说是大规模使用了单例模式。

##单例模式的原理及实现

###在非ARC情况下实现一个单例

~~~
@implementation DZSinglonNoARC
+ (DZSinglonNoARC*) shareInstance
{
    static DZSinglonNoARC* share = nil;
    @synchronized(self)
    {
        if (!share) {
            share = [[super allocWithZone:NULL] init];
        }
    }
    return share;
}
+ (instancetype) allocWithZone:(struct _NSZone *)zone
{
    return [self shareInstance];
}
- (instancetype) copyWithZone:(NSZone*)zone
{
    return self;
}
- (id) retain
{
    return [DZSinglonNoARC shareInstance];
}
- (oneway void) release
{

}
- (instancetype) autorelease
{
    return self;
}
- (unsigned) retainCount
{
    return UINT_MAX;
}
@end
~~~
首先要初始化一个该类的静态化变量


~~~
static DZSinglonNoARC* share = nil;
    @synchronized(self)
    {
        if (!share) {
            share = [[super allocWithZone:NULL] init];
        }
    }
    return share;
~~~
在这里进行了加锁处理，是为了防止多线程重入的情况下，造成静态变量多次分配内存和初始化，从而会导致数据混乱。这里加锁的对象是self，实际上是

~~~
[DZSinglonNoARC class]
~~~
类DZSinglonNoARC的class对象，也就是说加锁对象是全局唯一的一个Class对象。而且在这里share的定义放在了函数shareInstance之内，是要让share变成一个函数内的局部变量这样可以防止，外部的异常访问。看到网上有些教程中，把share的定义放在函数之外，变成了文件内的一个全局静态变量，这样会存在其他函数异常操作share的情况。

注意在初始化share的时候我们使用的是

~~~
share = [[super allocWithZone:NULL] init];
~~~
之所以这样做，是为了防止想用[DZSinglonNoARC alloc]等函数的时候，会与allocWithZone引起的死锁和死循环。

而关于此处加锁的处理还有一个优化版本，在4.0以上的SDK有了闭包之后，我们可以这么做

~~~
+ (DZSinglonNoARC*) share2
{
    static DZSinglonNoARC* share = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&amp;onceToken, ^{    
        share = [[super allocWithZone:NULL] init];
    });    
    return share;
}
~~~
使用dispatch_once体带原来的加锁操作，这样做的好处就是可以减少每次加锁的时间，优化了程序性能。dispatch_once函数接收一个dispatch_once_t用于检查该代码块是否已经被调度的谓词（是一个长整型，实际上作为BOOL使用）。它还接收一个希望在应用的生命周期内仅被调度一次的代码块，对于本例就用于share实例的实例化。dispatch_once不仅意味着代码仅会被运行一次，而且还是线程安全的。完全可以替代相对低效的加锁操作。

然后是重载了一些列的函数，重载这些函数的目的，就是修改原有的NSOjbect的内存操作相关的函数，保持内存中有且只有一个该类的对象。

###arc下的实现

我们先看一个例子

~~~
@implementation DZSinglonARC
+  (DZSinglonARC*) shareInstance
{
    static DZSinglonARC* share = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&amp;onceToken, ^{
        share = [[super allocWithZone:NULL] init];
    });
    return share;
}
+ (instancetype) allocWithZone:(struct _NSZone *)zone
{
    return [self shareInstance];
}
@end
~~~
通过上面的代码我们能够看出其实在ARC下单例的实现方式与非ARC下大同小异，只不过是没有重载内存管理的函数而已。而这也得益于ARC这种技术。

##单例工厂

在实际的编程工作中，随着项目规模的不断扩大，我们往往发现在整个项目中存在着大量的单例。于是我们就会遇到一个问题如何去管理这些单例。 同时，也会遇到每一次都要按照上面的实现方式从头来一遍来实现一个单例，从编程效率上看难免有些低下，毕竟很多代码都是相同的。使用设计模式的目标之一就是合理的干掉重复的代码。那么有没有一个好的方式来管理这些单例呢，我们很自然的想到了工厂模式。

工厂方法模式（英语：Factory method pattern）是一种实现了“工厂”概念的面向对象设计模式。就像其他创建型模式一样，它也是处理在不指定对象具体类型的情况下创建对象的问题。工厂方法模式的实质是“定义一个创建对象的接口，但让实现这个接口的类来决定实例化哪个类。工厂方法让类的实例化推迟到子类中进行。” 创建一个对象常常需要复杂的过程，所以不适合包含在一个复合对象中。创建对象可能会导致大量的重复代码，可能会需要复合对象访问不到的信息，也可能提供不了足够级别的抽象，还可能并不是复合对象概念的一部分。工厂方法模式通过定义一个单独的创建对象的方法来解决这些问题。由子类实现这个方法来创建具体类型的对象。(引用自WIKI)

工厂模式解决的就是这种，重建同类型对象的问题。而这里，我们可以把单例看成同类型的一系列对象。那就创建一个单例工厂吧。 项目地址：https://github.com/yishuiliunian/DZSinglonFactory.git

先看一下如何实现一个简单的单例工厂

~~~
@interface DZSingletonFactory()
{
    NSMutableDictionary* data;
}
@end
@implementation DZSingletonFactory
- (id) init
{
    self = [super init];
    if (self) {
        data = [[NSMutableDictionary alloc] init];
    }
    return self;
}
+ (DZSingletonFactory*) shareFactory
{
    static DZSingletonFactory* share = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&amp;onceToken, ^{
        share = [[DZSingletonFactory alloc] init];
    });
    return share;
}
- (id) copyWithZone:(NSZone*)zone
{
    return self;
}
//over singlong
- (void) setShareData:(id)shareData  forKey:(NSString*)key
{
        if (shareData == nil) {
            return;
        }
        [data setObject:shareData forKey:key];
}
- (id) shareDataForKey:(NSString*)key
{
        return [data objectForKey:key];
}
- (id) shareInstanceFor:(Class)aclass
{
    NSString* className = [NSString stringWithFormat:@"%@",aclass];
     @synchronized(className)
    {
        id shareData = [self shareDataForKey:className];
        if (shareData == nil) {
            shareData = [[NSClassFromString(className) alloc] init];
            [self setShareData:shareData forKey:className];
        }
        return shareData;
    }

}
- (id) shareInstanceFor:(Class)aclass category:(NSString *)key
{
    NSString* className = [NSString stringWithFormat:@"%@",aclass];
    NSString* classKey = [NSString stringWithFormat:@"%@-%@",aclass,key];
    @synchronized(classKey)
    {
        id shareData = [self shareDataForKey:classKey];
        if (shareData == nil) {
            shareData = [[NSClassFromString(className) alloc] init];
            [self setShareData:shareData forKey:classKey];
        }
        return shareData;
    }
}
@end
~~~

其实单例工厂类也是一个实例，之所以这么做，是因为我们所生产的单例总得有个仓库存着吧。而这个单例工厂类除了有生产单例的功能，也担负着仓库存储生产的单例的功能。私有变量NSMutableDictionary* data;就是仓库。

而生产的车间是

~~~
- (id) shareInstanceFor:(Class)aclass
{
    NSString* className = [NSString stringWithFormat:@"%@",aclass];
     @synchronized(className)
    {
        id shareData = [self shareDataForKey:className];
        if (shareData == nil) {
            shareData = [[NSClassFromString(className) alloc] init];
            [self setShareData:shareData forKey:className];
        }
        return shareData;
    }
}
~~~
这个函数及其简单，我们使用了Objective-C一些动态语言的特性，直接通过类Class对象来生成实例，如果你对objc的底层有些了解的话，应该知道其实Class也是一个对象，他也能够执行objc的方法。也就是说，如果我们要生成类A的一个实例，只要我们有了类A的类型对象（Class）实例就OK，然后通过[(Class*)aClass new]你就能轻而易举的生成一个实例。

在我们通过这种技术生成了一个单例的实例之后，将其存储在仓库data里面，下次再次请求这个类的实例的时候，只要从仓库中取出来用就行了。

我们甚至为了编程时再少写点代码可以写一个函数和宏：

~~~
id  DZSingleForClass(Class a)
{
    return [DZShareSingleFactory shareInstanceFor:a];
}
..........
#define DZShareSingleFactory [DZSingletonFactory shareFactory]
#ifdef __cplusplus
extern "C" {
#endif
    id  DZSingleForClass(Class a);
#ifdef __cplusplus
}
#endif
~~~

这样我们在需要创建单例的时候一句话就能搞定：

~~~
+ (DZSingletonFactory*) shareInstance
{
    return DZSingleForClass([DZShareSingleFactory class]);
}
~~~
不过这只是一个极度简化版的单例工厂，很多保护性的措施还都没做。比如对于allocWithZone的重载等，还有一些单例注销的操作。

其实这是一种这种的策略，我们没有重载内存管理函数，是为了能够在后面为了节省内存，在单例较长时间不用的时候将其销毁掉，等下次用的时候再创建。

~~~
- (void) destoryInstanceFor:(Class)aclass
{
    NSString* className = [NSString stringWithFormat:@"%@",aclass];
    @synchronized(className) {
        if ([self shareDataForKey:className]) {
            [data removeObjectForKey:className];
        }
    }
}
~~~
当然这中使用方式，看起来不太像是严格意义上的单例模式，但是他却完成单例模式最根本的意图，把接口和功能统一。

##模块管理系统

还记得文章的标题吗？单例模式的进化，那么这里单例模式要进化到什么地步呢？在实际的编码过程中，随着工程规模的不断扩大，我们可能会在我们的项目中大规模的时候单例模式。就像是世界中，有了N多个独裁者，协调这些独裁者，对任何一个系统来说都是困难的。这个时候，就会遇到一个问题：如何有效的管理数量较多的单例。虽然我们使用单例工厂的方式，解决了单例统一构建的问题，但是没有能够解决统一管理的问题。这里统一管理的问题包括：

单例初始化的控制，不同的单例在初始化的时候可能需要不同的参数。甚至有些单例可以延迟加载。
单例注销的统一管理。
能够让开发者或者SDK使用者，直接了当的看到整个工程中使用了多少单例，每个单例的大概模样是怎么样子的。
管理各个单例之间的依赖关系。比如地理位置信息单例可能会依赖数据库的一个单例。
根据不同的需求，动态的加载某些单例。
其他等等……
这些需求一列，你会感觉怎么像是在说一个包管理系统比如debian的apt-get或者node的npm之类的东西呢，甚至还有点像windows的动态链接库或者linux的模块化机制。

其实如果我们剖析一下的话，其实单例模式是一种实现程序架构模块化的有利工具。他将某些内聚性非常高的功能，聚合在其一起，通过单一实例与外界交互。同时也降低了与其他单例之间的耦合性。从宏观的角度来看，我们可以把一个个单例看成一个个功能性模块，用一个形象的比喻就是插件。这样，一个充斥着大量单例，并且能够对这些单例有效管理的系统，从宏观的角度看，就像是一个插件系统（准确说是模块管理系统）。

而上面提到的项目[DZSinglonFactory](https://github.com/yishuiliunian/DZSinglonFactory.git)只是这个插件系统开了个头。有兴趣的可以一起来玩上一个iOS上的简单的插件管理系统。
