## 1.1 介绍

> 定义：
> `In CPython, the global interpreter lock, or GIL, is a mutex that prevents multiple  native threads from executing Python bytecodes at once. This lock is necessary mainly  because CPython’s memory management is not thread-safe. (However, since the GIL  exists, other features have grown to depend on the guarantees that it enforces.)`

```
什么是GIL（cpython解释器才有）
    GIL（全局解释器锁），本质就是一把互斥锁
    即：有了GIL的存在，单个进程内的多个线程同一时刻，只能有一个线程在运行，
        也就是说单个进程进程下的多个线程无法并行，也就是无法利用多核优势，但是并不影响并发
        不过，多个进程(每个进程一个线程)可以使用多核优势
    GIL可以比喻成执行权限，也就是说GIL是加在cpython解释器上的

为何要有GIL
    cpython解
  
结论：GIL也是互斥锁（对共享数据的修改从并发变成串行的），在Cpython解释器中，同一个进程下开启的多线程，同一时刻只能有一个线程执行，无法利用多核优势
```

**例子**

```python
from threading import Thread,Lock
mutex = Lock()
n = 100
def task():
    global n
    mutex.acquire()
    temp = n
    n = temp - 1
    mutex.release()

if __name__ == '__main__':
    t_l = []
    for i in range(100):
        t = Thread(target=task)
        t_l.append(t)
        t.start()
    for t in t_l:
        t.join()
    print(n)
  

```

![1036857-20170829190054374-1627611278](https://cos.liuqm.cc/1036857-20170829190054374-1627611278.png)

## 1.2 GIL与Lock

> - **普通互斥锁**：就是我们用户自己定义的，只用来保护我们自己的数据
> - **GIL**：是cpython定义的，用来抢夺执行权限，当抢到GIL的线程如果做IO了，那么GIL会被释放，CPU会被调走，但是用户的互斥锁不会被释放
> - **线程执行流程**：多个线程先抢到执行权限，抢到了才能执行任务，执行任务的时候可能会对共享数据的修改，这就需要普通互斥锁了

## 1.3 GIL与多线程

> 有了GIL的存在，同一时刻同一进程中只有一个线程被执行
>
> 听到这里，有的同学立马质问：进程可以利用多核，但是开销大，而python的多线程开销小，但却因为GIL无法利用多核优势，也就是说python多线程是个鸡肋？
>
> 别着急啊，老娘还没讲完呢。
>
> 要解决这个问题，我们需要在几个点上达成一致：
>
> - cpu到底是用来做计算的，还是用来做I/O的？
> - 多cpu，意味着可以有多个核并行完成计算，所以多核提升的是计算性能
> - 每个cpu一旦遇到I/O阻塞，仍然需要等待，所以多核对I/O操作没什么用处

### 1.3.1 分析

```python
#分析：
我们有四个任务需要处理，处理方式肯定是要玩出并发的效果，解决方案可以是：
方案一：开启四个进程
方案二：一个进程下，开启四个线程

#单核情况下
　　如果四个任务是计算密集型，没有多核来并行计算，方案一徒增了创建进程的开销，方案二胜
　　如果四个任务是I/O密集型，方案一创建进程的开销大，且进程的切换速度远不如线程，方案二胜

#多核情况下
　　如果四个任务是计算密集型，多核意味着并行计算，在python中一个进程中同一时刻只有一个线程执行用不上多核，方案一胜
　　如果四个任务是I/O密集型，再多的核也解决不了I/O问题，方案二胜

"""
结论：现在的计算机基本上都是多核，
IO密集型中，多进程与多线程都无法利用多核优势，但是多线程效率比多进程高，所以多线程在IO密集型不是鸡肋
计算密集型中，多进程可以利用多核优势，但是多线程就不行了，多线程甚至都没有有串行效率高(因为多线程要来回切换浪费时间)
"""

```

### 1.3.2 实验

**计算密集型任务**

> 计算密集型：多进程效率高

```python
from multiprocessing import Process
from threading import Thread
import os,time
def work():
    res=0
    for i in range(10000000):
        res*=i


if __name__ == '__main__':
    temp=[]
    cpu_count = os.cpu_count() #本机为8核
    start=time.time()
    for i in range(cpu_count):
        #p_or_l=Process(target=work)  # 1.0799839496612549
        p_or_l=Thread(target=work) # 3.3151869773864746
        temp.append(p_or_l)
        p_or_l.start()
    for p_or_l in temp:
        p_or_l.join()
    stop=time.time()
    print('run time is %s' %(stop-start))


```

**IO密集型任务**

> I/O密集型：多线程效率高

```python
from multiprocessing import Process
from threading import Thread
import threading
import os,time
def work():
    time.sleep(1)

if __name__ == '__main__':
    temp=[]
    cpu_count = os.cpu_count() #本机为8核
    start=time.time()
    for i in range(cpu_count):
        p_or_l=Process(target=work) #耗时1.030095100402832,大部分时间耗费在创建进程上
        #p_or_l=Thread(target=work) #耗时1.0068669319152832
        temp.append(p_or_l)
        p_or_l.start()
    for p_or_l in temp:
        p_or_l.join()
    stop=time.time()
    print('run time is %s' %(stop-start))