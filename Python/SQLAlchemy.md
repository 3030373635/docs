## 1.1 介绍

```
什么是ORM？
	对象关系映射，可以看成，class类对应数据库中的表，class类的属性对应数据库中表的字段，class类的方法对应着数据库中的各种操作

怎么用？
	python中使用sqlachemy模块进行ORM操作
```

## 1.2 使用

```python
# 重要说明：sqlachemy的一些内置数据类型

#1、如果查询语句在最后面只要没有执行all()或者first()，那么返回的数据类型一定是<class 'sqlalchemy.orm.query.Query'>，如果打印该对象，结果是该条语句所表达的sql语句
user_list = db_session.query(User).filter(User.name=="meng3") 
print(type(user_list)) # <class 'sqlalchemy.orm.query.Query'>
print(user_list) # SELECT user.id AS user_id, user.name AS user_name FROM user WHERE user.name = %(name_1)s

user_list = db_session.query(User) 
print(type(user_list)) # <class 'sqlalchemy.orm.query.Query'>
print(user_list) # SELECT user.id AS user_id, user.name AS user_name FROM user


#2、如果查询语句后面加了all()或者first()，那么返回的数据类型是该model的实例化对象
user_list = db_session.query(User).filter(User.name=="meng3").first()
print(type(user_list)) # <class 'create_table.User'>

```

### 1.2.2 单表操作

**创建表**

```python
from sqlalchemy.ext.declarative import declarative_base
# 导入orm对应数据库数据类型的字段
from sqlalchemy import Column,Integer,String

# 创建orm模型基类
Base = declarative_base()


# 创建orm对象
class User(Base):
    __tablename__ = "user" # 表名
    id = Column(Integer,primary_key=True)
    name = Column(String(32),index=True)

# 创建数据库连接
from sqlalchemy import create_engine
engine = create_engine('mysql+pymysql://root:lqm576576@localhost:3306/sqlalchemy?charset=utf8')

# 建表
Base.metadata.create_all(engine)


```

**增加数据**

```python
# 拿到数据库连接
from create_table import engine,User

# 创建会话,打开会话窗口,相当于Navicat中的新建查询吧
from sqlalchemy.orm import sessionmaker
session = sessionmaker(engine)
db_session = session()

"""
    # 增加单条数据 insert into user(name) values("meng")
    user_obj1 = User(name="meng")
    user_obj2 = User(name="mengmeng")
    db_session.add(user_obj1)
    db_session.add(user_obj2)
"""

"""
    # 批量增加数据
    db_session.add_all([
        User(name="meng2"),
        User(name="meng3"),
        User(name="meng3"),
    ])
    db_session.commit()
    db_session.close()
"""


# 执行会话窗口
db_session.commit()
db_session.close()
```

**删除数据**

```python
# 拿到数据库连接
from create_table import engine,User

# 创建会话,打开会话窗口,相当于Navicat中的新建查询吧
from sqlalchemy.orm import sessionmaker
session = sessionmaker(engine)
db_session = session()



# 删除数据 delete from user where user.id == 5
res = db_session.query(User).filter(User.id==5).delete()
print(res)


# 查询sql语句
sql1 = db_session.query(User).filter(User.id==5)
print(sql1)

db_session.commit()
db_session.close()



```

**改变数据**

```python
# 拿到数据库连接
from create_table import engine,User

# 创建会话,打开会话窗口,相当于Navicat中的新建查询吧
from sqlalchemy.orm import sessionmaker
session = sessionmaker(engine)
db_session = session()


# 更新数据 update user set user.name == "meng2" and ....
res = db_session.query(User).filter(User.name=="meng2").update({"name":"mengdsb"})
print(db_session.query(User).filter(User.name=="meng2"))



# 查询sql语句
sql1 = db_session.query(User).filter(User.name=="meng2")
print(sql1)


db_session.commit()
db_session.close()
```

**查询数据**

