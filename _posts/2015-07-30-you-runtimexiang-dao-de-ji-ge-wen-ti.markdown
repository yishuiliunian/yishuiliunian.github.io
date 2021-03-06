---
layout: post
title: "由Runtime想到的几个问题"
date: 2015-07-30 15:01:25 +0800
comments: true
categories:  iOS
---


这几天有同事开始关注runtime的东西，并做了一个分享。在分享的过程中，提到了一个super的问题。

比如这个例子,因为继承的层级过于复杂，而不知道父类有没有实现WebViewDelegate的特定方法，需要确认一下在执行：

~~~
- (void) webViewDidFinishLoad:(UIWebView *)webView
{
    if ([super respondsToSelector:@selector(webViewDidFinishLoad:)]) {
            [super webViewDidFinishLoad:webView];
     }
     ....
}
~~~

这个语义上看，是完成了我们要的功能，对父类是否实现特定方法进行了验证，但是实际使用的时候却发现，结果一直返回YES，不管父类有没有实现！！！那么问题出在了哪里。
self和super实质上都是向当前的Instance实例发送消息，而其不同在于进行函数查找时的起点不同。self是从当前的class，而super是从superclass。由于~~~respondsToSelector:~~~方法实质上是在NSObject中实现的，也就是说无论用self，还是super来调用该方法实质上，都是调用的NSObject中该方法。同事该方法，检查当前实例是否支持该方法，结果就一直返回YES了。

因为在使用的时候，要完成检查super是否支持某方法就得用其他的方法了：

~~~
  if (class_respondsToSelector(class_getSuperclass([self class]), @selector(webViewDidFinishLoad:))) {
        [super webViewDidFinishLoad:webView];
    }
~~~

虽然知道原因和解决方法，但是还是觉得这个apple的一个bug。因为语义和实际的执行结果出现了偏差。很容易让人在实际编程中，因为这个而造成method not found的crash。
