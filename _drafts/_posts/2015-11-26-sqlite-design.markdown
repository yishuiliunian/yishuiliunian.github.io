---
layout: post
title: "Sqlite-design"
date: 2015-11-26T16:29:15+08:00
categories: iOS
---



在IOS程序设计中，一般都需要管理本地化数据。apple为我们提供多种方式来本地化数据比如：core data，一般的平面文件，当然还有sqlite。core data在苹果的官方文档中说是一个高级功能，不建议新手程序员使用。我粗略的研究了一下core data，毕竟是苹果原生的东西。在很多地方，与苹果原生的系统结合的非常好。比如可以直接将core data作为UITableView的数据源来使用，无论是从编程效率还是程序的优雅型上提高了很多。但是，正如苹果说的那样这毕竟是一项高级功能，在使用起来还是有点费劲的，尤其是在多线程环境下。因为使用管了sqlite，所以在本地化的时候就多看了一下sqlite的东西。
      apple只提供了sqlite的库文件，如果想使用sqlite，还是得自己再去封装一层sqlite。开始，我用C++封装了简单的封装了一层数据库，方便上层调用。但是在使用的过程中越到了多线程的问题。sqlite在多线程环境下有三种运行模式：参见（http://blog.csdn.net/diyagoanyhacker/article/details/7209888）

单线程：禁用所有的mutex锁，并发使用时会出错。当SQLite编译时加了SQLITE_THREADSAFE=0参数，或者在初始化SQLite前调用sqlite3_config(SQLITE_CONFIG_SINGLETHREAD)时启用。 
多线程：只要一个数据库连接不被多个线程同时使用就是安全的。源码中是启用bCoreMutex，禁用bFullMutex。实际上就是禁用数据库连接和prepared statement（准备好的语句）上的锁，因此不能在多个线程中并发使用同一个数据库连接或prepared statement。当SQLite编译时加了SQLITE_THREADSAFE=2参数时默认启用。若SQLITE_THREADSAFE不为0，可以在初始化SQLite前，调用sqlite3_config(SQLITE_CONFIG_MULTITHREAD)启用；或者在创建数据库连接时，设置SQLITE_OPEN_NOMUTEX flag。 
串行：启用所有的锁，包括bCoreMutex和bFullMutex。因为数据库连接和prepared statement都已加锁，所以多线程使用这些对象时没法并发，也就变成串行了。当SQLite编译时加了SQLITE_THREADSAFE=1参数时默认启用。若SQLITE_THREADSAFE不为0，可以在初始化SQLite前，调用sqlite3_config(SQLITE_CONFIG_SERIALIZED)启用；或者在创建数据库连接时，设置SQLITE_OPEN_FULLMUTEX flag。、

     苹果提供的库在测试后发现，他们在编译的时候使用的是第二种方式。第二种方式，解决多个线程读写同一个数据库的问题。但是同一个数据库连接不能够多个线程间共享。一个解决方案是为每一个线程都生成一个数据库连接。于是得频繁的打开和关闭数据库连接。虽然打开和关闭sqlite消耗的时间并不是很大，但是每次打开都会系统会为这个数据库连接开辟一段内存作为缓存，但是关闭库连接的时候并没有回收（回收机制不太明确）。而苹果建议手机程序占用内存不能超过20MB，这样在多次打开关闭后，因为数据库缓存的问题，可能会引起程序的内存占用量激增超过20MB。于是用没有一种折中的方案呢。我在网上找了很多ios上sqlite的开源库，最终发现了很多程序都在用的FMDB（https://github.com/ccgus/fmdb）。这个库对于多线程使用blocks的方式进行处理，可以在多个线程之间共享数据库连接。使用这个库就可以在整个程序运行期间只维护几个数据库连接。
     整个程序在多线程环境下的数据库架构就清晰了起来。
     数据库连接类WizDataBase，保持一个FMDB的数据库连接，并提供针对程序的数据库查询方法。这个类实现WizDataBaseDeleage协议。WizDataBaseDeleage协议定义了程序中需要进行数据交互的所有数据库操作。然后
[pre]/*WizDataBaseDelegate.h*/@protocolWizDataBaseDelegate//具体的接口@end

/*
WizDataBase.h
*/
@interface WizDataBase<WizDataBaseDelegate>
{
　　FMDB* fmdb;
  ......
}
@end[/pre]


     数据库管理类（单例）WizDataBaseManager负责维护数据连接。实际上提供的是满足WizDataBaseDeleage协议的对象。而不是一个具体的WizDataBase对象。这样就可以通过类似与接口的方式，将数据库层的具体实现屏蔽，只关心所需要的具体借口。
　　[pre]#import "WizDataBase.h"#import "WizDataBaseDelegate.h"@interfaceWizDbManager(){    NSMutableDictionary*dbDataDictionary;}@property (atomic, retain) NSMutableDictionary*dbDataDictionary;- (void) clearDataBase;- (id<WizDataBaseDelegate>) getWizDataBase;@end@implementationWizDbManager@synthesizedbDataDictionary;- (void) dealloc{    [dbDataDictionary release];    [super dealloc];}...........@end[/pre]

      在具体使用的使用向WizDataBaseManager请求一个WizDataBase，然后使用。在这特意提一下一种常见的数据库应用场景，就是在使用MVC的使用模型控制器从数据库中读取数据然后去更新UI。读取数据库当然是一个很费事的操作，如果让主线程一直忙于读取数据库的话，将会降低UI的响应，让界面看起来很卡。于是就很有必要将读取数据的操作使用多线程的方式处理。apple提供很多多线程处理方式，NSOperation、GCD、Thread。在这种场景下，最适合的是G CD。写个例子：
 dispatch_async(dispatch_get_global_queue(0, 0), ^{        if (nil ==documentGuid) {            return;        }        id<WizDataBaseDelegate> dataBase =[[WizDataBaseManager shareDbManager] getWizTempDataBase];        WizAbstract* abstract =[dataBase abstractOfDocument:documentGuid];        dispatch_async(dispatch_get_main_queue(), ^{            if (nil == abstract) {                return;            }            [self.data setObject:abstractforKey:documentGuid];           imageView.image = abstract.image;        });    });[/pre]


-----
欢迎关注iOS开发公共账号iOS_Tips：扫描下方二维码关注  
![](http://ww4.sinaimg.cn/large/7df22103jw1exx11uhhkoj20by0by3zc.jpg)
