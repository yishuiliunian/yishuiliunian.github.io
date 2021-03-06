---
layout: post
title: "使用宏来减少代码重复"
date: 2015-11-11T21:14:06+08:00
categories: iOS
---


使用宏定义来简化输入，提高输入的效率。同时提高输入准确性。

案例一 属性定义 @property

我们在定义一个类的属性的时候，

~~~
@interface TestObject : NSObject

@property (strong, nonatomic) NSString* title;

@end
~~~

最起码要输入5个单词，四个符号和多个空格。写多了就会觉得这里重复输入的地方太多，为什么不想个办法优化一下输入呢。而且有些时候，中间的某个单词比如strong拼错了，还得会过头来继续修改。 优化输入效率，有很多种方式。比如使用sinepts。而且xcode的snip支持也不错。直接拖拽代码块就能够生成snip。

~~~
\\key is

\\@propertystrongnonatomic

@property (strong, nonatomic) <#type#>* <#name#>
~~~

这样的确是可以，但是你需要定义大量的snip来适应不同的定义peroperty的情况。那有没有更简单的一点的方法呢。必须有啊，使用宏啊。

~~~
#define DEFINE_PROPERTY(mnmKind, type , name)       @property (nonatomic, mnmKind)  type  name

#define DEFINE_PROPERTY_ASSIGN(type, name)          DEFINE_PROPERTY(assign, type, name)

#define DEFINE_PROPERTY_ASSIGN_Double(name) DEFINE_PROPERTY_ASSIGN(double, name)

#define DEFINE_PROPERTY_STRONG(type, name) DEFINE_PROPERTY(strong, type, name)

#define DEFINE_PROPERTY_STRONG_NSString(name) DEFINE_PROPERTY_STRONG(NSString*, name)
~~~

具体参见DZProgrameDefines 我们完全可以通过使用宏定义，来扩展出一些列的定义属性的宏方法，借助于XCode的强大的自动补全来方便我们输入，少敲了非常多的字符。并且还减少了出错的情况，在可读性上，如果宏定义的名字起得好，可读性也不错。

同时，不得不说的一点是我们借助于这种宏定义的方式，还规范和统一了定义属性的格式，方便维护同一个工程的多个同事修改同一份代码。让他们的代码质量能够保持在一个比较整齐的水平。 这种

代码模板

某些情况下，我们可能会写一些大量的重复代码，而这些代码又很难将其抽离出来做成一个独立的函数（甚至是lambda表达式），而这种时候宏的作用就体现出来了。考虑下述情况：

~~~
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
~~~

上述代码中，我们需要从字典infos中取一批参数并且要判断这些参数是否为空，为空的时候报错并返回。其中有大量的代码是重复的。而这种重复又不太适合抽离成函数那么这个时候就可以这样做了：

~~~
#define GetValueWithLocalNameAndKey(name , key) \

\

NSString* name = [infos getWBValueForKey:key error:&error];\

if (!error) { [self postPayError:error]; return;}\

\

...

GetValueWithLocalNameAndKey(a,aKey);

GetValueWithLocalNameAndKey(b,bKey);

...
~~~

这样做的时候，就将一段代码抽离成了模板。方便了使用和维护。



-
