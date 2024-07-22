---
title: CURD
---


# 一、数据库

```python
#1、增
	use config #如果数据库不存在，则创建数据库，否则切换到指定数据库。

#2、查
  show dbs #查看所有
  可以看到，我们刚创建的数据库config并不在数据库的列表中， 要显示它，我们需要向config数据库插入一些数据。
  db.table1.insert({'a':1})

#3、删
  use config 				#先切换到要删的库下
  db.dropDatabase() #删除当前库
```

# 二、集合

> 相当于关系型数据库的表

```python
#1、增
  当第一个文档插入时，集合就会被创建
  > use database1
  > db.table1.insert({'a':1})
  > db.table2.insert({'b':2})

#2、查
  > show collections
    table1
    table2

#3、删
  > db.table1.drop()
    true
  > show collections
```

# 三、文档

## 3.1 增

```python
#1、没有指定_id则默认ObjectId类型,_id不能重复，且在插入后不可变

#2、插入单条
user0={
    "name":"meng",
    "age":null,
  	"bool":true,
  	"age":10.5,
  	"time":new Date(),
  	"pattern":/^egon.*?nb$/i,
    'hobbies':['music','read','dancing'],
    'addr':{
        'country':'China',
        'city':'BJ'
    }
}

db.test.insertOne(user0)
db.test.find()

#3、插入多条
user1={
    "_id":1,
    "name":"alex",
    "age":10,
    'hobbies':['music','read','dancing'],
    'addr':{
        'country':'China',
        'city':'weifang'
    }
}

user2={
    "_id":2,
    "name":"wupeiqi",
    "age":20,
    'hobbies':['music','read','run'],
    'addr':{
        'country':'China',
        'city':'hebei'
    }
}


user3={
    "_id":3,
    "name":"yuanhao",
    "age":30,
    'hobbies':['music','drink'],
    'addr':{
        'country':'China',
        'city':'heibei'
    }
}

user4={
    "_id":4,
    "name":"jingliyang",
    "age":40,
    'hobbies':['music','read','dancing','tea'],
    'addr':{
        'country':'China',
        'city':'BJ'
    }
}

user5={
    "_id":5,
    "name":"jinxin",
    "age":50,
    'hobbies':['music','read',],
    'addr':{
        'country':'China',
        'city':'henan'
    }
}
db.user.insertMany([user1,user2,user3,user4,user5])
```

## 3.2 删

> remove() 方法已经过时了，现在官方推荐使用 deleteOne() 和 deleteMany() 方法。

```python
#1、删除多个中的第一个
db.user.deleteOne({ 'age': 8 })

#2、删除国家为China的全部
db.user.deleteMany( {'addr.country': 'China'} ) 

#3、删除全部
db.user.deleteMany({}) 
```

## 3.3 改

```python
# 语法格式如下：
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
参数说明：对比update db1.t1 set name='EGON',sex='Male' where name='egon' and age=18;

query : 相当于where条件。
update : update的对象和一些更新的操作符（如$,$inc...等，相当于set后面的，默认只改第一条记录，改更多需要配合multi使用
upsert : 可选，默认为false，代表如果不存在update的记录不更新也不插入，设置为true代表插入。
multi : 可选，默认为false，代表只更新找到的第一条记录，设为true,代表更新找到的全部记录。
writeConcern :可选，抛出异常的级别。
```

### 覆盖式修改

> 只保留修改字段

```python
db.user.update({'age':20},{"name":"Wxx","hobbies_count":3})
是用{"name":"Wxx","hobbies_count":3} 覆盖原来的记录，只保留这两个字段
```

### 替换式修改

> 这个只改自己需要改的字段，不会清除其他字段

```python
# update db1.user set  name="WXX" where id = 2
db.user.update({'_id':2},{"$set":{"name":"WXX",}})

# 没有匹配成功则新增一条{"upsert":true}
db.user.update({'_id':6},{"$set":{"name":"egon","age":18}},{"upsert":true})

# 默认只改匹配成功的第一条,{"multi":改多条}
db.user.update({'_id':{"$gt":4}},{"$set":{"age":28}})
db.user.update({'_id':{"$gt":4}},{"$set":{"age":38}},{"multi":true})

# 修改内嵌文档，把名字为alex的人所在的地址国家改成Japan
db.user.update({'name':"alex"},{"$set":{"addr.country":"Japan"}})

# 把名字为alex的人的地2个爱好改成piao
db.user.update({'name':"alex"},{"$set":{"hobbies.1":"piao"}})

# 删除alex的爱好,$unset
db.user.update({'name':"alex"},{"$unset":{"hobbies":""}})
```

