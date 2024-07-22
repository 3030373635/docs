> 官方文档：https://www.rabbitmq.com/getstarted.html

## 1.1 介绍

> 消息队列就是数据结构中的先进先出的结构，也就是队列

```
你了解的消息队列？
	- queue
	- redis列表
	- rabbitMQ/kafka/zeroMQ

消息队列可以解决什么问题
	- 任务处理，请求数量太多，需要把消息临时放在某个地方
	- 发布订阅，一旦发布消息，所有订阅者都会收到相同的消息
	- 任务解耦
```



## 1.2 安装

```shell
#1 RabbitMQ是用Erlang语言编写的，在本教程中我们将安装最新版本的Erlang到服务器中。 Erlang在默认的YUM存储库中不可用，因此您将需要安装EPEL存储库。 运行以下命令相同。
	yum -y install epel-release
	yum -y update
#2 安装Erlang
	yum -y install erlang socat
	
#3 安装RabbitMQ
	yum install rabbitmq-server
	
```

## 1.3 设置账号密码

```shell
# 设置用户为administrator角色,前提是rabbitmq服务要启动systemctl start rabbitmq-server
sudo rabbitmqctl add_user meng 123
sudo rabbitmqctl set_user_tags meng administrator 

# 设置权限
sudo rabbitmqctl set_permissions -p "/" meng ".*" ".*" ".*"

# 然后重启rabbiMQ服务
sudo systemctl restart rabbitmq-server
 
# 然后可以使用刚才的用户远程连接rabbitmq server了。
credentials = pika.PlainCredentials("meng","123")
connection = pika.BlockingConnection(pika.ConnectionParameters('192.168.14.47',credentials=credentials))

```

## 1.4 简单使用

**生产者.py**

```python
import pika
# 有账号密码
credentials = pika.PlainCredentials("meng","123")
connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3",credentials=credentials))

# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3"))

channel = connection.channel()
# 申明一个队列（没有就创建）
queue_name = "q1"
channel.queue_declare(queue=queue_name)
body = "第二条消息".encode("utf-8")
channel.basic_publish(exchange="",routing_key=queue_name,body=body)
connection.close()
```

**消费者.py**

```python
import pika
# 有账号密码
credentials = pika.PlainCredentials("meng","123")
connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3",credentials=credentials))

# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3"))

channel = connection.channel()
# 申明一个队列（没有就创建）
queue_name = "q1"
channel.queue_declare(queue=queue_name)

def callback(ch, method, properties, body):
    print("收到消息：%s" % body.decode("utf-8"))


channel.basic_consume(queue_name,callback,auto_ack=True)


channel.start_consuming()
```

## 1.5 消息不丢失之客户端挂掉怎么处理

> 消费者拿到消息后，给RabbitMQ反馈一个收到消息的信号，然后消息才会从队列中清除掉，如果消费者拿到了消息，但是并没有反馈信号，那么该条消息将会一直存在RabbitMQ里面
>
> 在新版RabbitMQ中，auto_ack=True帮我们解决了此事，它会在你收到消息后，自动反馈一个信号，如果你想自己反馈，
>
> 需要把auto_ack=False，并且channel.basic_ack(delivery_tag = method.delivery_tag)，具体看代码

**消费者.py**

```python
import pika
# 有账号密码
credentials = pika.PlainCredentials("meng","123")
connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3",credentials=credentials))

# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3"))

channel = connection.channel()
# 申明一个队列（没有就创建）
queue_name = "q1"
channel.queue_declare(queue=queue_name)

def callback(ch, method, properties, body):
    # raise
    print("收到消息：%s" % body.decode("utf-8"))
    ch.basic_ack(delivery_tag = method.delivery_tag)

channel.basic_consume(queue_name,callback,auto_ack=False)


channel.start_consuming()
```

## 1.6 消息不丢失之服务端挂掉怎么处理

**生产者.py**

