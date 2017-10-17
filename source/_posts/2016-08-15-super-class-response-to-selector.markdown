---
layout: post
title: "Objective-C中正确的判断父类是否实现了某个方法"
date: 2016-08-15T18:13:01+08:00
categories: iOS
---


Objective-C中正确的判断父类是否实现了某个方法
==

> 相关代码可在[Github](https://github.com/yishuiliunian/SuperResponseTest)得到


在某些特殊的场景下，我们会有判断父类是否实现了某个方法的需求。比如在tableViewDelegate中的didSelectCellAtIndexPath方法中：为了不覆盖父类的对应方法，在实现的时候需要实现判断一下父类是否实现了该方法，实现了则调用一下父类的方法，没有实现则不调用如下：


~~~

- (void) tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    Class supClass = class_getSuperclass([self class]);
    NSLog(@"self class is %@, super class is %@",self.class, supClass);
    if (class_respondsToSelector(supClass, _cmd)) {
        struct objc_super sup = {
            .receiver = self,
            .super_class = [self class]
        };
        void(*func)(struct objc_super*,SEL,UITableView*,NSIndexPath*) = (void*)&objc_msgSendSuper;
        func(&sup, _cmd,tableView, indexPath);
    }
}

~~~

But，这样做会造成死循环。为什么呢，且看下面的这个例子：


~~~
@interface A : AnSuper

@end
@implementation A

- (void) test
{
    Class supClass = class_getSuperclass([self class]);
    NSLog(@"self class is %@, super class is %@",self.class, supClass);
    if (class_respondsToSelector(supClass, _cmd)) {
        struct objc_super sup = {
            .receiver = self,
            .super_class = [self class]
        };
        void(*func)(struct objc_super*,SEL) = (void*)&objc_msgSendSuper;
        func(&sup, _cmd);
    }
}
@end

@interface B  : A
@end

@implementation B
@end

@interface C : B
@end

@implementation C
@end

void TestC() {
    C* c = [C new];
    [c test];
}

~~~

C类继承自B类，B类继承自A类，在A中有方法test，在test方法中判断了其父类是否实现过test方法，实现则行执行。当创建一个C的实例调用test方法的时候。结果就死循环到这个test方法了。

因为当调用[self class]的时候，无论当前方法实现在哪个类中，其都是以当前的instance的isa指针为准的，也就是说当C类的Instance上调用class方法的时候，返回的都是当前的类C，而去查找其父类则会一直是B类。不会像我们想想的一样，是从类A开始查找，找到A类的父类AnSuper。

而在test方法中，判断父类是否实现了test方法，则一直是在判断B类是否实现了test方法，返回YES，一直在调用。死循环了就。

而我们的目标是为了判断当前的方法是否在父类中有实现，很明显因为[self class]无法返回这个方法实现所在的类。导致无法满足的需求，还产生了BUG。本质上使用[self class]这个方法就是不对的。那应该使用什么呢？

在C++中有\_\_CLASS\_\_的环境变量，来指称当前的类。而OC中没有类似的环境变量,可以参见Apple Mailing List中的讨论[\_\_CLASS\_\_macro for current Obj-C class name?](http://lists.apple.com/archives/objc-language/2008/Aug/msg00177.html)。

要是有个\_\_CLASS\_\_能够指称当前的方法实现所在的类就好了，我们的需求就能迎刃而解。然后就去环境变量中挨个查找。终于发现了\_\_FOUNCTION\_\_。这个东西。


~~~
-[A test]
~~~

在A类的test方法中打印一下，能够看到其输出了当前的函数实现所在的位置。也就是说我们可以从中提取出，当前方法实现所在的类。哈哈。说干就干。


~~~
Class DZGetCurrentClassInvocationSEL(NSString* functionString)
{
    
    if (functionString.length == 0) {
        return nil;
    }
    
    NSRange rangeStart = [functionString rangeOfString:@"["];
    NSRange rangeEnd = [functionString rangeOfString:@" "];
    if (rangeStart.location == NSNotFound || rangeEnd.location == NSNotFound) {
        return nil;
    }
    NSInteger start = rangeStart.location + rangeStart.length;
    if (rangeEnd.location - start <= 0) {
        return nil;
    }
    NSRange classRange = NSMakeRange(start, rangeEnd.location - start);
    NSString *classString = [functionString substringWithRange:classRange];
    return NSClassFromString(classString);
}

BOOL DZCheckSuperResponseToSelector(Class cla, SEL selector) {
    Class superClass = class_getSuperclass(cla);
    return class_respondsToSelector(superClass, selector);
}



FOUNDATION_EXTERN Class DZGetCurrentClassInvocationSEL(NSString*  functionString);

FOUNDATION_EXTERN BOOL DZCheckSuperResponseToSelector(Class cla, SEL selector);

#define __IMP_CLASS__  DZGetCurrentClassInvocationSEL([NSString stringWithFormat:@"%s",__FUNCTION__])
#define __DZSuperResponseCMD__ DZCheckSuperResponseToSelector(__IMP_CLASS__, _cmd)

~~~


这里定义了两个宏~~~__IMP_CLASS__~~~用于指称当前方法实现所在的类，~~~__DZSuperResponseCMD__~~~用于检查父类是否实现了该方法。

测试一下，在A类中添加测试函数test2:


~~~
- (void) test2
{
    if (__DZSuperResponseCMD__) {
        struct objc_super sup = {
            .receiver = self,
            .super_class = [self class]
        };
        void(*func)(struct objc_super*,SEL) = (void*)&objc_msgSendSuper;
        func(&sup, _cmd);
    }
}
~~~

顺利通过。

现在可以使用上面构建的两个宏来做判断父类是否实现了特定方法的事情了。
