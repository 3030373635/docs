> python编程中可以使用pymysql进行数据库连接及增删改查操作，但每次连接mysql请求时，都是独立的去请求访问，比较浪费资源，而且访问数量达到一定数量时，对mysql的性能会产生较大的影响。因此实际使用中，通常会使用数据库的连接池技术，来访问数据库达到资源复用。

# 一、数据库连接池的使用

## 1.1 创建数据库连接池

```python
import time
import pymysql
import threading
from DBUtils.PooledDB import PooledDB, SharedDBConnection

POOL = PooledDB(
    creator=pymysql,  # 使用链接数据库的模块
    maxconnections=6,  # 连接池允许的最大连接数，0和None表示不限制连接数
    mincached=2,  # 初始化时，链接池中至少创建的空闲的链接，0表示不创建
    maxcached=5,  # 链接池中最多闲置的链接，0和None不限制
    maxshared=3,
    # 链接池中最多共享的链接数量，0和None表示全部共享。PS: 无用，因为pymysql和MySQLdb等模块的 threadsafety都为1，所有值无论设置为多少，_maxcached永远为0，所以永远是所有链接都共享。
    blocking=True,  # 连接池中如果没有可用连接后，是否阻塞等待。True，等待；False，不等待然后报错
    maxusage=None,  # 一个链接最多被重复使用的次数，None表示无限制
    setsession=[],  # 开始会话前执行的命令列表。如：["set datestyle to ...", "set time zone ..."]
    ping=0,
    # ping MySQL服务端，检查是否服务可用。# 如：0 = None = never, 1 = default = whenever it is requested, 2 = when a cursor is created, 4 = when a query is executed, 7 = always
    host='127.0.0.1',
    port=3306,
    user='root',
    password='123',
    database='pooldb',
    charset='utf8'
)
```

## 1.2 使用数据库连接池

```python
def func():
    conn = POOL.connection()
    cursor = conn.cursor()
    cursor.execute('select * from tb1')
    result = cursor.fetchall()
    conn.close()
```

# 二 自制sqlhelper

```python
class MySQLhelper(object):
    def __init__(self, host, port, dbuser, password, database):
        self.pool = PooledDB(
            creator=pymysql,  # 使用链接数据库的模块
            maxconnections=6,  # 连接池允许的最大连接数，0和None表示不限制连接数
            mincached=2,  # 初始化时，链接池中至少创建的空闲的链接，0表示不创建
            maxcached=5,  # 链接池中最多闲置的链接，0和None不限制
            maxshared=3,
            # 链接池中最多共享的链接数量，0和None表示全部共享。PS: 无用，因为pymysql和MySQLdb等模块的 threadsafety都为1，所有值无论设置为多少，_maxcached永远为0，所以永远是所有链接都共享。
            blocking=True,  # 连接池中如果没有可用连接后，是否阻塞等待。True，等待；False，不等待然后报错
            maxusage=None,  # 一个链接最多被重复使用的次数，None表示无限制
            setsession=[],  # 开始会话前执行的命令列表。如：["set datestyle to ...", "set time zone ..."]
            ping=0,
            # ping MySQL服务端，检查是否服务可用。# 如：0 = None = never, 1 = default = whenever it is requested, 2 = when a cursor is created, 4 = when a query is executed, 7 = always
            host=host,
            port=int(port),
            user=dbuser,
            password=password,
            database=database,
            charset='utf8'
        )

    def create_conn_cursor(self):
        conn = self.pool.connection()
        cursor = conn.cursor(pymysql.cursors.DictCursor)
        return conn,cursor

    def fetch_all(self, sql, args):
        conn,cursor = self.create_conn_cursor()
        cursor.execute(sql,args)
        result = cursor.fetchall()
        cursor.close()
        conn.close()
        return result


    def insert_one(self,sql,args):
        conn,cursor = self.create_conn_cursor()
        res = cursor.execute(sql,args)
        conn.commit()
        print(res)
        conn.close()
        return res

    def update(self,sql,args):
        conn,cursor = self.create_conn_cursor()
        res = cursor.execute(sql,args)
        conn.commit()
        print(res)
        conn.close()
        return res


sqlhelper = MySQLhelper("127.0.0.1", 3306, "root", "1233121234567", "dragon")

# sqlhelper.fetch_all("select * from user where id=%s",(1))

# sqlhelper.insert_one("insert into user VALUES (%s,%s)",("jinwangba",4))

# sqlhelper.update("update user SET name=%s WHERE  id=%s",("yinwangba",1))