```python
# 拿到数据库连接和model
from create_table import engine,User

# 创建会话,打开会话窗口,相当于Navicat中的新建查询吧
from sqlalchemy.orm import sessionmaker
session = sessionmaker(engine)
db_session = session()


# 查询所有 select * from user
user_list = db_session.query(User).all()
for i in user_list:
    print(i.name,i.id)

# 带条件的查询filter filter_by
user_list = db_session.query(User).filter(User.id < 4).all()
print(user_list)

user_list = db_session.query(User).filter_by(id=4).first()
print(user_list)

# 查看sql语句
sql1 = db_session.query(User).filter(User.id < 4)
sql2 = db_session.query(User)
print(sql1)
print(sql2)


```

**更多查询操作**

> 我这里只写了几个常见的，更多参考：https://www.cnblogs.com/DragonFire/p/10166527.html

```python
from my_create_table import User,engine
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(engine)
db_session = Session()


#1 查询所有数据 select * from user
r1 = db_session.query(User).all()

#2 查询单列数据 select name from user
r1 = db_session.query(User.name).all()

#3 条件查询
"""
select * from user where ....

- filter 
    - filter(User.name=="meng")
  
- filter_by
    filter_by(name="meng") 
  

  
这两个函数都可以，只不过是参数形式不一样
"""
#3.1 select * from user where user.name="meng"
ret = db_session.query(User).filter(User.name=="meng").all()
#3.2 and or
from sqlalchemy.sql import and_ , or_
# select * from user where user.id > 3 and user.name = "meng"
ret = db_session.query(User).filter(and_(User.id > 3, User.name == 'meng')).all()
# select * from user where user.id > 3 or user.name = "meng"
ret = db_session.query(User).filter(or_(User.id < 2, User.name == 'meng')).all()



#4 取别名as
# select name as username,id from user
r2 = db_session.query(User.name.label('username'), User.id).first()


#5 执行原生sql，它这个变量前面好像要加:
from sqlalchemy.sql import text
r7 = db_session.query(User).from_statement(text("SELECT * FROM user where name=:username")).params(username='meng').all()


#6 排序
user_list = db_session.query(User).order_by(User.id).all()
```

**更多修改操作**

```python
from my_create_table import User,engine
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(engine)
db_session = Session()

# 直接修改
db_session.query(User).filter(User.id > 0).update({"name" : "099"})

#在原有值基础上添加 - 1
db_session.query(User).filter(User.id > 0).update({User.name: User.name + "099"}, synchronize_session=False)

#在原有值基础上添加 - 2
db_session.query(User).filter(User.id > 0).update({"age": User.age + 1}, synchronize_session="evaluate")
db_session.commit()


```

### 1.2.3 一对多操作

**创建数据表**

```python
# 一对多建表操作

from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import create_engine
from sqlalchemy.orm import relationship
# 导入orm对应数据库数据类型的字段
from sqlalchemy import Column,Integer,String,ForeignKey



# 创建orm模型基类
Base = declarative_base()




# 创建orm对象
class Student(Base):
    __tablename__ = "student" # 表名
    id = Column(Integer,primary_key=True)
    name = Column(String(32))
    school_id = Column(Integer,ForeignKey("school.id"))
    stu2sch = relationship("School",backref="sch2stu")

class School(Base):
    __tablename__ = "school" # 表名
    id = Column(Integer,primary_key=True)
    name = Column(String(32),index=True)


engine = create_engine('mysql+pymysql://root:lqm576576@localhost:3306/sqlalchemy?charset=utf8')


# 建表
Base.metadata.create_all(engine)
```

**增加数据**

```python
from create_table_foreignkey import engine,Student,School

# 创建会话,打开会话窗口,相当于Navicat中的新建查询吧
from sqlalchemy.orm import sessionmaker
session = sessionmaker(engine)
db_session = session()


# 添加数据 - 笨方法
sch_obj = School(name="成都工业学院")
db_session.add(sch_obj)
sch_obj =  db_session.query(School).filter(School.name=="成都工业学院").first()
stu_obj = Student(name="meng",school_id=sch_obj.id)
db_session.add(stu_obj)
db_session.commit()

# 添加数据 - 反向（relationship）
sch_obj = School(name="川大")
sch_obj.sch2stu = [Student(name="meng"),Student(name="tom")]
db_session.add(sch_obj)
db_session.commit()


# 添加数据 - 正向（relationship）

stu_obj = Student(name="jack",stu2sch=School(name="北大"))
db_session.add(stu_obj)
db_session.commit()





```

