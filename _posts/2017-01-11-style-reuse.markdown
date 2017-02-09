---
layout: post
title: "iOS架构设计(解耦的尝试)之UI样式复用与布局管理"
date: 2017-01-11T16:29:13+08:00
categories: iOS
---

iOS架构设计(解耦的尝试)之UI样式复用与布局管理

> 该系列文章是2016年折腾的一个总结，对于这一年中思考和解决的一些问题做一些梳理和总结。Talk is cheap show me the code.

本来只是想写写ElementKit中对于MVVM的实践来着，结果发现这一年做的一些事情中，还有值的继续说的。而且基本上都是围绕着解耦和复用的主题。而且很多都是在日常的开发中常见的问题和其解决方案。也就继续写写，算是抛砖引玉，在闷头设计开发之后与业界交流一下。

这一次来说一下在日常的开发中最常见的两个问题：

1. 一个是样式，颜色、背景、font。。。。
2. 一个是布局管理

其中的成果自然也是搞成了类库方便使用（Talk is cheap show me the code.）：[StyleSheet](https://github.com/yishuiliunian/StyleSheet)和[DZGeometryTools](https://github.com/yishuiliunian/DZGeometryTools)。

样式和布局基本上是在iOS编程中最繁重的两个工作，之所以说繁重，

1. 一是因为常见，绝大多数iOSCoder每写一个业务逻辑，都得和这两个事情纠缠上半天，去还原UI/UE同学的设计。
2. 二是因为UI样式千变万化很难像其他业务逻辑那样用类继承了之类的策略来处理。每次都得蛮干。

再一次提一句话『懒是第一生产力』，有压迫的地方就有反抗，有繁重的地方就想着偷懒。所以得需要一个有效途径来降低UI工作的繁重。

## 抽象的威力

其实我一直希望自己的文章能够道业双关。在说明一个问题的解决方案的同时，也能够解释清楚自己设计一个类库或者做一个技术决定背后的东西：知识背景和思维方法。还原整个从问题发现到问题解决方案的过程。所以，发现之前的几篇文章中多多少少有一些东扯西扯的东西。自然这篇文章也难免其俗:)。这一次要扯的是抽象。之所以要说这个是因为，在解决UI工作繁重的问题中，就是用的这个最基本的方法。

> 删繁就简三秋树，霜叶红于二月花

从小老师就耳提面命：要看透问题的表象看本质。起初不解，随着干Coder时间长了，愈发察觉要想偷懒就得去解决本质问题，而不是围绕着问题的表层转来转去。而进一步说，不能停留在使用简单抽象出来的概念解决问题的层次，应该进一步使用一些高级抽象。就拿布局这个事情来说吧。

### 原始抽象

第一层的抽象是对现实世界在数字世界的描述，用坐标系统。用数学语言来描述布局这个事情。其实抽象到这一步也是一个非常大的进步了。在iOS中也就是我们常用的spring&struct的布局系统。通过frame等来标注一个view来控制平面布局，通过View-Tree来控制Z轴方向上的布局。

基于这种抽象的情况下，我们做的事情就是精准的描述每一个View在坐标系统中的位置。这个就是那个繁重的工作。在`layoutsubviews`函数里面里面，各种计算大小和偏移量。熟悉iOS的同学，随意看一下自己View里面`layoutSubviews`函数的大小就可以感受到其繁重。

### 描述抽象

抽象这个东西有意思的地方就在于他是可以递归使用的。在第一层抽象得到的概念基础上，进行第二层抽象；在第二层抽象得到的概念基础上，进行第三层抽象.....如此反复。当然，Apple牛逼的工程师们也会意识到spring&struct的繁重，于是就有了Autolayout（自动布局系统）。

Autolayout是基于描述的抽象，更多的是描述元素与元素之间的位置关系。通过解n元一次方程组来确定元素的位置（在坐标系统中的frame)。而这种基于模式的抽象，要比刚才第一层的抽象要高级很多。在这种抽象概念下，我们发现我们写的布局的代码有了一个大幅度的减少。原先需要拼了老命计算view frame的情况变成了，描述view之间关系的过程。原先需要非常多代码才能完成的任务，现在只要简单的写一句AView在x轴方向上距离BView固定间隔为10就可以完成了。随着代码量的降低，我们的对于布局系统理解成本都在降低。

