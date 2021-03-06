---
layout: post
title: "奇怪的动画"
date: 2015-10-24 19:03:35 +0800
comments: true
categories: iOS
---
在重构红点的UI过程中，发现一个奇怪的问题。使用block传进来的布局信息调用后会神奇的产生一个动画：
<!--more-->


在重构红点的UI过程中，发现一个奇怪的问题。使用block传进来的布局信息调用后会神奇的产生一个动画：

~~~
//扩展红点View
QQExtendViewRedPoint(self.buttton, @"1000100", YES, ^(UIView* originView, UIView* redPointView, RedPointShowInfo* showInfo) {
        redPointView.frame = CGRectMake(CGRectGetMaxX(originView.frame), CGRectGetMinY(originView.frame), CGRectGetWidth(redPointView.frame), CGRectGetHeight(redPointView.frame) );
})
//红点View布局的代码
- (void) layout {
        ....
        case VASHintTypeDot:
                redPointView = [self __layoutRedDot:showInfo]; //(1)第一次默认布局
        ....
            QQRedPointLayoutBlock layoutBlock = self.redPointLayoutBlock;
            if(layoutBlock) {
                        layoutBlock(self, redPointView, showInfo); //(2)用户自定义布局
            }

}
~~~

但是当给~~~self.button~~~加上了一个调整frame的动画之后，开始变得神奇了：

~~~
    [UIView animateWithDuration:1.0 delay:0.8 options:UIViewAnimationOptionCurveEaseOut animations:^{
        if (showOrHide) {
        //动画部分，改变了accountButton的frame
            self.accountButton.frame = CGRectMake(0, 1, self.bottomView.frame.size.width/2 -1 , self.bottomView.frame.size.height -1);
            self.addCardButton.frame = CGRectMake(self.bottomView.frame.size.width/2, 1, self.bottomView.frame.size.width/2, self.bottomView.frame.size.height-1);
            VIPDataReportWithOperationName(@"bindbtnshow", nil, nil, nil, nil);
        }else{
            self.accountButton.frame = CGRectMake(0, 1, self.bottomView.frame.size.width, self.bottomView.frame.size.height -1);
            self.addCardButton.frame = CGRectMake(self.bottomView.frame.size.width+1, 1, self.bottomView.frame.size.width/2, self.bottomView.frame.size.height-1);
        }
    } completion:^(BOOL finished){
        [self updateAddCardButtonWithSubTitle:self.bindCardPromotion.promoWord];//动画完成后update一下
        self.buttonIsAnimating = NO;
    }];
~~~

红点产生了一个奇怪的动画:

![](http://ww4.sinaimg.cn/large/7df22103gw1excfefctlzj20cd044t8p.jpg)

红点首先出现在（1）位置，然年跳动到（2），之后动画返回到(3).
奇怪的地方在于，我们没有给红点添加任何动画，唯一可能添加动画的地方，就是给accountButton添加的动画。而预期的动画效果，不应该出现在用户布局位置和默认布局位置之间抖动的情况。那么问题到底出在了哪里？

而且有意思的是，同样的代码在IOS8上没有问题，在IOS7上才会有问题。所以我们就怀疑到了在动画处理上IOS系统版本之间可能会有差异。

仔细研究代码发现，在layout中对红点进行了两次frame设置，如上面代码中标注的（1）与（2）位置的代码。而IOS7一下的系统在处理父View的layout的时候，同事调用了1与2处的代码，对红点的View进行了两次布局。而又把这两个布局之间的过程处理成了动画，所以产生了抖动的问题。而在IOS8上面apple应该修正了这个问题，把在同一个layout中的布局处理成了一个无动画过程。
