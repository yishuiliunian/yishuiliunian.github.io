---
layout: post
title: "iOS设计模式之观察者模式"
date: 2015-11-11T12:20:12+08:00
---


什么是观察者模式？我们先打个比方，这就像你订报纸。比如你想知道美国最近放生了些新闻，你可能会订阅一份美国周刊，然后一旦美国有了新的故事，美国周刊就发一刊，并邮寄给你，当你收到这份报刊，然后你就能够了解美国最新的动态。其实这就是观察者模式，A对B的变化感兴趣，就注册为B的观察者，当B发生变化时通知A，告知B发生了变化。这是一种非常典型的观察者的用法，我把这种使用方法叫做经典观察者模式。当然与之相对的还有另外一种观察者模式——广义观察者模式。

从经典的角度看，观察者模式是一种通知变化的模式，一般认为只在对象发生变化感兴趣的场合有用。主题对象知道有观察者存在，设置会维护观察者的一个队列；而从广义的角度看，观察者模式是中传递变化数据的模式，需要查看对象属性时就会使用的一种模式，主题对象不知道观察者的存在，更像是围观者。需要知道主题对象的状态，所以即使在主题对象没有发生改变的时候，观察者也可能会去访问主题对象。换句话说广义观察者模式，是在不同的对象之间传递数据的一种模式。

观察者模式应当是在面向对象编程中被大规模使用的设计模式之一。从方法论的角度出发，传统的认知论认为，世界是由对象组成的，我们通过不停的观察和了解就能够了解对象的本质。整个人类的认知模型就是建立在“观察”这种行为之上的。我们通过不停与世界中的其他对象交互，并观察之来了解这个世界。同样，在程序的世界中，我们构建的每一个实例，也是通过不不停的与其他对象交互（查看其他对象的状态，或者改变其他对象的状态），并通过观察其他实例的变化并作出响应，以来完成功能。这也就是，为什么会把观察模式单独提出来，做一个专门的剖析的原因——在我看来他是很多其他设计模式的基础模式，并且是编程中极其重要的一种设计模式。

##经典观察者模式

经典观察者模式被认为是对象的行为模式，又叫发布-订阅(Publish/Subscribe)模式、模型-视图(Model/View)模式、源-监听器(Source/Listener)模式或从属者(Dependents)模式。经典观察者模式定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个主题对象。这个主题对象在状态上发生变化时，会通知所有观察者对象，使它们能够自动更新自己或者做出相应的一些动作。在文章一开始举的例子就是典型观察者模式的应用。

而在IOS开发中我们可能会接触到的经典观察者模式的实现方式，有这么几种：NSNotificationCenter、KVO、Delegate等

###感知通知方式

在经典观察者模式中，因为观察者感知到主题对象变化方式的不同，又分为推模型和拉模型两种方式。

####推模型

