---
layout: post
title: Block引起的循环引用案例剖析（二）MRC下的__block关键字
date:   2014-03-05 10:00:00
categories: Objective-C
---

在Objective-C中，类的某个成员是block类型，容易引起循环引用，造成内存泄露。本文举例分析在使用手动引用计数（MRC）情况下的代码片段和解决方案。

- - -

# 错误代码

```objc
// MRC or ARC-Disabled (-fno-objc-arc)
typedef void (^blk_t)(void);
@interface MyObject : NSObject {
    blk_t blk_;
}
@end
@implementation MyObject
- (id)init {
    self = [super init];
    blk_ = [^{NSLog(@"self = %@", self);} copy]; 
    return self;
}
- (void)dealloc {
    NSLog(@"dealloc"); 
    [blk_ release];
    [super dealloc];
}
@end

int main() {
    id obj = [[MyObject alloc] init]; 
    NSLog(@"%@", obj);
    [obj release];
    return 0;
}
```


# 错误现象

*-[MyObject dealloc]永远不会被调用到。*


# 错误原理

self是持有堆上的block的引用，堆上block持有self的引用，循环引用，谁都无法释放，表现为dealloc无法被调用。

1. 在-[MyObject init]中，当那个block literal执行copy方法后，该block被(从栈上)复制到堆上，blk\_指向堆上的新复制出来的block，self引用该堆上block。
1. 在block literal被（从栈上）复制到堆上时，堆上block持有self的引用（self的引用计数增加）。
1. 当-[MyObject init]方法执行完毕，栈上的block literal被自动释放。
1. 因为self和堆上block相互引用，谁都无法释放，导致内存泄露。


# 解决方案

将self赋给声明为\_\_block的自动变量，让block使用(捕获)该\_\_block变量，这时，block不会增加self的引用应用计数，打破了循环引用，避免了内存泄露。

```objc
// MRC or ARC-Disabled (-fno-objc-arc)
- (id)init {
    self = [super init];
    __block typeof(self) unsafe_self = self;
    blk_ = ^{NSLog(@"self = %@", unsafe_self);}; 
    return self;
}
```

该方案等价于ARC下面的\_\_unsafe\_\_unretained，因为MRC下没有weak机制，只能如此了。

```objc
// ARC-Enabled (-fobjc-arc)
- (id)init {
    self = [super init];
    __unsafe_unretained typeof(self) unsafe_self = self;
    blk_ = ^{NSLog(@"self = %@", unsafe_self);}; 
    return self;
}
```

**该方案的缺陷是：如果self被释放了，block代码访问到的unsafe_self会是野指针，会导致崩溃。**

使用该方法，要确保block只被self引用，这样当self不存在时，block也就被释放了，没有机会执行，也就没有机会访问所谓的野指针而造成崩溃了。