> 两处改动
>
> 1. durable=True
> 2. properties=pika.BasicProperties(delivery_mode = 2, # make message persistent)

```python
import pika
# 有账号密码
credentials = pika.PlainCredentials("meng","123")
connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3",credentials=credentials))

# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3"))

channel = connection.channel()
# 申明一个队列（没有就创建）
queue_name = "qq"
channel.queue_declare(queue=queue_name,durable=True) # 加入durable=True
body = "第二条消息".encode("utf-8")
channel.basic_publish(exchange="",routing_key=queue_name,body=body,properties=pika.BasicProperties(
    delivery_mode = 2, # make message persistent
))
connection.close()
```

**消费者.py**

> 一处改动
>
> 1. durable=True

```python
import pika
# 有账号密码
credentials = pika.PlainCredentials("meng","123")
connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3",credentials=credentials))

# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3"))

channel = connection.channel()
# 申明一个队列（没有就创建）
queue_name = "qq"
channel.queue_declare(queue=queue_name,durable=True) # 加入durable=True

def callback(ch, method, properties, body):
    # raise
    print("收到消息：%s" % body.decode("utf-8"))
    # ch.basic_ack(delivery_tag = method.delivery_tag)

channel.basic_consume(queue_name,callback,auto_ack=True)


channel.start_consuming()
```

## 1.7 任务分发

> 假设有两个消费者1，消费者2，rabbitmq默认是把消息按顺序分配，例如有5个消息，消息1分配给消费者1，消息2分配给消费者2，假设消费者2做完了，那么消息3默认不会分配给消费者2，还是要分配给消费者1，然后消息4分配给消费者2
>
> 这样很不好，消费者1，或者2，一直在做任务，另外一个空闲着，什么都做不成。
>
> 解决方法如下

**消费者.py**

> 1处改动 
>
> 1. channel.basic_qos(prefetch_count=1)

```python
import pika
# 有账号密码
credentials = pika.PlainCredentials("meng","123")
connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3",credentials=credentials))

# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3"))

channel = connection.channel()
# 申明一个队列（没有就创建）
queue_name = "qq"
channel.queue_declare(queue=queue_name,durable=True)

def callback(ch, method, properties, body):
    # raise
    print("收到消息：%s" % body.decode("utf-8"))
    # ch.basic_ack(delivery_tag = method.delivery_tag)
channel.basic_qos(prefetch_count=1)
channel.basic_consume(queue_name,callback,auto_ack=True)


channel.start_consuming()
```

## 1.8 发布订阅

> 当需要广播信息的时候，也就是多人都能收到同一条消息的时候，就需要用到发布订阅的概念了

### 1.8.1 群发

**发布者(生产者).py**

```python
import pika
# 有账号密码
credentials = pika.PlainCredentials("meng","123")
connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3",credentials=credentials))

# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3"))

channel = connection.channel()

channel.exchange_declare(exchange="meng",exchange_type='fanout')


channel.basic_publish(exchange="meng",routing_key="",body=b"hello")
connection.close()
```

**订阅者1，订阅者2......(消费者).py**

```python
import pika
# 有账号密码
credentials = pika.PlainCredentials("meng","123")
connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3",credentials=credentials))

# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3"))

channel = connection.channel()
# exchange：中间代理名字
# exchange_type：代理工作方式(fanout:发送给所有订阅者)
channel.exchange_declare(exchange="meng",exchange_type="fanout")

# 随机生成一个队列
result = channel.queue_declare(queue='',exclusive=True)
queue_name = result.method.queue

# 让代理和队列绑定关系
channel.queue_bind(queue=queue_name,exchange="meng",)



def callback(ch, method, properties, body):

    print("收到消息：%s" % body.decode("utf-8"))


channel.basic_consume(queue_name,callback,auto_ack=True)


channel.start_consuming()
```

### 1.8.2 指定某人发

> 前面的群发示例，会把消息转发给每一个人，不过有些时候，我们又需要给指定的人发，实现如下

**发布者(生产者).py**

> 一处变更
>
> 1. exchange_type='direct'

```python
import pika
# 有账号密码
credentials = pika.PlainCredentials("meng","123")
connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3",credentials=credentials))

# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3"))

channel = connection.channel()

channel.exchange_declare(exchange="meng1",exchange_type='direct')


channel.basic_publish(exchange="meng1",routing_key="Bob",body=b"hello")
connection.close()
```

**订阅者1(消费者).py**

> 两处变更：
>
> 1. exchange_type="direct"
> 2. 给队列取一个别名routing_key，比如routing_key="Jack"，生产者在对这个绑定了别名的队列发消息时，就能收到

```python
import pika
# 有账号密码
credentials = pika.PlainCredentials("meng","123")
connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3",credentials=credentials))

# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3"))

channel = connection.channel()
# exchange：中间代理名字
# exchange_type：代理工作方式(fanout:发送给所有订阅者)
channel.exchange_declare(exchange="meng1",exchange_type="direct")

# 随机生成一个队列
result = channel.queue_declare(queue='',exclusive=True)
queue_name = result.method.queue

# 让代理和队列绑定关系
channel.queue_bind(queue=queue_name,exchange="meng1",routing_key="Jack")




def callback(ch, method, properties, body):
    print("收到消息：%s" % body.decode("utf-8"))


channel.basic_consume(queue_name,callback,auto_ack=True)


channel.start_consuming()



```

**订阅者2.(消费者).py**

两处变更：

1. exchange_type="direct"
2. 给队列取一个别名routing_key，比如routing_key="Jack"，生产者在对这个绑定了别名的队列发消息时，就能收到

```python
import pika
# 有账号密码
credentials = pika.PlainCredentials("meng","123")
connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3",credentials=credentials))

# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters("192.168.1.3"))

channel = connection.channel()
# exchange：中间代理名字
# exchange_type：代理工作方式(fanout:发送给所有订阅者)
channel.exchange_declare(exchange="meng1",exchange_type="direct")

# 随机生成一个队列
result = channel.queue_declare(queue='',exclusive=True)
queue_name = result.method.queue

# 让代理和队列绑定关系
channel.queue_bind(queue=queue_name,exchange="meng1",routing_key="Bob")
channel.queue_bind(queue=queue_name,exchange="meng1",routing_key="Jack")



def callback(ch, method, properties, body):

    print("收到消息：%s" % body.decode("utf-8"))


channel.basic_consume(queue_name,callback,auto_ack=True)


channel.start_consuming()
```