---
layout: post
title: Block引起的循环引用案例剖析（一）成员变量为Block类型[ARC]
date:   2014-03-04 10:00:00
categories: Objective-C
---

在iOS/Mac开发中，类的某个成员是block类型，容易引起循环引用，造成内存泄露。本文举例分析在使用ARC（自动引用计数）的情况下的代码片段和解决方案。

- - -
<p></p>

# 错误代码

{% highlight objc %}
// ARC-Enabled (-fobjc-arc)
typedef void (^blk_t)(void);
@interface MyObject : NSObject {
    blk_t blk_;
}
@end
@implementation MyObject
- (id)init {
    self = [super init];
    blk_ = ^{NSLog(@"self = %@", self);}; 
    return self;
}
- (void)dealloc {
    NSLog(@"dealloc"); 
}
@end

int main() {
    id obj = [[MyObject alloc] init]; 
    NSLog(@"%@", obj);
    return 0;
}
{% endhighlight %}

<p></p>

# 错误现象

<code>-[MyObject dealloc]永远不会被调用到。</code>

<p></p>

# 错误原理

self强引用block，block强引用self，循环引用，谁都无法释放，表现为dealloc无法被调用。

1. 在ARC环境下，成员变量blk_t blk_;等价于\_\_strong blk_t blk_;
1. 在-init中，当那个block literal赋给blk_时，该block被从栈上复制到堆上，self持有block的引用。
1. 在ARC环境下，block代码内部直接引用self，也是\_\_strong引用。
1. 在那个block literal被从栈上复制到堆上时，self的引用计数增加，block持有self的引用。
1. 因为self和block相互强引用，谁都无法释放，导致内存泄露。

<p></p>

# 解决方案一：简单方案

* 将self赋给声明为\_\_weak的自动变量，让block使用(捕获)该\_\_weak变量，这样block不再持有self的引用，打破了循环引用，避免了内存泄露。
* 因为block捕获的wself是一个\_\_weak类型，当self被释放时，wself会自动被置为nil，这样block代码访问wself时，不会引用到野指针。

{% highlight objc %}
- (id)init {
    self = [super init];
    __weak typeof(self) wself = self;
    blk_ = ^{NSLog(@"self = %@", wself);}; 
    return self;
}
{% endhighlight %}

<p></p>

# 解决方案二：完善方案

上面的解决方案一存在两个问题：

* 在多线程环境下，self在一个线程被释放，而blk_在另一个线程执行，有可能在wself要被置为但还未被置为nil的瞬间，blk_代码访问了wself的方法或成员变量而引起崩溃。
* 如果block代码希望在执行期间，self总是有效的对象，而不是nil，上面的方案也无法保证。

因此，可以在block代码开始的时候，将wself赋值给一个\_\_strong引用的临时变量来解决上面的两个问题。

{% highlight objc %}
- (id)init {
    self = [super init];
    __weak typeof(self) wself = self;
    blk_ = ^{
        __strong typeof(self) sself = wself;
        if(!sself)
            return;
        NSLog(@"self = %@", sself);
    }; 
    return self;
}
{% endhighlight %}

block内部的sself是临时变量，只会在block被执行的短暂时间段持有对self的引用，在block代码执行完成之后（出了sself的作用域），该sself会自动被释放。
这样虽然在block被执行的短暂时间段存在循环引用，但是该循环引用是动态的、短暂的、必定被打破的，所以是安全的，不会引起内存泄露。

* 在block被执行之前，临时变量sself不存在，不存在循环引用。
* 在block被执行当中，临时变量block被建立，block通过sself短暂持有对self的强引用，self通过blk_持有对block的强引用，存在循环引用。
* 在block被执行之后，临时变量sself被释放，不存在循环引用。

<p></p>

# 真实案例

最后，举一个使用方案二的实例，AFNetworking的实现代码：
[UIImageView+AFNetworking](https://github.com/AFNetworking/AFNetworking/blob/master/UIKit%2BAFNetworking/UIImageView%2BAFNetworking.m#L142-L146)
