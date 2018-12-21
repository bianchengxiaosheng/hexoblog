---
title: C++ Thread 同步问题
date: 2018-12-21 17:22:22
tags: [C++,Thraed]
categories: [C++]
---
http://www.cnblogs.com/lidabo/p/7852033.html
https://www.cnblogs.com/jzincnblogs/p/5188051.html
## C++ Thread
在面临并发需求时候，久需要多线程操作，C++11提供了线程操作,在库**#include <thread>中

### 简单例子
``` C
#include <iostream>
#include <thread>
using  namespace std;
const int numLength = 100000;
void ThreadFun(int &value)
{
    for(int i = 0;i< numLength;i++ )
    {
        value++;
    }

}

int main() {

    int tempValue = 0;

    cout << "Thread Count:" << thread::hardware_concurrency() << endl;
    thread th1(ThreadFun,ref(tempValue));
    thread th2(ThreadFun,ref(tempValue));
    th1.join();
    th2.join();
    cout << "Final Result:" << tempValue << endl;
    return 0;
}

```
执行结果
```
Thread Count:8
Final Result:139951
```
发现输出值tempValue不是想象中的200000，这是因为线程冲突导致
### 改善方案
- 互斥量(mutex)
``` C
#include <iostream>
#include <mutex>
#include <thread>
using  namespace std;
const int numLength = 100000;
mutex muxTemp;
void ThreadFun(int &value)
{
    for(int i = 0;i< numLength;i++ )
    {
        muxTemp.lock();
        value++;
        muxTemp.unlock();
    }

}

int main() {

    int tempValue = 0;

    cout << "Thread Count:" << thread::hardware_concurrency() << endl;
    clock_t start = clock();
    thread th1(ThreadFun,ref(tempValue));
    thread th2(ThreadFun,ref(tempValue));
    th1.join();
    th2.join();
    clock_t end = clock();
    cout << "Final Result:" << tempValue <<"use time:" << end - start << " ms"<< endl;

    return 0;
}


```
执行结果:
```
Thread Count:8
Final Result:200000use time:14 ms
```
虽然互斥量简单，但是加解锁需要时间，如果计算量特别大会导致时间消耗
- 原子变量(atomic)
``` C
#include <iostream>
#include <atomic>
#include <thread>
using  namespace std;
const int numLength = 100000;
atomic_int tempValue {0};
void ThreadFun()
{
    for(int i = 0;i< numLength;i++ )
    {

        tempValue++;

    }

}

int main() {



    cout << "Thread Count:" << thread::hardware_concurrency() << endl;
    clock_t start = clock();
    thread th1(ThreadFun);
    thread th2(ThreadFun);
    th1.join();
    th2.join();
    clock_t end = clock();
    cout << "Final Result:" << tempValue <<"use time:" << end - start << " ms"<< endl;

    return 0;
}
```
执行结果:
```
Thread Count:8
Final Result:200000use time:5 ms
```
atomic是线程安全的，速度比互斥量快不少
