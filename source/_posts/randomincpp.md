---
title: C++中的随机数问题
author: ado
date: 2019-08-26 10:57:09
description: 最近遇到了随机数问题
tags: [cpp]
---

# Preface

最近遇到了随机数问题，在同一进程同一时间对同样的序列使用 `srand(time(0))`  和 `rand_shuffle()` 进行乱序的时候 ,会出现一样的结果；后面又发现在分布式系统中，同一时间对同样的序列进行乱序也会出现一样的结果。

# Test Case

以下对 **rand/rand_shuffle** 和 **mt19973/shuffle** 进行测试：

```c++

void testRandom()
{

    for(int i =0; i < 10; i++)
        testRandomShuffle();
    std::cout << "end of a" << endl;

    for(int i =0; i < 10; i++)
        testrRandomShuffle();
}

void testRandomShuffle()
{
    vector<int> a = {1,2,3,4,5,6,7,8,9,10};
    srand(time(0));
    random_shuffle(a.begin(),a.end());
    std::copy(a.begin(),a.end(), std::ostream_iterator<int>(std::cout," "));
    std::cout << endl;
}

void testrRandomShuffle()
{
    vector<int> b = {1,2,3,4,5,6,7,8,9,10};
    std::random_device rd;
    std::mt19937 g(rd());
    std::shuffle(b.begin(), b.end(), g);
    std::copy(b.begin(),b.end(), std::ostream_iterator<int>(std::cout," "));
    std::cout << endl;
}
```

结果如下:

![随机乱序测试](./randtest.png)

# Reference

* rand 使用的是 [LCG算法](<https://en.wikipedia.org/wiki/Linear_congruential_generator>)
* mt19937 使用的是 [Mersenne Twister](<https://en.wikipedia.org/wiki/Mersenne_Twister>)
* reddit上关于rand的讨论帖 [is_random_really_better_than_stdrand/](<https://www.reddit.com/r/cpp/comments/9sb3rj/is_random_really_better_than_stdrand/>)

# Terms

* [PRNG](<https://en.wikipedia.org/wiki/Pseudorandom_number_generator>) : Pseudorandom number generator 伪随机数生成器
* LCG : linear congruential generator 线性同余随机数生成器 
*  [纯线性同余随机数生成器](https://www.cnblogs.com/xkfz007/archive/2012/03/27/2420154.html)
* [DRBG](<https://en.wikipedia.org/wiki/Pseudorandom_number_generator>) :  deterministic random bit generator 确定性（伪随机）数产生器， 也就是PRNG
* [伪随机数发生器（DRBG）](<https://blog.csdn.net/wenxiaohua_113/article/details/18145305>)
* [HRNG](<https://en.wikipedia.org/wiki/Hardware_random_number_generator>) : Hardware random number generator, TRNG
* NRBG : Non-deterministic Random Bit Generators ,非确定性（真随机）数产生器