### $inc

```python
#增加和减少：$inc
#1、所有人年龄增加一岁
db.user.update({},
    {
        "$inc":{"age":1}
    },
    {
        "multi":true
    }
    )
#2、所有人年龄减少5岁
db.user.update({},
    {
        "$inc":{"age":-5}
    },
    {
        "multi":true
    }
    )
```

### $push,\$pop,$pul

```python
#添加删除数组内元素：$push,$pop,$pul

#1、为名字为yuanhao的人添加一个爱好read
db.user.update({"name":"yuanhao"},{"$push":{"hobbies":"read"}})

#2、为名字为yuanhao的人一次添加多个爱好tea，dancing
db.user.update({"name":"yuanhao"},{"$push":{
    "hobbies":{"$each":["tea","dancing"]}
}})


#3、从数组末尾删除一个元素
db.user.update({"name":"yuanhao"},{"$pop":{
    "hobbies":1}
})

#4、{"$pop":{"key":-1}} 从头部删除
db.user.update({"name":"yuanhao"},{"$pop":{
    "hobbies":-1}
})

#5、按照条件删除元素,："$pull" 把符合条件的统统删掉，而$pop只能从两端删
db.user.update({'addr.country':"China"},{"$pull":{
    "hobbies":"read"}
},
{
    "multi":true
}
)
```

### $addToSet

```python
#数组避免重复添加可以使用："$addToSet"

db.urls.insert({"_id":1,"urls":[]})

db.urls.update({"_id":1},{"$addToSet":{"urls":'http://www.baidu.com'}})
db.urls.update({"_id":1},{"$addToSet":{"urls":'http://www.baidu.com'}})
db.urls.update({"_id":1},{"$addToSet":{"urls":'http://www.baidu.com'}})

db.urls.update({"_id":1},{
    "$addToSet":{
        "urls":{
        "$each":[
            'http://www.baidu.com',
            'http://www.baidu.com',
            'http://www.xxxx.com'
            ]
            }
        }
    }
)
```

## 3.4 查

```python
# 语法格式如下
db.collection.find(query, projection)
query ：可选，使用查询操作符指定查询条件
projection ：可选，指定返回的键。查询时如果需要文档中所有键值， 只需省略该参数即可（默认省略）。

# 比如下面这句，查询name为meng，并且显示字段name和height，不显示age，其他没设置的默认也是1，展示出来
db.collection.find({"name":"meng"}, {"name":1,"age":0,"height":1})
# 如果你需要以易读的方式来读取数据，可以使用 pretty() 方法
db.collection.find().pretty()

# 查找所有
db.collection.find() 

# 查找一个，与find用法一致，只是只取匹配成功的第一个
db.user.findOne({"_id":{"$gt":3}})
```

### 3.4.1 比较运算

```python
# SQL：=,!=,>,<,>=,<=
# MongoDB：{key:value}代表什么等于什么,"$ne","$gt","$lt","gte","lte",其中"$ne"能用于所有数据类型

#1、select * from db1.user where name = "alex";
db.user.find({'name':'alex'})

#2、select * from db1.user where name != "alex";
db.user.find({'name':{"$ne":'alex'}})

#3、select * from db1.user where id > 2;
db.user.find({'_id':{'$gt':2}})

#4、select * from db1.user where id < 3;
db.user.find({'_id':{'$lt':3}})

#5、select * from db1.user where id >= 2;
db.user.find({"_id":{"$gte":2,}})

#6、select * from db1.user where id <= 2;
db.user.find({"_id":{"$lte":2}})
```

### 3.4.2 逻辑运算

