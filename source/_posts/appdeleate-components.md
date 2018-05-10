---
title: appdeleate-components
date: 2018-05-10 16:32:42
tags:
---


前段时间看到一个关于AppDelegate瘦身的文章 [AppDelegate瘦身指南](http://www.cocoachina.com/apple/20180316/22638.html)。想起来自己在这方面做得一些事情，拿出来一起分享一下。我写了一个单独的类库来处理这个问题： [MRAppDelegateComponents](https://github.com/yishuiliunian/MRAppDelegateComponents)。

使用可以直接：

~~~
pod 'MRAppDelegateComponents'
~~~

关于AppDelegate碰到的问题不再赘述。但是我们可以把问题再抽象一层，AppDelegate碰到的问题本质上是 `AppDelegate` 作为整个App的一个入口，特别容易承载过多的业务逻辑，从而变成一个臃肿的类。这样的问题同样也存在于 ViewController 中。现在我们只讨论 AppDelegate 的解耦问题。关于 ViewController 的解耦问题可以参考我的另外一个库 [DZViewControllerLifeCircleAction](https://github.com/yishuiliunian/DZViewControllerLifeCircleAction)。所谓解耦，就是把不同的代码按照其职责，拆解到不同的单元或者模块中（具体是什么，要看拆解的手段）。

而拆解之后必然首先要保障的是原有的业务逻辑能够顺利执行。而且各个业务逻辑之间如果有交互，还能够顺利的完成。


之前我们看到的方案主要是有两类：

1. 一类注重于分发，比如 FRDModuleManager 和 JSDecoupledAppDelegate。 通过一定的机制，将Delegate分发到对应执行的业务单元上执行。而各个业务单元之间执行可以做到不互斥。但是要穷举所有的方法，进行转发也是个麻烦事。

2. 一种是Category。这种虽然可以拆解，但是由于Category的技术限制，同一个方法不能出现在不同的Category中。结果导致了互斥。也就是说，同一个方法你只能写一份。不然就会出问题。

而当我们仔细去分析 AppDelegate 中的方法的时候，会发现有些方法是有返回值的：

~~~
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions;
~~~

这里自然就会延伸出一个问题，这个返回值怎么处理？

比如在多个模块中都需要对 `application: didFinishLaunchingWithOptions:` 做出一定的动作。一般情况下我们都会丢弃掉返回值。直接按照一个固定的返回输出 `return YES;`。

而当同一个函数都有一定的返回值的时候，我们自然会想到一个设计模式：责任链模式。所有的相同调用处在一个链条上，当前处理单元会根据上一个处理单元的结果来做些动作，并且能够决定是否继续往下去处理。

到这里我们把解决方案再明确一下：把 AppDelegate 的调用解耦成一个 责任链 模型。并且满足以下特征：

1. 每个模块都可以无限制的实现 AppDelegate 的方法。
2. 每个模块将会实现多个 AppDelegate 的函数以完成一定的业务逻辑，比如对于URLScheme的处理。
3. 对于有返回值的函数，将其调用关系转化成一个 责任链。 能够让调用者之间根据返回值交互。



下面先看一下解决方案，下面是拆解出来的两个逻辑单元 (代码可以从[MRAppDelegateComponents](https://github.com/yishuiliunian/MRAppDelegateComponents)) 中的 Demo 中找到，只要把这两个类创建好，关键的是实现协议 `MRApplicationDelegateInjectionProtocol `，框架会自动执行你重载的方法：

~~~
#import <MRAppDelegateComponents/MRAppDelegateComponents.h>
@interface MRAppDelegateLogic1 :NSObject  <UIApplicationDelegate, MRApplicationDelegateInjectionProtocol>

@end
@implementation MRAppDelegateLogic1
- (BOOL) application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    SEL sel = @selector(application:didFinishLaunchingWithOptions:);
    if (__MRSuperImplatationCurrentCMD__(sel)) {
        MRPrepareSendSuper(BOOL, id, id);
        MRSendSuperSelector(sel, application, launchOptions);
    }
    NSLog(@"MRAppDelegateLogic1 handle");
    return YES;
}
@end
~~~


~~~
#import <MRAppDelegateComponents/MRAppDelegateComponents.h>
@interface MRAppDelegateLogic2 :NSObject <UIApplicationDelegate, MRApplicationDelegateInjectionProtocol>

@end
@implementation MRAppDelegateLogic2
- (BOOL) application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    SEL sel = @selector(application:didFinishLaunchingWithOptions:);
    if (__MRSuperImplatationCurrentCMD__(sel)) {
        MRPrepareSendSuper(BOOL, id, id);
        MRSendSuperSelector(sel, application, launchOptions);
    }
    NSLog(@"MRAppDelegate Logic2 handle magic");
    return YES;
}
@end
~~~


先讲解一下用法，然后我们再去解释实现原理。以 `MRAppDelegateLogic2` 为例。我们可以看到，我新建了一个类 `MRAppDelegateLogic2` 并且重载了 `application:didFinishLaunchingWithOptions:` 方法。 然后在函数实现体里面有几个比较有意思的东西。

我首先定义了当前函数的一个变量。然后通过一个我写的宏去判断其父类有没有实现该方法，如果有就调用，如果没有则不调用。然后执行一段自定义的逻辑。这里忽略了父类方法的返回值。当然你也可以处理。

当你运行Demo的时候，你会你的逻辑正常的输出了。


~~~
:MRAppDelegateLogic1 handle
:MRAppDelegate Logic2 handle magic
~~~

这里是怎么实现的呢？使用了，我之前写的一个基础类库 [MRLogicInjection](https://github.com/yishuiliunian/MRLogicInjection)。


1. Hook UIApplication 的 setDelegate 方法
2. 搜寻所有遵循了我们自定义的协议 （`MRApplicationDelegateInjectionProtocol `）的类
3. 把类中同名方法的逻辑，注入到上一个类中，生成一个新的元类，重复直到所有的类都注入完成
4. 将 AppDeletge 的 isa 指针指向这个新的元类

这样就可以完成类的拆解了。同时，满足了我们最开始说的几个诉求。在 `MRLogicInjection `中我对于LogicInjection的东西做了一些深入的说明。其他的细节可以参考 [MRLogicInjection](https://github.com/yishuiliunian/MRLogicInjection) 的 ReadMe 文件。

同时，我们也会发现。这样的处理策略其实是实现了一套可以将一个类拆解成多个类，然后再合并到一起的能力。当然使用OC原始的继承策略可以实现。但是当处理Delegate的时候原生的继承就比较尴尬了。比如UITextView的delegate其中还实现了UIScrollDelegate。两个delegate揉在一起，当你想在另外一个类监听Scroll的变化的时候，就会影响原始的delegate，毕竟delegate的指针只有一个。你继续使用刚刚说的这套机制，一样可以处理UITextView的delegte。

总之 MRAppDelegateComponents 是 基于 MRLogicInjection 对delegate 进行解耦和拆解模块的一个 实例。你要解决 AppDeleagte 的问题可以直接只用这个类库。如果想处理其他 Delegate 的问题，可以参考这个类库的实现处理。