> 复杂性由固有复杂性和认知复杂性组成。

而基于描述的抽象，因为是对第一层抽象概念的操作，忽略了第一层抽象的很多细节，很大程度上降低了布局系统的认知复杂度。无论从代码量上还是我们的认知成本上，都让我们可以『偷懒』了。

### 任务抽象

而有了基于描述抽象的Autolayout之后，我们会发现其实很多任务还是很繁重，比如我们要写个List布局，不可能每次都把List中每个元素的位置关系描述一边啊。比如我们要写个Collection的布局，也不可能每次都把每个元素的位置关系都描述一边啊.....

于是真对某些特定任务，会使用任务抽象。把List布局抽象成UITableView，把Collection布局抽象成UICollectionView，把线性布局抽象成UIStackView。。。。。而且Apple的工程师们也会在这条路上越走越远。在近几个版本的iOS-SDK更新中，我们也看到了更多的这种布局View的出现。相信以后也会更多。

而这种类型的View在简化编程工作这件事情上要比autolayout来的更加厉害。可以随意感受一下，我原先为了实现一个TableView所写的代码量[DZTableView](https://github.com/yishuiliunian/DZTableView)。就知道当我们把通用任务抽象出来的时候，能偷多大的懒了。

而关于这种任务抽象我们听到的最多就是面试中经常被问及的GoF设计模式。在想想应用设计模式时的爽，也就知道抽象这个工具的确很好用。在这里强烈推荐一本书[《元素模式》](https://item.jd.com/11500855.html)。个人认为这本提供了一整套的瑞士军刀，来帮你进行抽象或者去设计『设计模式』。

### ....

当然这里还会存在更高层次的抽象。不过嘛，抽象并非银弹。并不是说一味的抽象下去就能够得到极致的『偷懒』。随着抽象层次提高，概念密度也在提高，而那些在这个过程中所忽略的特殊场中的细节，将变得难以还原。有些时候，处理问题反而更加困难，编码量却在增加。老生常谈的问题啊：度。当抽象层次满足业务需求和业务发展的时候，就可以临时先止步了。




## DZGeometryTools

简单描述了一下抽象的威力之后，开始切入如何使用部分。顺着刚才的话题讲布局的事情。我自己写界面的时候也是尽可能的"偷懒"。于是就有了这个库[DZGeometryTools](https://github.com/yishuiliunian/DZGeometryTools)。主要是用了描述抽象和任务抽象两种技术。看起来忽悠人，其实实现部分一看就明白了。无非是将常用的一些任务转换成了函数而已。其实在`CoreGraphics`框架中又一个`CGGeometry.h`文件，其中提供了很多方便操作CGRect等几何类型的函数。而在DZGemetryTools中所做的工作可以看做是对CGGeometry的一个扩展。

### DZGeometryTools.h中主要是对几何类型的操作

通过抽象我将比较常见的几何操作归结为以下几种比较基础的操作：

1. 偏移操作 
2. 缩放操作
3. 间距计算操作 
4. margin施加操作

现在把他们提取出来，基本上就是完成了一个工具集的构建。而后在这些工具集上，就可以来描述每个Rect之间的关系，这个有点类似于autolayout，不过是提前收工算好了，而autolayout是自动计算并赋值的。贴段真实使用中代码来感受一下：

~~~
- (void) layoutSubviews
{
    [super layoutSubviews];
    CGSize imageSize = {62*2, 84};
    CGRect contentRect = CGRectCenterSubSize(self.bounds, CGSizeMake(20, 20));
    CGRect textRect;
    CGRect imageRect;
    
    CGRectDivide(contentRect, &textRect, &imageRect, 30, CGRectMaxYEdge);
    imageRect = CGRectCenter(imageRect, imageSize);
    
    CGRect imgRs[2];
    CGRectHorizontalSplit(imageRect, imgRs, 2, 0);
    _indicatorImageView.frame = imgRs[0];
    _powerImageView.frame = imgRs[1];
    _textLabel.frame = textRect;
    _backgroundView.frame = self.bounds;
    _lastTimeLabel.frame = imageRect;
}
~~~

其中使用到了`CGRectCenterSubSize `来对contentRect做了margin计算，并用`CGRectHorizontalSplit `对rect坐了纵向均分的操作，这些都是描述UI元素之间相对位置关系的关系，我们通过手工计算来完成了对于UI元素的布局操作。这些函数都没有采用直接操作view.frame的方式，只进行了几何运算，计算出了UI元素坐标。之所以采取这种方式，是因为想基于目前的抽象层次应该是针对于几何概念的操作。对于UI元素的操作，已经简化成了一个赋值操作，没有太大必要去优化处理了。而相较于以前的编码量和思维量来说，基于目前构建的这个工具集来进行编码已经省了不少力气了。体力活少了。

### DZLayoutMacros.h 常用的任务

在这个文件里面依旧是一些常用的布局工具集，不过和上面不一样的是采用了另外的表达方式:`宏`。针对于一些比较简单的任务，做了抽象。比如：

1. y依赖于顶部元素，并且尽可能填充满width的布局
2. 顶部固定高度，铺满width的布局
3. //底部固定高度，铺满width的布局
4. .....

### 小结

这些函数是我在日常的编码中抽象出来的一些比较基础的工具。没有完整的模型，是比较零散的收集吧，更多的东西还是看库中代码的注释[DZGeometryTools](https://github.com/yishuiliunian/DZGeometryTools)。这是个遗憾，没有统一的模型。一直都想搞一个类似于autolayout或者flexbox之类的布局系统，能够更加方面的来完成布局的操作。这个工作看来得放到17年做了，惟愿完成。



## StyleSheet


据说一个终端开发人员将会有70%以上的时间在和UI打交道。自己想想也对，貌似有很大一部分时间花费在了调整UI样式，addSubView还有layout上面。猛然间就发现自己的代码中有大量这种东西存在

~~~
    self.label.layer.cornerRadius = 3;
    self.label.textColor = [UIColor darkTextColor];
    self.label.font = [UIFont systemFontOfSize:13];
    self.label.backgroundColor = [UIColor greenColor];
    self.label.layer.borderWidth = 2;
    self.label.layer.borderColor = [UIColor redColor].CGColor;


    self.label2.layer.cornerRadius = 3;
    self.label2.textColor = [UIColor darkTextColor];
    self.label2.font = [UIFont systemFontOfSize:13];
    self.label2.backgroundColor = [UIColor greenColor];
    self.label2.layer.borderWidth = 2;
    self.label2.layer.borderColor = [UIColor redColor].CGColor;


    self.button.layer.cornerRadius = 3;
    self.button.backgroundColor = [UIColor greenColor];
    self.button.layer.borderWidth = 2;
    self.button.layer.borderColor = [UIColor redColor].CGColor;

    self.aView.layer.cornerRadius = 3;
    self.aView.backgroundColor = [UIColor greenColor];
    self.aView.layer.borderWidth = 2;
    self.aView.layer.borderColor = [UIColor redColor].CGColor;
    ......
    
~~~
 
上面的代码是为了实现这样的效果而写的代码。

很多几乎是一毛一样的代码，充斥着整个APP。自己花在这些样式调整上的时间也非常多。为了实现一个样式效果，需要配置各种各样的属性。而且很多界面中这些样式都是一样的。于是又是无数次的重复上面的工作。oy my god！时间啊，就这样流走了。做为一个懒人，就会发问有没有一种可以少写点代码的方式呢？你可以写一个子类嘛，但是会有类污染的问题，单纯为了一个公有样式，就创建个子类有点大材小用。那写一批样式渲染的函数呗，恩这个注意不错，但是细想一下工作量也不小，而且不通用。于是，花了几天的时间我写了StyleSheet这个库。为了的就是来简化UI样式的编码。

通过上述描述我们可以发现，原始的写UI样式的问题：

1. 繁琐的代码，大量重复性的工作
2. 样式无法共享，每一个View都需要重新进行样式赋值。

而StyleSheet的设计目标就是：

1. 样式配置轻便化，能够使用更加少的代码来描述View的样式
2. 样式在View之间的共享.不止是相同类的实例之间的共享，甚至是跨类的共享。

So，先看看上述代码使用StyleSheet之后的效果：

~~~
self.label.style = 
DZLabelStyleMake(
  style.backgroundColor = [UIColor greenColor];
  style.cornerRedius = 3;
  style.borderColor = [UIColor redColor];
  style.borderWidth = 2;
  style.textStyle.textColor = [UIColor darkTextColor];
  style.textStyle.font = [UIFont systemFontOfSize:13];
);
self.label2.style = self.label.style;
self.aView.style = self.label.style;
[self.button.style copyAttributesWithStyle:self.label.style];
~~~

### 设计与使用

基础抽象模型很简单，就是要让界面上关于展示的属性可以被组合使用。而我们所谓的样式，其实也就是各种基础属性组合出来的结果。基于这个模型，在设计StyleSheet的时候故意淡化了被渲染的View的类型的概念，任何一种类型的Style可以对任何类型的View进行渲染，但是必须是这种类型的View支持Style所指称的属性。比如你可以使用真对Button设计的DZButtonStateStyle来渲染一个UILabel，但由于UILabel不支持DZButtonStateStyle中的渲染属性，所以渲染结果是无效的。

但是当使用DZButtonStyle(继承自DZViewStyle)来渲染UILabel的时候，会使用DZButtonStyle中其父类的某些渲染属性，来渲染UILabel的父类UIView所支持的那些属性。

#### 使用

##### 直接使用Style对View进行渲染：

~~~
DZLabelStyle* style =
DZLabelStyleMake(
    style.backgroundColor = [UIColor greenColor];
    style.cornerRedius = 3;
    style.borderColor = [UIColor redColor];
    style.borderWidth = 2;
    style.textStyle.textColor = [UIColor darkTextColor];
    style.textStyle.font = [UIFont systemFontOfSize:13];
);

[style decorateView:self.label];
~~~

直接渲染的好处是，不用再次生成Style对象，更加方便样式在多个View之间渲染。
##### 赋值渲染
对UIKit中常用的一些组件进行了扩张为他们增利了style属性，直接进行style属性的赋值，会出发一次渲染操作。当第一次调用style属性的时候，会自动生成一个zeroStyle并赋值。
self.label.style = style;
或者
self.label.style = DZLabelStyleMake(
    style.backgroundColor = [UIColor greenColor];
    style.cornerRedius = 3;
    tyle.borderColor = [UIColor redColor];
    style.borderWidth = 2;
    style.textStyle.textColor = [UIColor darkTextColor];
    style.textStyle.font = [UIFont systemFontOfSize:13];
);
当进行赋值渲染的时候，会将Style的Copy后的实例与当前View绑定，当更改Style的属性的时候，对应View的样式会立刻改变。

### 通用样式的共享
使用原有的配置，进行通用样式的共享是个非常困难的事情，基本上都是体力活，靠人力来维护。我们的代码中会掺杂大量的用于配置样式的代码，而且是独立且散在。
现在你可以通过StyleSheet解决：
定义共享的样式：

~~~
//在头文件中使用 xxx.h 声明一个公有样式
EXTERN_SHARE_LABEL_STYLE(Content)

//在实现文件中使用 xxx.m ，实现一个公有样式
IMP_SHARE_LABEL_STYLE(Content,
   style.backgroundColor = [UIColor clearColor];
   style.cornerRedius = 2;
   style.textStyle.textColor = [UIColor redColor];
)

~~~

（1）使用共享样式,方式一
self.label.style =  DZStyleContent();

（2）使用共享样式，方式二（推荐）
很多时候, 如果不需要进一步更改样式,可以不采复制赋值的方式来进行渲染，可以直接使用：
[DZStyleContent() decorateView:self.label];
只进行渲染，而不进行复制。
好了，现在可以尝试着换这种方式来写UI样式了。




