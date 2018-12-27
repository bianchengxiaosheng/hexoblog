---
title: C++ Thread 同步问题
date: 2018-12-21 17:22:22
tags: [C++,Thraed]
categories: [C++]
---

## C++ Thread同步
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

- 加锁和解锁的另一种改善  

互斥类的最重要成员函数是lock()和unlock()。在进入临界区时，执行lock()加锁操作，如果这时已经被其它线程锁住，则当前线程在此排队等待。退出临界区时，执行unlock()解锁操作。更好的办法是采用”资源分配时初始化”(RAII)方法来加锁、解锁，这避免了在临界区中因为抛出异常或return等操作导致没有解锁就退出的问题。极大地简化了程序员编写mutex相关的异常处理代码。C++11的标准库中提供了std::lock_guard类模板做mutex的RAII。

std::lock_guard类的构造函数禁用拷贝构造，且禁用移动构造。std::lock_guard类除了构造函数和析构函数外没有其它成员函数。

在std::lock_guard对象构造时，传入的mutex对象(即它所管理的mutex对象)会被当前线程锁住。在lock_guard对象被析构时，它所管理的mutex对象会自动解锁，不需要程序员手动调用lock和unlock对mutex进行上锁和解锁操作。lock_guard对象并不负责管理mutex对象的生命周期，lock_guard对象只是简化了mutex对象的上锁和解锁操作，方便线程对互斥量上锁，即在某个lock_guard对象的生命周期内，它所管理的锁对象会一直保持上锁状态；而lock_guard的生命周期结束之后，它所管理的锁对象会被解锁。程序员可以非常方便地使用lock_guard，而不用担心异常安全问题。

std::lock_guard在构造时只被锁定一次，并且在销毁时解锁。
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
        lock_guard<mutex> mtx_locker(muxTemp);
        //muxTemp.lock();
        value++;
        //muxTemp.unlock();
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
执行结果  
```
Thread Count:8
Final Result:200000use time:12 ms
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