```python
# SQL：and，or，not
# MongoDB：$and，$or，$not,字典中逗号分隔的多个条件是and关系，也可以使用$and,一个字典表示一种条件

#1、select * from db1.user where id >= 2 and id < 4;
# 方式一 
db.user.find({'$and':[
    {"id":{"$gte":2}},
  	{"id":{"$lt":4}},
  ]
})
# 方式二 
db.user.find({'_id':{"$gte":2,"$lt":4}})

#2、select * from db1.user where id >= 5 or name = "alex";
db.user.find({
    "$or":[
        {'_id':{"$gte":5}},
        {"name":"alex"}
        ]
})

#3、select * from db1.user where id % 2=1;
db.user.find({'_id':{"$mod":[2,1]}})

#4、上题，取反
db.user.find({'_id':{"$not":{"$mod":[2,1]}}})
```

### 3.4.3 成员运算

```python
# SQL：in，not in
# MongoDB："$in","$nin"

#1、select * from db1.user where age in (20,30,31);
db.user.find({"age":{"$in":[20,30,31]}})

#2、select * from db1.user where name not in ('alex','yuanhao');
db.user.find({"name":{"$nin":['alex','yuanhao']}})
```

### 3.4.4 正则匹配

```python
# SQL: regexp 正则
# MongoDB: 方式一：{content:{"$regex":/正则表达/i} 方式二：{content:"/正则表达/i"}

#1、select * from db1.user where name regexp '^j.*?(g|n)$';
db.user.find({'name':/^j.*?(g|n)$/i})

#2、select * from user where name like "%花%";
db.user.find({"name":"/花/"})
```

### 3.4.5 显示指定字段

```python
#1、select name,age from db1.user where id=3;
db.user.find({'_id':3},{'_id':0,'name':1,'age':1})

# 1表示显示该子段，0表示不显示该字段
```

### 3.4.6 基于数组的查询

```python
# 例如
{
	"_id" : 1,
	"name" : "alex",
	"age" : 10,
	"hobbies" : [
		"music",
		"read",
		"dancing"
	],
	"addr" : {
		"country" : "China",
		"city" : "weifang"
	}
}

#1、查看有dancing爱好的人
db.user.find({'hobbies':'dancing'})

#2、查看既有dancing爱好又有tea爱好的人
db.user.find({
    'hobbies':{
        "$all":['dancing','tea']
        }
})

#3、查看第4个爱好为tea的人
db.user.find({"hobbies.3":'tea'})

#4、查看所有人最后两个爱好
db.user.find({},{'hobbies':{"$slice":-2},"age":0,"_id":0,"name":0,"addr":0})

#5、切片
db.user.find({},{'hobbies':{"$slice":[1,2]},"age":0,"_id":0,"name":0,"addr":0}) # 查看所有人的第2个到第3个爱好
db.blog.find({},{'comments':{"$slice":-2}}).pretty() #查询最后两个
db.blog.find({},{'comments':{"$slice":[1,2]}}).pretty() #查询1到2
```

### 3.4.7 基于内嵌文档的查询

```sql
# 例如
{
	"_id" : 1,
	"name" : "alex",
	"age" : 10,
	"hobbies" : [
		"music",
		"read",
		"dancing"
	],
	"addr" : {
		"country" : "China",
		"city" : "weifang"
	}
}

# 查询addr中country为china的
db.user.find(
{"add.country":"china"}
)

```

### 3.4.8 排序

```python
# 排序:--1代表升序，-1代表降序
db.user.find().sort({"name":1,})
db.user.find().sort({"age":-1,'_id':1})
```

### 3.4.9 分页

```python
# 分页:limit代表取多少个document，skip代表跳过前多少个document。 
db.user.find().sort({'age':1}).skip(2).limit(1)
# 注意事项：执行顺序sort(),skip(),limit()
```

### 3.4.10 获取数量

```python
db.user.find({'age':{"$gt":30}}).count()
```

## 3.5 聚合

> 聚合框架是MongoDB的高级查询语言，它允许我们通过转换和合并多个文档中的数据来生成新的单个文档中不存在的信息。
>
> 聚合框架：可以使用多个构件创建一个管道，上一个构件的结果传给下一个构件。这些构件包括：筛选($match)、投射($project)、分组($group)、排序($sort)、限制($limit)、跳过($skip)等等，不同的管道操作符可以任意组合，重复使用

```python
# 语法格式如下
db.collection.aggregate([])
```


