## 1.1 介绍

**什么是celery**

```
Celery是一个简单、灵活且可靠的，处理大量消息的分布式系统,专注于实时处理的异步任务队列,同时也支持任务调度
自己理解就是，有三个功能，异步执行任务，延迟执行任务，定时执行任务
```

## 1.2 celery架构

<img src="https://cos.liuqm.cc/1825659-20191012154647420-564686789..png">

```
celery由三部分组成
    - 任务队列 broker
    		Celery本身不提供消息服务，但是可以方便的和第三方提供的消息中间件集成。包括，RabbitMQ, Redis等等
  
    - 执行任务 worker
    		Worker是Celery提供的任务执行的单元，worker并发的运行在分布式的系统节点中。
  
    - 存储执行任务的结果 backend
    		backend - task result store用来存储Worker执行的任务的结果，Celery支持以不同方式存储任务的结果，包括AMQP, redis等
  
大致流程：
		后台一直运行worker，将任务到broker，后台的worker检测到broker中有任务就会立刻取出来执行任务，将结果交到backend，也就是图中的task result store
```

## 1.3 使用

> 说明：以下代码全是基于mac操作系统，并且celery是5.0以下跑的，如果非mac或者linux系统，需要额外配置，并且celery5.0以上删除了部分代码，也需要额外设置才行

### 1.3.1 简单使用

**celery_app.py**

```python
"""
celery由三部分组成
    - 任务队列 broker
    - 执行任务 worker
    - 存储执行任务的结果 backend
"""

from celery import Celery

broker = "redis://:lqm576576@localhost:6379/1"
backend = "redis://:lqm576576@localhost:6379/2"
app = Celery(__name__, broker=broker, backend=backend)


# 将任务添加到broker
@app.task
def add(a, b):
    return a + b
```

**send_task.py**

```python
from celery_app import add

task_id = add.delay(5, 4)  # 将broker中的任务发送到worked中
print(task_id)

```

**result.py**

```python
from celery.result import AsyncResult
from celery_app import app

async = AsyncResult(id="30593966-81ef-476c-a19d-f4df80656de6", app=app)

if async.successful():
    result = async.get()
    print(result)
    # result.forget() # 将结果删除
elif async.failed():
    print('执行失败')
elif async.status == 'PENDING':
    print('任务等待中被执行')
elif async.status == 'RETRY':
    print('任务异常后正在重试')
elif async.status == 'STARTED':
    print('任务已经开始被执行')
```

```python
# 启动一个worker
celery -A celery_app worker -l info
```

### 1.3.2 多任务结构使用

```python
├── proj # celery相关文件夹
│   ├── celery.py   # celery连接和配置相关文件,必须叫这个名字
│   └── config.py   #  配置
│   └── task1.py    #  任务
│   └── task2.py    #  任务
├── result.py  # 检查结果
└── send.py    # 触发任务
```

**celery.py**

```python
"""
celery由三部分组成
    - 任务队列 broker
    - 执行任务 worker
    - 存储执行任务的结果 backend
"""

from celery import Celery

app = Celery(__name__)


# 引入配置
app.config_from_object('proj.config')

```

**config.py**

```python
from celery.schedules import crontab
from datetime import timedelta

# 使用redis存储任务
broker_url = "redis://:lqm576576@localhost:6379/1"

# 使用redis存储结果
result_backend = "redis://:lqm576576@localhost:6379/2"

# 时区设置
timezone = 'Asia/Shanghai'

# 导入任务所在文件
imports = [
    'proj.task1',
    'proj.task2'
]

# 延迟执行任务的配置
beat_schedule = {
    '随意命名1': {
        # 具体需要执行的函数
        'task': 'proj.task1.add',
        # 定时时间，每分钟执行一次
        'schedule': crontab(minute='*/1'),
        # 执行的函数需要的参数
        'args': (10, 20)
    },
    '随意命名2': {
        # 具体需要执行的函数
        'task': 'proj.task2.mut',
        # 定时时间，每10秒执行一次
        'schedule': timedelta(seconds=10),
        # 执行的函数需要的参数
        'args': (20, 30)
    }
}

```

**task1.py**

```python
from .celery import app
@app.task
def add(a,b):
    return a + b
```

**task2.py**

```python
from .celery import app

@app.task
def mut(a,b):
    return a * b
```

**result.py**

```python
from celery.result import AsyncResult
from celery_task.celery import app

async = AsyncResult(id="08eb2778-24e1-44e4-a54b-56990b3519ef", app=app)

if async.successful():
    result = async.get()
    print(result)
    # result.forget() # 将结果删除,执行完成，结果不会自动删除
    # async.revoke(terminate=True)  # 无论现在是什么时候，都要终止
    # async.revoke(terminate=False) # 如果任务还没有开始执行呢，那么就可以终止。
elif async.failed():
    print('执行失败')
elif async.status == 'PENDING':
    print('任务等待中被执行')
elif async.status == 'RETRY':
    print('任务异常后正在重试')
elif async.status == 'STARTED':
    print('任务已经开始被执行')
```

**send.py**

```python
from proj.tasks import *
from datetime import datetime,timedelta

# 异步执行任务
# id1 = add.delay(10,20)
# id2 = mut.delay(10,20)


# 延迟执行任务，这里我们延迟十秒执行
#id3 = add.apply_async(args=(10,20),countdown=10)
#id4 = mut.apply_async(args=(10,20),countdown=10)

# 定时任务

```

```python
# 启动一个beat
celery  -A proj beat -l info
```

## 1.4 flower

```python
# Celery提供了一个工具flower，将各个任务的执行情况、各个worker的健康状态进行监控并以可视化的方式展现。
pip install flower
celery -A proj flower --address=0.0.0.0 --port=5555