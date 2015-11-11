---
layout: post
title: "有序构建之概念篇"
date: 2015-11-11T01:17:42+08:00
---

有序构建
====
我们应该都有过吃自助的经验，有些自助餐厅，果盘免费，但是只给一个盘子，你能装多少就是多少。所以这个时候，拼实力的时候到了。

一般的入门级的选手是这样的：

![](http://ww1.sinaimg.cn/large/7df22103jw1exwdwim43pj20850643ym.jpg)

水平稍微高一点的是这样的：

![](http://ww3.sinaimg.cn/large/7df22103jw1exwdwrm88ij20rq0ns41g.jpg)

但是，这个时候高手来了：

![](http://ww3.sinaimg.cn/large/7df22103jw1exwdx5hznpj208c0b5752.jpg)

最后的高手的确惊艳，但是他是怎样把果盘垒成这个样子的——[果盘堆砌之法](http://www.blogbus.com/paradise-sugar-logs/27274140.html)。

简单说就是：

1. 内圈抹平摆整齐以后再在外圈（胡萝卜条上）再整齐地摆上一圈菠萝。这步是第一层地基，一定要保证整齐，侧面看要正！！
2. 再放一层黄桃
3. 在黄桃的外面，外层菠萝的上面堆上黄瓜。为下一层菠萝做准备
4. 黄瓜放好后在表层撒点玉米粒火腿肠之类的小东西以使表面平一些
5. 再在黄瓜上堆一层菠萝
6. 如此往复，能堆多高，就要看你的功力了。

从这个过程中我们发现了什么：**我们吃过盘是有预谋的，而且是有预谋，有计划！**。目标是尽可能的堆更多的水果，计划是打地基、筑结构，多填充。这是典型的有序构建的过程。相比较一般入门级的人来，我们能够明显的看出，入门级的选手果盘堆得杂乱无章，完全没有秩序。他们要么是不在意能装多少水果，要么就是无计划，随意堆砌。

而所谓的有序构建就是：
在明确构建目标的前提下，按照一定的规则来构建。

这个和架构的定义多少有些类似，只是架构的观念太大。一提起架构，脑子里面往往会冒出一些非常宏大的概念，比如三层架构，CS架构等等。而这里我们需要一个小的很多的概念，来指导日常的编码。一个小需求，我们可能不需要去设计那么宏大的架构。但是我们必须保持我们程序的构建过程有序。我们要明确我们的需求，这个需求可能是产品侧的，也可能是性能的，也可能是来自第三方使用者的。之后我们需要按照一定的规则或者秩序去构建我们的程序。

这些规则可能是软件工程层面的：瀑布流模型、螺旋上升模型、敏捷开发之类，也可能是编程范式：面向对象、切面范式、函数式、命令式之类的语言思想，也可能是设计模式之类的前人总结的最佳实践，也可能是一些我们通常挂在嘴边的一些经验：死程序不说谎，尽可能的减少重复之类，甚至我们在一起开发某个App的同学们自己一起制定的编码规范都是规则。这里有楼主之前写过的一篇关于编程思想的文章，里面也有不少规则:[关于程序设计和思维的思考](http://km.oa.com/group/22128/articles/show/185307)。

那么问题来了：在真正日常的编码中，我们到底要遵循哪些规则去Coding？又有哪些规则使我们可以借鉴的?

其实你使用了哪些规则去Coding并不关键，最关键的是要有规则，断不可无序构建，让代码有序生长就好了。而规则各有优劣，又繁复庞杂，反正我是说不清楚应该用哪个不应该用哪个。只能是根据具体的业务与场景选择一个，并且尽可能的让其他人能够很容易的看出我使用了什么。之前和一个同事讨论过一个问题，在写某一个需求的时候，可以不用设计模式简单粗暴的去写，也可以使用策略模式，他说：简单粗暴的写，省时省力，用个设计模式以后又不一定会扩展，干嘛费哪个劲，我说：用个策略模式，别人看一眼代码就知道，你是按照什么思路写这个代码的了啊，他看起来改起来要省事多了。代码很多时候，写了不止是为了满足眼下的需求，也有很多是为了未来维护。而按照一定的规则和秩序去写，去有序构建，则能让代码有序生长。

下面是开脑洞的时间，之说IOS开发相关的东西。因为楼主目前是搞这个的。说一些在IOS上我们可能会用到的一些构建的规则。一般的东西比如：面向对象、敏捷开发、设计模式等等这些耳熟能像的，大家比我懂得多，我就不班门弄斧了。我们来看一些，没有那么高大上，但是非常实用的一些构建的规则。只是为了脑洞大开。

#语言特性
##动态语言
###切面范式

什么是切面范式？
如果一个程序是一个管道系统，AOP就是在管道上钻一些孔，在每个孔中注入新的代码流。因此AOP实现的关键是将advice的代码嵌入到主体程序之中，术语称编织（weaving）。这是很自然的——将问题分解之后再合成，问题才得以还原。编织可分两种：一种是静态编织，通过修改源码或字节码（bytecode）在编译期（compile-time）、后编译期（post-compile）或加载期（load-time）嵌入代码——请注意，这里涉及到刚才提到的元编程和产生式编程；另一种是动态编织，通过代理（proxy）等技术在运行期（run-time）实现嵌入。具体的工具包括一些扩展性语言如AspectJ、AspectC++等和一些框架如AspectWerkz、Spring、Jboss AOP等。

在Java中切面范式可能比较常见，但是在OC中貌似比较少用。而往往，很多时候这种东西能够解决一些很大的问题。我们现在有这样的一个业务需求，就是用于提醒的各种样式的小红点。他可能出现在各式各样的地方上。比如“动态”tab里面的cell上面，在抽屉里面的cell上面，钱包里面的View上面......而这些cell或者view所属的类，繁复而众多。比如：QQTableViewCell，UITableViewCell,UIButton.....而我们的任务是给这些类的LayoutSubview方法中加上红点的布局信息，并在其他相关的地方加上红点相关的业务逻辑。
无疑这里按照面向对象的思路去分析，最常见的一个策略就是让这些类都继承自同一个父类，然后在父类中添加相关的业务逻辑。但是这个方案的问题是，改动量太大，需要所有让所有使用的红点的地方都进行修改实现类继承。这个改动量不是一朝一夕能改完的，而且后续的需求也得注意类继承时候的一些问题。同时，已经有很多业务按照最笨的每个地方自己布局的方式接入到了红点系统中，让他们重新来过也是件很痛苦的事情。

那么问题来了：有没有一种方式可以做到：

1. 对现有的代码影响最小，可以删但是不要改和增加代码。
2. 对以后的业务逻辑扩展方便。包括引入新的红点类型和其他业务接入红点系统。
3. 不影响现有的类结构，而且能够让红点相关的业务逻辑对使用者透明。

如果我们一直按照面向对象的方式去思考，那么这个问题会看起来很困难。何不换个思路，既然不让影响类结构，那么我能不能做到，动态注入业务逻辑呢？而这就是切面范式要做的事情，在现有代码的基础上，从某个关注点就将代码嵌入到主体程序之中。

我们就是以在layoutSubviews里面进行红点布局为关注点，进行编织。通过isa-swzzing和method-swizzing这两种得力于OC动态语言的技术来实现编织过程。最终实现了我们刚刚说的几个要求，具体的实现参见：[红点系统UI层的重构](http://km.oa.com/group/21772/articles/show/196667)。

在使用切面范式之后，某个Cell接入红点就无需继承UItableViewCell，然后在子类Cell和UITableViewController里面写红点的业务逻辑。以前完全无序的业务方的代码，可以省去，在重构后的UI框架的控制之下逐步构建。

可能是以前接触到的编程思想大都基于OOP，导致在思考问题的时候，总是喜欢从OO出发。而OO并非万能的神药，某些时候换个角度，或者换种思路来思考问题，或许有不一定的解题之道。而切面范式的构建秩序与OOP明显不同，一种是从面到点，一种是从点到面。切面范式给我提供了一种能够从软件的某个功能横切面进行构建的规则。而这种规则在某些时候恰恰能够弥补OO的不足之处。

###元编程

元编程是一个很有意思的概念，简单说就是用程序来写程序。颇有科幻电影里面，机器有了智能能够自我编程的意味。然而，其实不然：

```
元编程是指某类计算机程序的编写，这类计算机程序编写或者操纵其它程序（或者自身）作为它们的数据，或者在运行时完成部分本应在编译时完成的工作。多数情况下，与手工编写全部代码相比，程序员可以获得更高的工作效率, 或者给与程序更大的灵活度去处理新的情形而无需重新编译。
```



元编程在很多动态语言中非常常见，比如Ruby中最出名的rails框架就大量使用了元编程。不过元编程在里面有了另外的一个名字——DSL，领域特定语言。而如C++之类的静态编译性的语言借助于模板也可实现元编程[C++模板元编程](http://book.douban.com/subject/4136223/)。那么对于OC或者IOS呢，元编程是否有些应用？

1. lua虚拟机操作OC
2. 调试界面的一些工具，比如reveal
3. .....

其中lua虚拟机操作OC的框架[WAX](https://github.com/probablycorey/wax)在手Q中已经应用。其主要思路是，加载一个lua虚拟机执行lua代码，并通过WAX的OC<->lua的双向桥接实现使用lua来控制OC的目的。而能够实现这种使用lua控制OC的元编程也是得益于OC的动态特性。关于更详细的WAX框架信息，可以看一下这片文章的总结[LUA化总结](http://km.oa.com/group/21772/articles/show/205565?kmref=search)。

元编程是另外的一种构建规则，我们提供基础的构建规则之后，让程序按照既定的规则来构建另外的程序。



##Block闭包

什么是block？
Block是C语言的扩展功能，可以用一句话来表示Blocks的带来的扩充功能：带有自动变量的匿名函数。有些地方称之为lambda表达式或者闭包。IOS4.0之后引入了闭包的功能。而上文提到的WAX框架的作者当时写WAX的初衷就是为了给OC引入这个特性。而我们在另外一些现代语言中，频频见到闭包的影子，在ruby中，在swift中，在js中......那么闭包为什么会这么有吸引力呢？因为闭包是函数式编程的基础要素之一。(BTW:关于apple如何实现的闭包，可以参考[《Objective-C高级编程》](http://item.jd.com/11258970.html?utm_source=www.googleadservices.com&utm_medium=tuiguang&utm_campaign=t_50_s_sda1&utm_term=802e7d6365a04596b8b70cbf7ad6a221)第二章，blocks)。

###基本用法
###函数式编程

> 函数式编程（英语：Functional programming）或者函数程序设计，又称泛函编程，是一种编程范型，它将电脑运算视为数学上的函数计算，并且避免使用程序状态以及易变对象。函数编程语言最重要的基础是λ演算（lambda calculus）。而且λ演算的函数可以接受函数当作输入（引数）和输出（传出值）。
> 
> 和命令式编程相比，函数式编程强调程序的执行结果比执行过程更重要，倡导利用若干简单的执行单元让计算结果不断渐进，逐层推导复杂的运算，而不是设计一个复杂的执行过程。
> ————————出自[WIKI](http://zh.wikipedia.org/wiki/%E5%87%BD%E6%95%B8%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80)

这是一种古老而又现代的编程范式，他所提供的构建规则，真是历久弥新。从古老如lisp，到心如swift之类的语言中，我们都能看到函数式编程的影子。有意思的是，在最近的一段的时间内不同各种主流语言都在加入这种构建规则，oc的blocks，switf原生支持，golang也是.......在多核的时代，函数式编程又生机勃发。还要多说句题外话，swift这门语言都说是OC的进化版和替代者，我是从函数式编程这个角度看出来的。OC可以实现函数式编程但是，会显得略微笨拙，而swift实现起来则是灰常优雅。写obc.io的那群人写了一本书[Functional Programming in Swift](http://www.objc.io/books/)，非常详细的阐述了函数式编程在swift中的应用。

言归正传，我们来看一下在OC中实现的函数式编程的案例。现在我们有这样的一个功能要实现，

![](./QQ20141126-1.png)

横向布局一些图片和文字，而且文字的颜色不同，点击之后还会有效果。你可能会说使用NSAttributeString，但是考虑到兼容性的问题，这个方案临时不考虑。我们就用最笨的方法展示：用UIImageView展示图片，用UILabel展示灰色的文字，用UIButton展示可点击的文字。那么问题来了，在给这些元素布局的时候，我们应该怎样code？一个一个的算frame然后赋值吗，那么代码将会丑陋不堪。这个时候何不尝试一下函数式编程。

```
 CGFloat offsetY = IS_IPHONE5 ? 80 : 50;
    
    CGFloat textOffsetY = CGRectGetMaxY(_urlImageView.frame) + offsetY;
    
    typedef UILabel*  (^GetLabel)(UIView* aView);
    void (^SetFameWithLabel)(CGFloat x, UIView* aView, GetLabel block) = ^(CGFloat x, UIView* aView, GetLabel block) {
        
        UILabel* label = block(aView);
        
        NSString* str = label.text;
        CGSize size = [str sizeWithFont:label.font];
        aView.frame = CGRectMake(x,
                                 textOffsetY,
                                 size.width, CGRectGetHeight(_checkButton.frame));
        
    };
    
    GetLabel lableBlock =  ^(UIView* aview) {
        return (UILabel*)aview;
    };
    
    
    GetLabel buttonBlock  =  ^(UIView* aview){
        UIButton* button = (UIButton*) aview;
        return button.titleLabel;
    };
    
    _checkButton.frame = CGRectMake(15, textOffsetY, 30, 30);
    SetFameWithLabel(CGRectGetMaxX(_checkButton.frame), _headLabel, lableBlock );
    SetFameWithLabel(CGRectGetMaxX(_headLabel.frame), _serviceContractButton, buttonBlock);
    SetFameWithLabel(CGRectGetMaxX(_serviceContractButton.frame), _tailLabel, lableBlock);
    SetFameWithLabel(CGRectGetMaxX(_tailLabel.frame), _userageButton, buttonBlock);
```

我们构建了几个基础的block，然后用这几个block之间的组合完成了上述的功能。而这正是函数式编程的精髓：利用若干简单的执行单元让计算结果不断渐进，逐层推导复杂的运算，而不是设计一个复杂的执行过程。而这个思想同样可以应用在我们日常的编程之中——保持代码的简洁，尽量使用简单的规则祖曾退到负责的运算，而不是一上来就构建一个复杂的系统。

####函数响应式编程

函数响应式编程（RF）是最近一场火爆的一个编程范式。RP提高了代码的抽象层级，所以你可以只关注定义了业务逻辑的那些相互依赖的事件，而非纠缠于大量的实现细节。RP的代码往往会更加简明。
特别是在开发现在这些有着大量与Data events相关的UI events的高互动性Webapps、Mobile apps的时候，RP的优势将更加明显。10年前，网页的交互就只是提交一个很长的表单到后端，而前端只有简单的渲染。Apps就表现得更加的实时了：修改一个表单域就能自动地把修改后的值保存到后端，为一些内容"点赞"时，会实时的反应到其它在线用户那里等等。

现在的Apps有着大量各种各样的实时Events，以给用户提供一个交互性较高的体验。我们需要工具去应对这个变化，而RP就是一个答案。

在IOS中的使用案例：

[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)

RF一直都缺少比较好的学习资料，终于一个国外的大神忍无可忍，自己动手写了一个：
[The introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)
[中文版](https://github.com/benjycui/introrx-chinese-edition)

##混编

###与C混编

####宏

使用宏定义来简化输入，提高输入的效率。同时提高输入准确性。

#####案例一 属性定义 @property

我们在定义一个类的属性的时候，

```
@interface TestObject : NSObject
@property (strong, nonatomic) NSString* title;
@end
```

最起码要输入5个单词，四个符号和多个空格。写多了就会觉得这里重复输入的地方太多，为什么不想个办法优化一下输入呢。而且有些时候，中间的某个单词比如strong拼错了，还得会过头来继续修改。
优化输入效率，有很多种方式。比如使用sinepts。而且xcode的snip支持也不错。直接拖拽代码块就能够生成snip。


```
\\key is
\\@propertystrongnonatomic
@property (strong, nonatomic) <#type#>* <#name#>
```

这样的确是可以，但是你需要定义大量的snip来适应不同的定义peroperty的情况。那有没有更简单的一点的方法呢。必须有啊，使用宏啊。


```
#define DEFINE_PROPERTY(mnmKind, type , name)       @property (nonatomic, mnmKind)  type  name
#define DEFINE_PROPERTY_ASSIGN(type, name)          DEFINE_PROPERTY(assign, type, name)
#define DEFINE_PROPERTY_ASSIGN_Double(name) DEFINE_PROPERTY_ASSIGN(double, name)

#define DEFINE_PROPERTY_STRONG(type, name) DEFINE_PROPERTY(strong, type, name)
#define DEFINE_PROPERTY_STRONG_NSString(name) DEFINE_PROPERTY_STRONG(NSString*, name)
```

具体参见[DZProgrameDefines](https://github.com/yishuiliunian/DZProgrameDefines)
我们完全可以通过使用宏定义，来扩展出一些列的定义属性的宏方法，借助于XCode的强大的自动补全来方便我们输入，少敲了非常多的字符。并且还减少了出错的情况，在可读性上，如果宏定义的名字起得好，可读性也不错。

同时，不得不说的一点是我们借助于这种宏定义的方式，还规范和统一了定义属性的格式，方便维护同一个工程的多个同事修改同一份代码。让他们的代码质量能够保持在一个比较整齐的水平。
这种

####代码模板

某些情况下，我们可能会写一些大量的重复代码，而这些代码又很难将其抽离出来做成一个独立的函数（甚至是lambda表达式），而这种时候宏的作用就体现出来了。考虑下述情况：

```
NSString* a = infos[@"aKey"];
if(!a) {
        [self postError:@"need aKey"];
        return;
}

NSString* b = infos[@"bKey"];
if(!b) {
        [self postError:@"need bKey"];
        return;
}
....
```

上述代码中，我们需要从字典infos中取一批参数并且要判断这些参数是否为空，为空的时候报错并返回。其中有大量的代码是重复的。而这种重复又不太适合抽离成函数那么这个时候就可以这样做了：

```
#define GetValueWithLocalNameAndKey(name , key) \
\
NSString* name = [infos getWBValueForKey:key error:&error];\
if (!error) { [self postPayError:error]; return;}\
\
...
GetValueWithLocalNameAndKey(a,aKey);
GetValueWithLocalNameAndKey(b,bKey);
...
```

这样做的时候，就将一段代码抽离成了模板。方便了使用和维护。

###与C++混编

其实这个比较大的话题，OC中与C++混编使用，往往是因为C++的一些优秀的特性。当然这里有很多，我们也只能撷取一二。在code的时候，往往我们需要考虑我们需要用哪种语言来表达我们对于需求的认知：OC or c++ or c？这个时候，能帮助我们做最后决定的往往是一些简单的规则，比如保持代码有序，最大复用原则等等。

####模板

使用过JEC与后台通讯的同学可能有过，写接口的痛苦。构建发送包，接口回调消息，解析回调，然后向外传递内容。这个过程，是多么的类似啊。那么问题来了，有没有什么方法可以简化这个过程中的编码，让大多数接口通信过程开起来更有序呢？何不使用模板呢。

```
template<typename T>
inline const int decodeForResponse(T& response , unsigned char* pData ,int nDataLen ,std::string key)
{
    .....
}

template<typename T>
inline const int getDecodedRespnse(T& response ,
                                    unsigned char* pData ,
                                  int nDataLen ,
                                  std::string key,
                                  int seq,
                                  WBServiceCenter* self) {
    ......
}

template<typename T>
inline int sendRequest(std::string funcName,
                        std::string servantName,
                        const std::string& dataKey,
                        const T& data,
                        WBServiceResponseBlock responseBlock,
                        WBServiceErrorBlock errorBlock,
                        WBServiceCenter* self,
                        NSString* cmd)
{
  .....
}
```

上述的过程中，把一直在变动的结构体，当成模板参数传进来，不就省事了很多嘛。（BTW：这里使用了一个重定义self的技巧。）

#改造

这里还有一个比较有意思的想法，将Ruby的一些语法特性以语法糖的形式提供在了OC中。可以借鉴一下，很多时候一些优秀的东西是可以通用的。

[ObjectiveSugar](https://github.com/supermarin/ObjectiveSugar.git)

#总结

上面只是在使用OC的过程中，总结出来的一些开脑洞的使用方式。而且这些方式，从某些意义上能够提高整个程序的质量，甚至在某些时候能够带来不一样的技术解决方案。本来，一直以为OC是一门神奇的语言，直到遇见了swift。上文中，提到的很多的构建方式，swift原生支持，函数式编程更加便捷，模板也不用和C++混编。这从另外一个角度，印证了swift的确是OC的升级版。
言归正传，所谓有序构建其中的“序”，大家理解不一。但是，可能都同意的是，软件的构建过程，代码的增长过程应当保持一定的秩序和规则。那么你在coding的时候将会遵循什么样的“有序构建”？