| **命令** | **功能描述**                               |
| -------- | ------------------------------------------ |
| $project | 指定输出文档里的字段.                      |
| $match   | 选择要处理的文档，与fine()类似。           |
| $limit   | 限制传递给下一步的文档数量。               |
| $skip    | 跳过一定数量的文档。                       |
| $unwind  | 扩展数组，为每个数组入口生成一个输出文档。 |
| $group   | 根据key来分组文档。                        |
| $sort    | 排序文档。                                 |
| $geoNear | 选择某个地理位置附近的的文档。             |
| $out     | 把管道的结果写入某个集合。                 |
| $redact  | 控制特定数据的访问。                       |
| $lookup  | 多表关联（3.2版本新增）                    |

**准备数据**

```python
from pymongo import MongoClient
import datetime

client=MongoClient('mongodb://root:123@localhost:27017')
table=client['db1']['emp']


l=[
('egon','male',18,'20170301','老男孩驻沙河办事处外交大使',7300.33,401,1), #以下是教学部
('alex','male',78,'20150302','teacher',1000000.31,401,1),
('wupeiqi','male',81,'20130305','teacher',8300,401,1),
('yuanhao','male',73,'20140701','teacher',3500,401,1),
('liwenzhou','male',28,'20121101','teacher',2100,401,1),
('jingliyang','female',18,'20110211','teacher',9000,401,1),
('jinxin','male',18,'19000301','teacher',30000,401,1),
('成龙','male',48,'20101111','teacher',10000,401,1),

('歪歪','female',48,'20150311','sale',3000.13,402,2),#以下是销售部门
('丫丫','female',38,'20101101','sale',2000.35,402,2),
('丁丁','female',18,'20110312','sale',1000.37,402,2),
('星星','female',18,'20160513','sale',3000.29,402,2),
('格格','female',28,'20170127','sale',4000.33,402,2),

('张野','male',28,'20160311','operation',10000.13,403,3), #以下是运营部门
('程咬金','male',18,'19970312','operation',20000,403,3),
('程咬银','female',18,'20130311','operation',19000,403,3),
('程咬铜','male',18,'20150411','operation',18000,403,3),
('程咬铁','female',18,'20140512','operation',17000,403,3)
]

for n,item in enumerate(l):
    d={
        "_id":n,
        'name':item[0],
        'sex':item[1],
        'age':item[2],
        'hire_date':datetime.datetime.strptime(item[3],'%Y%m%d'),
        'post':item[4],
        'salary':item[5]
    }
    table.save(d)
```

### 3.5.1 $match

```python
{"$match":{"字段":"条件"}},可以使用任何常用查询操作符$gt,$lt,$in等

#例1、select * from db1.emp where post='teacher';
db.emp.aggregate({"$match":{"post":"teacher"}})

#例2、select _id,avg(salary) as avg_salary from db1.emp where id > 3 group by post
db.emp.aggregate(
    {"$match":{"_id":{"$gt":3}}},
    {"$group":{"_id":"$post",'avg_salary':{"$avg":"$salary"}}}
)

#例3、select _id,avg(salary) as avg_salary from db1.emp where id > 3 group by post having avg(salary) > 10000;  
db.emp.aggregate(
    {"$match":{"_id":{"$gt":3}}},
    {"$group":{"_id":"$post",'avg_salary':{"$avg":"$salary"}}},
    {"$match":{"avg_salary":{"$gt":10000}}}
)
```

### 3.5.2 $project

```python
{"$project":{"要保留的字段名":1,"要去掉的字段名":0,"新增的字段名":"表达式"}}

#1、select name,post,(age+1) as new_age from db1.emp;
db.emp.aggregate(
    {"$project":{
        "name":1,
        "post":1,
        "new_age":{"$add":["$age",1]}
        }
})

#2、表达式之数学表达式
{"$add":[expr1,expr2,...,exprN]} #相加
{"$subtract":[expr1,expr2]} #第一个减第二个
{"$multiply":[expr1,expr2,...,exprN]} #相乘
{"$divide":[expr1,expr2]} #第一个表达式除以第二个表达式的商作为结果
{"$mod":[expr1,expr2]} #第一个表达式除以第二个表达式得到的余数作为结果

#3、表达式之日期表达式:$year,$month,$week,$dayOfMonth,$dayOfWeek,$dayOfYear,$hour,$minute,$second
#例如：select name,date_format("%Y") as hire_year from db1.emp
db.emp.aggregate(
    {"$project":{"name":1,"hire_year":{"$year":"$hire_date"}}}
)

#例如查看每个员工的工作多长时间
db.emp.aggregate(
    {"$project":{"name":1,"hire_period":{
        "$subtract":[
            {"$year":new Date()},
            {"$year":"$hire_date"}
        ]
    }}}
)


#4、字符串表达式
{"$substr":[字符串/$值为字符串的字段名,起始位置,截取几个字节]}
{"$concat":[expr1,expr2,...,exprN]} #指定的表达式或字符串连接在一起返回,只支持字符串拼接
{"$toLower":expr}
{"$toUpper":expr}

db.emp.aggregate( {"$project":{"NAME":{"$toUpper":"$name"}}})

#5、逻辑表达式
$and
$or
$not
```

