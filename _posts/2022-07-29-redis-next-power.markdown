---
layout: post
title: Redis算法 - 向上取整到 2 的次幂
date:   2022-07-29 09:00:00
categories: Storage Redis
---

### 1、向上取整到2的次幂

Redis在字典扩容时会选择向上取整到2的次幂：

```c
/* This is the initial size of every hash table */
#define DICT_HT_INITIAL_SIZE     4

/* Our hash table capability is a power of two */
unsigned long _dictNextPower(unsigned long size) {
    unsigned long i = DICT_HT_INITIAL_SIZE;

    if (size >= LONG_MAX) return LONG_MAX;
    while(1) {
        if (i >= size)
            return i;
        i *= 2;
    }
}
```

### 2、判断是不是2的次幂

思路：2的次幂的二进制只有最高位是1，其余为0，减1后最高位向后借位为0，其余为1

```c
// 返回值 1表示n是2的次幂，0表示不是
int isPowerOfTwo(int n) {
    //注意按位与的优先级小于等于号 需要加括号
    if(n>0 && (n&n-1)==0) {  
        return 1;
    }
    return 0;
}
```



### 3、参考信息

参考信息：[源码链接](https://github.com/redis/redis/blob/unstable/deps/hiredis/dict.c#L310)、[更多解法](https://www.techiedelight.com/zh/round-next-highest-power-2/)、[判断整数次幂问题](https://zhuanlan.zhihu.com/p/419952714)