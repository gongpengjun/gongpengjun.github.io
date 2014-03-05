---
layout: default
title: Block引起的循环引用案例（二）成员变量为Block类型[MRC]
date:   2014-03-05 10:00:00
categories: Objective-C
---

# Block引起的循环引用案例（二）成员变量为Block类型[MRC]

在iOS/Mac开发中，类的某个成员是block类型，容易引起循环引用，造成内存泄露。本文举例分析在使用MRC（手动引用计数）的情况下的代码片段和解决方案。

<code></code>
- - -
<code></code>

## 错误代码

<code>
{% highlight objc %}
// ARC-Disabled (-fno-objc-arc) ==> MRC
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
{% endhighlight %}
</code>

## 错误现象

<code>-[MyObject dealloc]永远不会被调用到。</code>

## 错误原理

self是持有堆上的block的引用，堆上block持有self的引用，循环引用，谁都无法释放，表现为dealloc无法被调用。

<code>

1. 在-init中，当那个block literal赋给blk_时，该block被从栈上复制到堆上，blk_指向堆上的新复制出来的block，self是该堆上block的所有者（owner）。
1. 在那个block literal被从栈上复制到堆上时，self的引用计数增加，堆上block持有self的引用。
1. 当-init方法执行完毕，栈上的block literal被自动释放。
1. 因为self和block相互强引用，谁都无法释放，导致内存泄露。

</code>

## 解决方案

将self赋给声明为\_\_block的自动变量，让block使用(捕获)该\_\_block变量，这时，block不会增加self的引用应用计数，打破了循环引用，避免了内存泄露。

*该方案的缺陷是：如果self被释放了，block代码访问到的wself会是野指针，会崩溃。*

使用该方法，要确保block只被self引用，这样当self不存在时，block也就被释放了，没有机会执行，也就没有机会访问所谓的野指针而造成崩溃了。

<code>
{% highlight objc %}
- (id)init {
    self = [super init];
    __block typeof(self) wself = self;
    blk_ = ^{NSLog(@"self = %@", wself);}; 
    return self;
}
{% endhighlight %}
</code>
