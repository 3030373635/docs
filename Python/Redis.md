# Python操作redis

**安装模块**

```
pip install redis
```

**连接方式1**

```
redis-py提供两个类Redis和StrictRedis用于实现Redis的命令，StrictRedis用于实现大部分官方的命令，并使用官方的语法和命令，
Redis是StrictRedis的子类，用于向后兼容旧版本的redis-py。
import redis

r = redis.Redis()  # 默认连本地
# r = redis.Redis(host='11.12.13.14', port=6379)  # 可配置要连接的IP和端口
r.set('name', 'q1mi')
print(r.get('name'))
```

**连接方式2**

```
redis-py使用connection pool来管理对一个redis server的所有连接，避免每次建立、释放连接的开销。默认，
每个Redis实例都会维护一个自己的连接池。可以直接建立一个连接池，然后作为参数Redis，这样就可以实现多个Redis实例共享一个连接池。
import redis

pool = redis.ConnectionPool()

r = redis.Redis(connection_pool=pool)
r.set('age', 18)
print(r.get('age'))