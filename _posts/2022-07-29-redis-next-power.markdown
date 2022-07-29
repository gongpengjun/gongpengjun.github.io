---
layout: post
title: Redis算法 - 向上取整到 2 的次幂
date:   2022-07-29 09:00:00
categories: Storage Redis
---

Redis在字典扩容时会选择向上取整到 2 的次幂。具体代码如下：

```language:c
/* This is the initial size of every hash table */
#define DICT_HT_INITIAL_SIZE     4
/* Our hash table capability is a power of two */
static unsigned long _dictNextPower(unsigned long size) {
    unsigned long i = DICT_HT_INITIAL_SIZE;

    if (size >= LONG_MAX) return LONG_MAX;
    while(1) {
        if (i >= size)
            return i;
        i *= 2;
    }
}
```

源码链接：https://github.com/redis/redis/blob/unstable/deps/hiredis/dict.c#L310

更多解法：
https://www.techiedelight.com/zh/round-next-highest-power-2/