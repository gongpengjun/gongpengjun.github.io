---
layout: post
title: ObjC Block循环引用案例剖析（三）
date:   2014-03-12 10:00:00
categories: iOS Objective-C
---

# ARC下的__block关键字

为了避免Block相关的循环引用，MRC下使用\_\_block关键字，ARC下使用\_\_Weak关键字，它们有什么区别？既然ARC下的\_\_weak关键字更好，那为什么在ARC下看到有时使用\_\_block呢？是为了避免循环引用吗？不是的话，那ARC下的\_\_block关键字又是干什么用的呢？下面我们就来一一回答这几个问题，并举一个真实的案例说明ARC下的\_\_weak和\_\_block两个关键字各自的妙用。

- - -

### 问题一：为避免循环引用，MRC下使用\_\_block关键字，ARC下使用\_\_Weak关键字，它们效果一样吗？有什么区别？


参见本系列的前两篇文章

* <a href="/ios/objective-c/block-circular-reference-in-arc.html" target="_blank">Block引起的循环引用案例剖析（一）ARC下的__weak关键字</a>
* <a href="/ios/objective-c/block-circular-reference-in-mrc.html" target="_blank">Block引起的循环引用案例剖析（二）MRC下的__block关键字</a>

我们知道

* 在ARC环境下，使用\_\_weak关键字，可以避免循环引用
* 在MRC环境下，使用\_\_block关键字，可以避免循环引用

在避免循环引用这一点上，两者效果相同，都可以避免block引用self，从而打破block和self之间的循环引用。但当被引用对象self释放时，两者的行为就不一样了：在MRC下，指定了\_\_block关键字的变量指向的地址不变，这时，如果访问该变量就会产生访问错误令程序崩溃；而在ARC下的\_\_weak关键字变量会自动被置为nil，这时，如果访问该变量，nil，像Objective-C一贯一样，不会有任何副作用。所以ARC下的\_\_weak可以让程序更安全更健壮。

### 问题二：既然ARC下的\_\_weak关键字更好，那为什么在ARC下看到有时使用\_\_block呢？是为了避免循环引用吗？那ARC下的\_\_block关键字又是干什么用的呢？

\_\_block关键字的本意是表示通过引用捕获变量，通过引用(即变量的地址)来访问变量，这样就可以给该变量赋值(writable)。即在block里面可以给指定了\_\_block关键字的外部变量赋值，它是为了让block将一些信息传递到block之外。比如下面的例子：

```objc
__block int x = 1; //  x lives in block storage
void (^printXAndY)(int) = ^(int y) {
    x = x + y;
    printf("%d %d\n", x, y);
};
printXAndY(2); // prints: 3 2
// x is now 3
```

在该例子中，x被指定了\_\_block关键字，这样printXAndY这个block里面的代码就可以将新的值赋给block外部的变量x：``` x = x + y; ```。

同样，对于id类型的变量，也可以指定\_\_block关键字，这样，也可以在block里面给这样的变量赋值，含义是让它指向新的对象（retain或者non-retain）。

所以，在ARC下，\_\_block关键字不是为了解决循环引用的，而是为了解决让变量可以在block内部被赋值。比如我们在SDWebImage的库代码里可以看到如下的写法：
<a href="https://github.com/rs/SDWebImage/blob/42f97369726f1ee282b40b63616e339adfcb2c8a/SDWebImage/SDWebImageDownloader.m#L108-L164" target="_blank">SDWebImageDownloader.m (line:108)</a>

```objc
- (id<SDWebImageOperation>)downloadImageWithURL:...
{
    __block SDWebImageDownloaderOperation *operation;
    __weak SDWebImageDownloader *wself = self;
    
    [self addProgressCallback: ... createCallback:^
     {
         // ...
         operation = [SDWebImageDownloaderOperation.alloc
                      // ...
                      ];
     }];
    
    return operation;
}
```

**上面代码的含义是这样的：**

1. 代码中变量operation被指定了\_\_block关键字:

    ```objc
    __block SDWebImageDownloaderOperation *operation; 
    ```
1. 这是为了在block('createCallback')里可以对外部变量operation赋值:

    ```objc
    operation = [SDWebImageDownloaderOperation.alloc
    ```

1. 从而让block('createCallback')外部的代码可以引用在block里面生成的对象:

    ```objc
    return operation;
    ```

如果不使用\_\_block关键字，那么block('createCallback')里就无法给外部变量operation赋值，运行时，``` return operation; ```就永远返回nil。其实，不用等到运行时，编译器在编译时就会发现这个问题，并报告错误。

- - -

### 结论

1. 在ARC下，\_\_weak关键字是为了解决循环引用。
1. 在ARC下，\_\_block关键字是为了让block里可以对外部的变量赋值(writable)，从而将block里面计算的值或生成的对象传递出去。

