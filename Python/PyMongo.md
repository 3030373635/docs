---
title: pymongo
---



# 一、pymongo

```python
from pymongo import MongoClient

#1、链接
client=MongoClient('mongodb://root:123@localhost:27017/')
# client = MongoClient('localhost', 27017)

#2、use 数据库
db=client['db2'] #等同于：client.db1

#3、查看库下所有的集合
print(db.collection_names(include_system_collections=False))

#4、创建集合
table_user=db['userinfo'] #等同于：db.user

#5、插入文档(多条)
  import datetime
  user0={
      "_id":1,
      "name":"egon",
      "birth":datetime.datetime.now(),
      "age":10,
      'hobbies':['music','read','dancing'],
      'addr':{
          'country':'China',
          'city':'BJ'
      }
  }

  user1={
      "_id":2,
      "name":"alex",
      "birth":datetime.datetime.now(),
      "age":10,
      'hobbies':['music','read','dancing'],
      'addr':{
          'country':'China',
          'city':'weifang'
      }
  }
  res=table_user.insert_many([user0,user1]).inserted_ids
  print(res)
  print(table_user.count())

#5、插入文档(单条)
	user_info = request.form.to_dict()
  user_info["avater"] = "mama.jpg" if user_info.get("gener") == 1 else "baba.jpg"
  user_info["friend_list"] = []
  user_info["bind_toy"] = []
  res = MONGO_DB.users.insert_one(user_info)
  Response["code"] = 0
  Response["msg"] = "用户注册成功"
  Response["data"] = {"user_id":str(res.inserted_id)} # 拿到插入成功的那条记录的ID


#6、查找

# from pprint import pprint#格式化细
# pprint(table_user.find_one())
# for item in table_user.find():
#     pprint(item)

# print(table_user.find_one({"_id":{"$gte":1},"name":'egon'}))

#7、更新
table_user.update({'_id':1},{'name':'EGON'})

#8、传入新的文档替换旧的文档
table_user.save(
    {
        "_id":2,
        "name":'egon_xxx'
    }
)