**删除数据**

```python
from create_table_foreignkey import engine,Student,School

# 创建会话,打开会话窗口,相当于Navicat中的新建查询吧
from sqlalchemy.orm import sessionmaker
session = sessionmaker(engine)
db_session = session()

# 删除 删除校区为北大的所有学生
sch_obj = db_session.query(School).filter(School.name=="北大").first()
stu_obj = db_session.query(Student).filter(Student.school_id==sch_obj.id).delete()

db_session.commit()
db_session.close()
```

**修改数据**

```python
from create_table_foreignkey import engine,Student,School

# 创建会话,打开会话窗口,相当于Navicat中的新建查询吧
from sqlalchemy.orm import sessionmaker
session = sessionmaker(engine)
db_session = session()

# 修改数据 将meng的学校改为北大
sch_obj = db_session.query(School).filter(School.name=="北大").first()
stu_obj = db_session.query(Student).filter(Student.name=="meng").update({"school_id":sch_obj.id})

db_session.commit()
db_session.close()
```

**查询数据**

```python
from create_table_foreignkey import engine,Student,School

# 创建会话,打开会话窗口,相当于Navicat中的新建查询吧
from sqlalchemy.orm import sessionmaker
session = sessionmaker(engine)
db_session = session()

# 查询数据 - 正向
stu = db_session.query(Student).all()
for i in stu:
    print(i.id,i.name,i.school_id,i.stu2sch.name)

# 查询数据 - 反向

sch = db_session.query(School).all()
for i in sch:
    print(i.id,i.name,[stu_obj.name for stu_obj in i.sch2stu])
```

### 1.2.4 多对多操作

**创建数据表**

```python
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship


class Girl_Boy(Base):
    __tablename__ = "girl_boy"
    id = Column(Integer, primary_key=True)
    girl_id = Column(Integer, ForeignKey("girl.id"))
    boy_id = Column(Integer, ForeignKey("boy.id"))


class Girl(Base):
    __tablename__ = "girl"
    id = Column(Integer, primary_key=True)
    name = Column(String(32), index=True)

    # 创建关系,这个字段随便写在哪个model都行
    girl2boy = relationship("Boy", secondary="Girl_Boy", backref="boy2girl")


class Boy(Base):
    __tablename__ = "boy"
    id = Column(Integer, primary_key=True)
    name = Column(String(32), index=True)


from sqlalchemy import create_engine

engine = create_engine('mysql+pymysql://root:lqm576576@localhost:3306/sqlalchemy?charset=utf8')

Base.metadata.create_all(engine)

```

**增加数据**

> 因为是粘贴的人家的代码，执行的时候报错了，也不知道怎么解决，就暂时把代码留在这里吧

```python
from create_table_m2m import Girl,Boy,Girl_Boy,engine

# 创建连接
from sqlalchemy.orm import sessionmaker
# 创建数据表操作对象 sessionmaker
DB_session = sessionmaker(engine)
db_session = DB_session()

# 1.通过Boy添加Girl和Hotel数据
boy = Boy(name="meng")
boy.boy2girl = [Girl(name="王祖贤"),Girl(name="周慧敏")]

db_session.add(boy)
db_session.commit()

# 2.正向 - 通过Girl添加Boy和Hotel数据
girl = Girl(name="珊珊")
girl.girl2boy = [Boy(name="jack")]
db_session.add(girl)
db_session.commit()
```

**查询数据**

```python
from my_M2M import Girl,Boy,Hotel,engine

# 创建连接
from sqlalchemy.orm import sessionmaker
# 创建数据表操作对象 sessionmaker
DB_session = sessionmaker(engine)
db_session = DB_session()

# 1.通过Boy查询约会过的所有Girl
hotel = db_session.query(Boy).all()
for row in hotel:
     for row2 in row.girl2boy:
         print(row.name,row2.name)

# 2.通过Girl查询约会过的所有Boy
hotel = db_session.query(Girl).all()
for row in hotel:
     for row2 in row.boys:
         print(row.name,row2.name)