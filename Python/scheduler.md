python## 1.1 schedule

```python
import schedule
import time
 
def job():
    print("I'm working...")
 
schedule.every(10).minutes.do(job)
schedule.every().hour.do(job)
schedule.every().day.at("10:30").do(job)
schedule.every(5).to(10).days.do(job)
schedule.every().monday.do(job)
schedule.every().wednesday.at("13:15").do(job)
 
while True:
    schedule.run_pending()
    time.sleep(1)
```
这是在pypi上面给出的示例。这个栗子简单到我不需要怎么解释。而且，通过这个栗子，我们也可以知道，schedule其实就只是个定时器。在while True死循环中，schedule.run_pending()是保持schedule一直运行，去查询上面那一堆的任务，在任务中，就可以设置不同的时间去运行。跟crontab是类似的。

但是，如果是多个任务运行的话，实际上它们是按照顺序从上往下挨个执行的。如果上面的任务比较复杂，会影响到下面任务的运行时间。比如我们这样：

```python
import datetime
import schedule
import time
 
def job1():
    print("I'm working for job1")
    time.sleep(2)
    print("job1:", datetime.datetime.now())
 
def job2():
    print("I'm working for job2")
    time.sleep(2)
    print("job2:", datetime.datetime.now())
 
def run():
    schedule.every(10).seconds.do(job1)
    schedule.every(10).seconds.do(job2)
 
    while True:
        schedule.run_pending()
        time.sleep(1)

```
接下来你就会发现，两个定时任务并不是10秒运行一次，而是12秒。是的。由于job1和job2本身的执行时间，导致任务延迟了。

其实解决方法也很简单：用多线程/多进程。不要幼稚地问我“python中的多线程不是没有用吗？”这是两码事。开了一条线程，就把job独立出去运行了，不会占主进程的cpu时间，schedule并没有花掉执行一个任务的时间，它的开销只是开启一条线程的时间，所以，下一次`执行就变成了10秒后而不是12秒后。

```python
import datetime
import schedule
import threading
import time
 
def job1():
    print("I'm working for job1")
    time.sleep(2)
    print("job1:", datetime.datetime.now())
 
def job2():
    print("I'm working for job2")
    time.sleep(2)
    print("job2:", datetime.datetime.now())
 
def job1_task():
    threading.Thread(target=job1).start()
 
def job2_task():
    threading.Thread(target=job2).start()
 
def run():
    schedule.every(10).seconds.do(job1_task)
    schedule.every(10).seconds.do(job2_task)
 
    while True:
        schedule.run_pending()
        time.sleep(1)
```
唯一要注意的是，这里面job不应当是死循环类型的，也就是说，这个线程应该有一个执行完毕的出口。一是因为线程万一僵死，会是非常棘手的问题；二是下一次定时任务还会开启一个新的线程，执行次数多了就会演变成灾难。如果schedule的时间间隔设置得比job执行的时间短，一样会线程堆积形成灾难，所以，还是需要注意一下的。

## 1.2 apscheduler

```python
# 几个核心概念
1）job
	即需要被执行的具体任务，主要对应Python中的函数或方法。在APScheduler中即可提前配置，也可以动态添加job。
2）executors
	即执行job的对象。通常可以是多线程、多进程、协程等对象。
3）jobstores
	即存储job元数据的地方。可以是memory、sqllite、mysql等。
4）trigger
	即决定任务的触发模式。通常有指定时间、指定时间间隔、指定周期策略等。
5）scheduler
	用于调度和管理上述提到的所有对象。任意一个APScheduler的实例启动的时候都需要配置这些初始参数，如果没有指定则会使用默认的值。
```
## 1.3 celery

> http://lqmblog.com/article/192