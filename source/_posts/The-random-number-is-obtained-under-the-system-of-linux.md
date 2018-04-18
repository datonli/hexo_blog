---
title: linux系统下获取随机数
date: 2018-04-18 22:01:16
tags: linux
---
linux系统下一般调用rand/rand_r函数来获取随机数，但是这个随机数随不随机其实跟seed有关，如果seed是一样的，那么多轮的随机数其实是完全一样的。这时可以依赖系统时间来获取真正的随机数，实现如下：

```
#include <sys/time.h>
#include <stdlib.h>
#include <stdio.h>

int
main(int argc, char *argv[])
{
    int j, r, nloops = 20;
    unsigned int seed;

    struct timeval tv;
    struct timezone tz;
    gettimeofday(&tv, &tz);
    seed = tv.tv_usec;
    srand(seed);
    for (j = 0; j < nloops; j++) {
        r =  rand();
        printf("%d\n", r);
    }

    exit(EXIT_SUCCESS);
}
```