### 3.5.3 $group

```python
{"$group":{"_id":分组字段,"新的字段名":聚合操作符}}

#1、将分组字段传给$group函数的_id字段即可
{"$group":{"_id":"$sex"}} #按照性别分组
{"$group":{"_id":"$post"}} #按照职位分组
{"$group":{"_id":{"state":"$state","city":"$city"}}} #按照多个字段分组，比如按照州市分组


#2、分组后聚合得结果,类似于sql中聚合函数的聚合操作符：$sum、$avg、$max、$min、$first、$last
#例1：select post,max(salary) as  max_salary from db1.emp group by post; 
db.emp.aggregate({"$group":{"_id":"$post","max_salary":{"$max":"$salary"}}})

#例2：取每个部门最大薪资与最低薪资
db.emp.aggregate({"$group":{"_id":"$post","max_salary":{"$max":"$salary"},"min_salary":{"$min":"$salary"}}})

#例3：如果字段是排序后的，那么$first,$last会很有用,比用$max和$min效率高
db.emp.aggregate({"$group":{"_id":"$post","first_id":{"$first":"$_id"}}})

#例4：求每个部门的总工资
db.emp.aggregate({"$group":{"_id":"$post","count":{"$sum":"$salary"}}})

#例5：求每个部门的人数
db.emp.aggregate({"$group":{"_id":"$post","count":{"$sum":1}}})


#例6：查询岗位名以及各岗位内的员工姓名:select post,group_concat(name) from db1.emp group by post;
db.emp.aggregate({"$group":{"_id":"$post","names":{"$push":"$name"}}}) # 可以重复
db.emp.aggregate({"$group":{"_id":"$post","names":{"$addToSet":"$name"}}}) # 不重复
```

### 3.5.4 \$sort,\$limit,\$skip

```python
{"$sort":{"字段名":1,"字段名":-1}} # 1升序，-1降序
{"$limit":n} # 取多少个文档
{"$skip":n} # 跳过多少个文档，我自己实践出来的好像是，先skip，后再limit

#例1、取平均工资最高的前两个部门
db.emp.aggregate(
  {
      "$group":{"_id":"$post","平均工资":{"$avg":"$salary"}}
  },
  {
      "$sort":{"平均工资":-1}
  },
  {
      "$limit":2
  }
)
```

### 3.5.5 $lookup

> 俗称连表，详细请看https://www.cnblogs.com/xuliuzai/p/10055535.html

```python
# 语法格式
{
   $lookup:
     {
       from: <collection to join>,
       localField: <field from the input documents>,
       foreignField: <field from the documents of the "from" collection>,
       as: <output array field>
     }
}
# 类比：select * from student join school on student.school_id = school.id
{
  "$lookup":{
    "from":"school",
    "localField":"school_id",
    "foreignField":"id",
    "as":"from_school"
  }
}
# 这里需要注意的是，连表后的文档就会取as对应的名字from_school，然后添加到student集合里面
```

```python
# 数据准备
db.school.insertMany([
    {"_id":1,"name":"成都工业大学"},
    {"_id":2,"name":"家里蹲大学"}
])

db.user.insertMany([
    {"name":"meng","school_id":1},
    {"name":"刘xx","school_id":2},
    {"name":"伍xx","school_id":2},
    {"name":"彭xx","school_id":2}
])


# 连表
db.user.aggregate(
    {
        "$lookup":{
            "from":"school",
            "localField":"school_id",
            "foreignField":"_id",
            "as":"from_shool"
        }
    }
)