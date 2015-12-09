---
layout: post
title: "Objective-C的内存布局"
date: 2015-11-30T22:09:21+08:00
categories: iOS
---

#Objective-C的内存布局


##对象的内存

~~~
//引自 objc-private.h

struct objc_object {
private:
    isa_t isa;

public:

    Class ISA();
    Class getIsa();
    void initIsa(Class cls /*indexed=false*/);
    void initClassIsa(Class cls /*indexed=maybe*/);
    void initProtocolIsa(Class cls /*indexed=maybe*/);
    void initInstanceIsa(Class cls, bool hasCxxDtor);
    Class changeIsa(Class newCls);
    bool hasIndexedIsa();
    bool isTaggedPointer();
    bool isClass();
    bool hasAssociatedObjects();
    void setHasAssociatedObjects();
    bool isWeaklyReferenced();
    void setWeaklyReferenced_nolock();
    bool hasCxxDtor();
    id retain();
    void release();
    id autorelease();
    // Implementations of retain/release methods
    id rootRetain();
    bool rootRelease();
    id rootAutorelease();
    bool rootTryRetain();
    bool rootReleaseShouldDealloc();
    uintptr_t rootRetainCount();
    // Implementation of dealloc methods
    bool rootIsDeallocating();
    void clearDeallocating();
    void rootDealloc();

private:
    void initIsa(Class newCls, bool indexed, bool hasCxxDtor);

    // Slow paths for inline control
    id rootAutorelease2();
    bool overrelease_error();

#if SUPPORT_NONPOINTER_ISA
    // Unified retain count manipulation for nonpointer isa
    id rootRetain(bool tryRetain, bool handleOverflow);
    bool rootRelease(bool performDealloc, bool handleUnderflow);
    id rootRetain_overflow(bool tryRetain);
    bool rootRelease_underflow(bool performDealloc);

    void clearDeallocating_weak();

    // Side table retain count overflow for nonpointer isa
    void sidetable_lock();
    void sidetable_unlock();

    void sidetable_moveExtraRC_nolock(size_t extra_rc, bool isDeallocating, bool weaklyReferenced);
    bool sidetable_addExtraRC_nolock(size_t delta_rc);
    bool sidetable_subExtraRC_nolock(size_t delta_rc);
    size_t sidetable_getExtraRC_nolock();
#endif

    // Side-table-only retain count
    bool sidetable_isDeallocating();
    void sidetable_clearDeallocating();

    bool sidetable_isWeaklyReferenced();
    void sidetable_setWeaklyReferenced_nolock();

    id sidetable_retain();
    id sidetable_retain_slow(SideTable *table);

    bool sidetable_release(bool performDealloc = true);
    bool sidetable_release_slow(SideTable *table, bool performDealloc = true);

    bool sidetable_tryRetain();

    uintptr_t sidetable_retainCount();
#if !NDEBUG
    bool sidetable_present();
#endif
};
~~~

成为objc_object的只有一个isa指针。

###weak-object对象内存布局

###-object辅助对象内存布局





###类加载

新创建默认的一个demo，将会加载4060个类。所有的类占用内存为1369120，此事程序总共占用内存4.9MB。其中27%的内存用来存储类型信息。





