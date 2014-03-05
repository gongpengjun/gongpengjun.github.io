---
layout: default
title: Block相关常见错误
date:   2014-03-03 10:00:00
categories: Objective-C
---

# Block相关常见错误

本文列举错误代码、错误现象、错误原理、解决方案，以供iPhone/iPad/Mac开发者参考借鉴。

## 循环引用案例（一）

<code></code>
- - -
<code></code>

### 错误代码

<code>
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
</code>

### 错误现象

<code>-[MyObject dealloc]永远不会被调用到。</code>

### 错误原理

self强引用block，block强引用self，循环引用，谁都无法释放，表现为dealloc无法被调用。

<code>

1. 在ARC环境下，成员变量blk_t blk_;等价于\_\_strong blk_t blk_;
1. 在-init中，当那个block literal赋给blk_时，该block被从栈上复制到堆上，self持有block的引用。
1. 在ARC环境下，block代码内部直接引用self，也是\_\_strong引用。
1. 在那个block literal被从栈上复制到堆上时，self的引用计数增加，block持有self的引用。
1. 因为self和block相互强引用，谁都无法释放，导致内存泄露。

</code>

### 解决方案

将self赋给声明为\_\_weak的自动变量，让block使用该\_\_weak变量，这样block不再持有self的应用，打破了循环应用。

<code>
{% highlight objc %}
- (id)init {
    self = [super init];
    __weak typeof(self) wself = self;
    blk_ = ^{NSLog(@"self = %@", wself);}; 
    return self;
}
{% endhighlight %}
</code>

- - -
<code></code>

## 循环引用案例（二）

<code></code>
- - -
<code></code>