![](http://ww1.sinaimg.cn/large/7df22103jw1exwx2dkgj7j20ol0dhgn2.jpg)

主题对象向观察者推送主题的详细信息，不管观察者是否需要，推送的信息通常是主题对象的全部或者部分数据。推模型实现了观察者和主题对象的解耦，两者之间没有过度的依赖关系。但是推模型每次都会以广播的方式，向所有观察者发送通知。所有观察者被动的接受通知。当通知的内容过多时，多个观察者同时接收，可能会对网络、内存（有些时候还会涉及IO）有较大影响。

在IOS中典型的推模型实现方式为NSNotificationCenter和KVO。

#####NSNotificationCenter

NSnotificationCenter是一种典型的有调度中心的观察者模式实现方式。以NSNotificationCenter为中心，观察者往Center中注册对某个主题对象的变化感兴趣，主题对象通过NSNotificationCenter进行变化广播。这种模型就是文章开始发布订阅报纸在OC中的一种类似实现。所有的观察和监听行为都向同一个中心注册，所有对象的变化也都通过同一个中心向外广播。

NSNotificationCenter就像一个枢纽一样，处在整个观察者模式的核心位置，调度着消息在观察者和监听者之间传递。

![](http://ww3.sinaimg.cn/large/7df22103jw1exwx3ni8l8j20nl06amxm.jpg)

一次完整的观察过程如上图所示。整个过程中，关键的类有这么几个（介绍顺序按照完成顺序):  

######1. 观察者Observer，一般继承自NSObject，通过NSNotificationCenter的~~~addObserver:selector:name:object~~~接口来注册对某一类型通知感兴趣.在注册时候一定要注意，~~~NSNotificationCenter~~~不会对观察者进行引用计数+1的操作，我们在程序中释放观察者的时候，一定要去报从center中将其注销了。

######2. 通知中心NSNotificationCenter，通知的枢纽。

######3. 主题对象，被观察的对象，通过postNotificationName:object:userInfo:发送某一类型通知，广播改变。

~~~
- (void) postMessage
 {
     [[NSNotificationCenter defaultCenter] postNotificationName:kDZTestNotificatonMessage object:Nil userInfo:@{}];
 }
~~~

######4. 通知对象NSNotification，当有通知来的时候，Center会调用观察者注册的接口来广播通知，同时传递存储着更改内容的NSNotification对象。

#####apple版实现的NotificationCenter让我用起来不太爽的几个小问题

在使用NSNotificationCenter的时候，从编程的角度来讲我们往往不止是希望能够做到功能实现，还能希望编码效率和整个工程的可维护性良好。而Apple提供的以NSNotificationCenter为中心的观察者模式实现，在可维护性和效率上存在以下缺点：

每个注册的地方需要同时注册一个函数，这将会带来大量的编码工作。仔细分析能够发现，其实我们每个观察者每次注册的函数几乎都是雷同的。这就是种变相的CtrlCV，是典型的丑陋和难维护的代码。
每个观察者的回调函数，都需要对主题对象发送来的消息进行解包的操作。从UserInfo中通过KeyValue的方式，将消息解析出来，而后进行操作。试想一下，工程中有100个地方，同时对前面中在响应变化的函数中进行了解包的操作。而后期需求变化需要多传一个内容的时候，将会是一场维护上的灾难。
当大规模使用观察者模式的时候，我们往往在dealloc处加上一句:
[[NSNotificationCenter defaultCenter] removeObserver:self]
而在实际使用过程中，会发现该函数的性能是比较低下的。在整个启动过程中，进行了10000次RemoveObserver操作，


~~~
@implementation DZMessage
 - (void) dealloc
 {
    [[NSNotificationCenter defaultCenter] removeObserver:self];
 }
 ....
for (int i = 0 ; i &lt; 10000; i++) {
      DZMessage* message = [DZMessage new];
}
~~~

通过下图可以看出这一过程消耗了23.4%的CPU，说明这一函数的效率还是很低的。
![](http://ww1.sinaimg.cn/large/7df22103jw1exwx55g3mdj20m708bjui.jpg)
这还是只有一种消息类型的存在下有这样的结果，如果整个NotificationCenter中混杂着多种消息类型，那么恐怕对于性能来说将会是灾难性的。

~~~
for (int i = 0 ; i &lt; 10000; i++) {
      DZMessage* message = [DZMessage new];
      [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(handle) name:[@(i) stringValue] object:nil];
  }
~~~

增加了多种消息类型之后，RemoveObserver占用了启动过程中63.9%的CPU消耗。
![](http://ww2.sinaimg.cn/large/7df22103jw1exwx5xzf1tj20lf07tju8.jpg)
而由于Apple没有提供Center的源码，所以修改这个Center几乎不可能了。

#####改进版的有中心观察者模式（DZNotificationCenter）

GitHub地址 在设计的时候考虑到以上用起来不爽的地方，进行了优化：

1. 将解包到执行函数的操作进行了封装，只需要提供某消息类型的解包block和消息类型对应的protocol，当有消息到达的时候，消息中心会进行统一解包，并直接调用观察者相应的函数。
2. 对观察者的维护机制进行优化（还未做完），提升查找和删除观察者的效率。

DZNotificationCenter的用法和NSNotificationCenter在注册和注销观察者的地方是一样的，不一样的地方在于，你在使用的时候需要提供解析消息的block。你可以通过两种方式来提供。

######1. 直接注册的方式

~~~
[DZDefaultNotificationCenter addDecodeNotificationBlock:^SEL(NSDictionary *userInfo, NSMutableArray *__autoreleasing *params) {
      NSString* key = userInfo[@"key"];
      if (params != NULL) {
          *params = [NSMutableArray new];
      }
      [*params  addObject:key];
      return @selector(handleTestMessageWithKey:);
  } forMessage:kDZMessageTest];
~~~

######2. 实现DZNotificationInitDelegaete协议，当整个工程中大规模使用观察者的时候，建议使用该方式。这样有利于统一管理所有的解析方式。

~~~
- (DZDecodeNotificationBlock) decodeNotification:(NSString *)message forCenter:(DZNotificationCenter *)center
{
    if (message == kDZMessageTest) {
        return ^(NSDictionary* userInfo, NSMutableArray* __autoreleasing* params){
            NSString* key = userInfo[@"key"];
            if (params != NULL) {
                *params = [NSMutableArray new];
            }
            [*params  addObject:key];
            return @selector(handlePortMessage:);
        };
    }
    return nil;
}
~~~

在使用的过程中为了，能够保证在观察者处能够回调相同的函数，可以实现针对某一消息类型的protocol

~~~
@protocol DZTestMessageInterface &lt;NSObject&gt;
- (void) handleTestMessageWithKey:(NSString*)key;
@end
~~~
这样就能够保证，在使用观察者的地方不用反复的拼函数名和解析消息内容了。

~~~
@interface DZViewController () &lt;DZTestMessageInterface&gt;
@end
@implementation DZViewController
....
- (void) handleTestMessageWithKey:(NSString *)key
{
    self.showLabel.text = [NSString stringWithFormat:@"get message with %@", key];
}
....
~~~

###KVO

KVO的全称是Key-Value Observer，即键值观察。是一种没有中心枢纽的观察者模式的实现方式。一个主题对象管理所有依赖于它的观察者对象，并且在自身状态发生改变的时候主动通知观察者对象。 让我们先看一个完整的示例：

~~~
static NSString* const kKVOPathKey = @"key";
@implementation DZKVOTest
- (void) setMessage:(DZMessage *)message
{
    if (message != _message) {
        if (_message) {
            [_message removeObserver:self forKeyPath:kKVOPathKey];
        }
        if (message) {
            [message addObserver:self forKeyPath:kKVOPathKey options:NSKeyValueObservingOptionNew context:Nil];
        }
        _message = message;
    }
}
- (void) observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
    if ([keyPath isEqual:kKVOPathKey] &amp;&amp; object == _message) {
        NSLog(@"get %@",change);
    }
}
- (void) postMessage
{
    _message.key = @"asdfasd";
}
@end
~~~
完成一次完整的改变通知过程，经过以下几次过程:

#####1. 注册观察者[message addObserver:self forKeyPath:kKVOPathKey options:NSKeyValueObservingOptionNew context:Nil];

#####2. 更改主题对象属性的值，即触发发送更改的通知 _message.key = @"asdfasd";

#####3.在制定的回调函数中，处理收到的更改通知

~~~
- (void) observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
 if ([keyPath isEqual:kKVOPathKey] &amp;&amp; object == _message) {
     NSLog(@"get %@",change);
 }
}
~~~

#####4.注销观察者 [_message removeObserver:self forKeyPath:kKVOPathKey];

####KVO实现原理

一般情况下对于使用Property的属性，objc会为其自动添加键值观察功能，你只需要写一句@property (noatomic, assign) float age 就能够获得age的键值观察功能。而为了更深入的探讨一下，KVO的实现原理我们先手动实现一下KVO：

{% highlight ruby linenos %}
@implementation DZKVOManual
- (void) setAge:(int)age
{
    [self willChangeValueForKey:kKVOPathAge];
    if (age !=_age) {
        _age = age;
    }
    [self didChangeValueForKey:kKVOPathAge];
}
//经验证  会先去调用automaticallyNotifiesObserversForKey:当该函数没有时才会调用automaticallyNotifiesObserversOfAge。这个函数应该是编译器，自动增加的一个函数，使用xcode能够自动提示出来。的确很强大。
//+(BOOL) automaticallyNotifiesObserversOfAge
//{
//    return NO;
//}
+ (BOOL) automaticallyNotifiesObserversForKey:(NSString *)key
{
    if (key ==kKVOPathAge) {
        return NO;
    }
    return [super automaticallyNotifiesObserversForKey:key];
}
@end
{% endhighlight %}
首先，需要手动实现属性的 setter 方法，并在设置操作的前后分别调用 willChangeValueForKey: 和 didChangeValueForKey方法，这两个方法用于通知系统该 key 的属性值即将和已经变更了；

其次，要实现类方法 automaticallyNotifiesObserversForKey，并在其中设置对该 key 不自动发送通知（返回 NO 即可）。这里要注意，对其它非手动实现的 key，要转交给 super 来处理。

在这里的手动实现，主要是手动实现了主题对象变更向外广播的过程。后续如何广播到观察者和观察者如何响应我们没有实现，其实这两个过程apple已经封装的很好了，猜测一下的话，应该是主题对象会维护一个观察者的队列，当本身属性发生变动，接受到通知的时候，找到相关属性的观察者队列，依次调用observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context来广播更改。 还有一个疑问，就是在自动实现KVO的时候，系统是否和我们手动实现做了同样的事情呢？

####自动实现KVO及其原理

我们仔细来观察一下在使用KVO的过程中类DZMessage的一个实例发生了什么变化： 在使用KVO之前：

![](http://ww4.sinaimg.cn/large/7df22103jw1exwxex4ozvj20d204rt9f.jpg)

当调用Setter方法，并打了断点的时候：

![](http://ww1.sinaimg.cn/large/7df22103jw1exwxf3vu2rj20b3040dge.jpg)

神奇的发现类的isa指针发生了变化，我们原本的类叫做DZMessage，而使用KVO后类名变成了NSKVONotifying_DZMessage。这说明objc在运行时对我们的类做了些什么。

我们从Apple的文档[Key-Value Observing Implementation Details](https://developer.apple.com/library/mac/documentation/cocoa/conceptual/KeyValueObserving/Articles/KVOImplementation.html)找到了一些线索。

>Automatic key-value observing is implemented using a technique called isa-swizzling. The isa pointer, as the name suggests, points to the object’s class which maintains a dispatch table.This dispatch table essentially contains pointers to the methods the class implements, among other data. When an observer is registered for an attribute of an object the isa pointer of the observed object is modified, pointing to an intermediate class rather than at the true class. As a result the value of the isa pointer does not necessarily reflect the actual class of the instance. You should never rely on the isa pointer to determine class membership. Instead, you should use the class method to determine the class of an object instance.

当某一个类的实例第一次使用KVO的时候，系统就会在运行期间动态的创建该类的一个派生类，该类的命名规则一般是以NSKVONotifying为前缀，以原本的类名为后缀。并且将原型的对象的isa指针指向该派生类。同时在派生类中重载了使用KVO的属性的setter方法，在重载的setter方法中实现真正的通知机制，正如前面我们手动实现KVO一样。这么做是基于设置属性会调用 setter 方法，而通过重写就获得了 KVO 需要的通知机制。当然前提是要通过遵循 KVO 的属性设置方式来变更属性值，如果仅是直接修改属性对应的成员变量，是无法实现 KVO 的。

同时派生类还重写了 class 方法以“欺骗”外部调用者它就是起初的那个类。因此这个对象就成为该派生类的对象了，因而在该对象上对 setter 的调用就会调用重写的 setter，从而激活键值通知机制。此外，派生类还重写了 dealloc 方法来释放资源。

##拉模型

![](http://ww2.sinaimg.cn/large/7df22103jw1exwxg9a68nj20o50f1q4l.jpg)
拉模型是指主题对象在通知观察者的时候，只传递少量信息或者只是通知变化。如果观察者需求要更具体的信息，由观察者主动从主题对象中拉取数据。相比推模型来说，拉模型更加自由，观察者只要知道有情况发生就好了，至于什么时候获取、获取那些内容、甚至是否获取都可以自主决定。但是，却存在两个问题：

1. 如果某个观察者响应过慢，可能会漏掉之前通知的内容
2. 观察者必须保存一个对目标对象的引用，而且还需要了解主题对象的结构，这就使观察者产生了对主题对象的依赖。
可能每种设计模式都会存在或多或少的一些弊端，但是他们的确能够解决问题，也有更多有用的地方。在使用的时候，就需要我们权衡利弊，做出一个合适的选择。而工程师的价值就体现在，能够在纷繁复杂的工具世界中找到最有效的那个。而如果核桃没被砸开，不是你手力气不大的问题，而是你选错了工具，谁让你非得用门缝夹，不用锤子呢！

当然，上面那段属于题外话。言归正传，在OBJC编程中，典型的一种拉模型的实现是delegate。可能很多人会不同意我的观点，说delegate应当是委托模式。好吧，我不否认，delegate的确是委托模式的一种极度典型的实现方式。但是这并不妨碍，他也是一种观察者模式。其实本来各种设计模式之间就不是泾渭分明的。在使用和解释的时候，只要你能够说得通，而且能够解决问题就好了，没必要纠缠他们的名字。而在通知变化这个事情上delegate的确是能够解决问题的。

我们来看一个使用delegate实现拉模型的观察者的例子：

####1. 先实现一个delegate方便注册观察者，和回调函数

{% highlight ruby linenos %}
@class DZClient;
@protocol DZClientChangedDelegate &lt;NSObject&gt;
- (void) client:(DZClient*)client didChangedContent:(NSString*)key;
@end
@interface DZClient : NSObject
@property (nonatomic, weak) id&lt;DZClientChangedDelegate&gt; delegate;
@property (nonatomic, strong) NSString* key;
@end
{% endhighlight %}

####2. 注册观察者

{% highlight ruby linenos %}
//DZAppDelegate
DZClient* client = [DZClient new];
client.delegate = self;
client.key = @"aa";
{% endhighlight %}

####3. 当主题对象的属性发生改变的时候，发送内容有变化的通知

{% highlight ruby linenos %}

@implementation DZClient
- (void) setKey:(NSString *)key
{
    if (_key != key) {
        _key = key;
        if ([_delegate respondsToSelector:@selector(client:didChangedContent:)]) {
            [_delegate client:self didChangedContent:@"key"];
        }
    }
}
{% endhighlight %}

####4. 观察者收到主题对象有变化的通知后，主动去拉取变化的内容。

{% highlight ruby linenos %}
//DZAppDelegate
- (void) client:(DZClient *)client didChangedContent:(NSString *)key
{
    if ([key  isEqual: @"key"]) {
        NSLog(@"get changed key %@",client.key);
    }
}
{% endhighlight %}

##广义观察者模式

在上面介绍了，观察者被动的接受主题改变的经典意义上的观察者模式之后，我们再来看一下广义观察者模式。当然上面所讲的经典观察者模式，也是一种一种传递数据的方式。广义观察者涵盖了经典观察者模式。

往往我们会有需要在“观察者”和“主题对象”之间传递变化的数据。而这种情况下，主题对象可能不会像经典观察者模式中的主题对象那样勤劳，在发生改变的时候不停的广播。在广义观察者模式中，主题对象可能是懒惰的，而是由观察者通过不停的查询主题对象的状态，来获知改变的内容。

我们熟悉的服务器CS架构，始终比较典型的冠以观察者模式，服务器是伺服的，等待着客户端的访问，客户端通过访问服务器来获取最新的内容，而不是服务器主动的推送。

之所以，要提出广义观察者模式这样一个概念。是为了探讨一下观察者模式的本质。方便我们能够更深刻的理解观察者模式，并且合理的使用它。而且我们平时更多的将注意力放在了通知变化上面，而观察者根本的目的是在于，在观察者和主题对象之间，传递变化的数据。这些数据可能是变化这个事件本身，也可能是变化的内容，甚至可能是一些其他的内容。

从变化数据传递的角度来思考的话，能够实现这个的模式和策略实在是数不胜数，比如传统的网络CS模型，比如KVC等等。在这里就先不详细展开讨论了。