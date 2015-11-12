---
layout: post
title: "OC中使用defer操作"
date: 2015-11-11T21:32:15+08:00
categories: iOS
---

类似于 golang的defer  将一个操作延迟到作用域结束的时候 执行：常见于 关闭文件等。。。。这是异常处理的一种替代方案。

~~~
#define DEFER(block) __unused OCDefer* defer___ = [[OCDefer alloc] initWithBlock:block];

typedef void(^OCDeferBlock)(void);

@interface OCDefer : NSObject

- (instancetype) initWithBlock:(OCDeferBlock)block;

@end

@interface OCDefer ()

{

    OCDeferBlock _block;

}

@end

@implementation OCDefer

- (instancetype) initWithBlock:(OCDeferBlock)block

{

    self = [super init];

    if (!self) {

        return self;

    }


    _block = block;

    return self;

}

- (void) dealloc

{

    if (_block) {

        _block();

    }

    _block = nil;

}

@end

//测试的例子

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions

{


    FILE* file = fopen("/afile.txt", "r");

    DEFER(^{

        NSLog(@"hello");

        fclose(file);

    });


    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];

    // Override point for customization after application launch.

    self.window.backgroundColor = [UIColor whiteColor];

    [self.window makeKeyAndVisible];

    return YES;

}

~~~


-----
欢迎关注iOS开发公共账号iOS_Tips：扫描下方二维码关注  
![](http://ww4.sinaimg.cn/large/7df22103jw1exx11uhhkoj20by0by3zc.jpg)
