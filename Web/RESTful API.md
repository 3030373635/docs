## 一、RESTful介绍

> REST与技术无关，代表的是一种软件架构风格，REST是Representational State Transfer的简称，中文翻译为“表征状态转移”或“表现层状态转化”。
>
> 推荐阅读 [阮一峰 理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)

## 二、RESTful API规范设计

### 2.1 API与用户的通信协议

> 总是使用[HTTPS协议](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)。

### 2.2 API体现

```python
https://api.example.com  # 建议使用，不过可能会出现跨域

https://example.org/api/ # 简单，不出现跨域，不推荐使用
```

### 2.3 版本体现

```python
1. 将版本信息放在URL中，如：https://api.example.com/v1/ # 建议使用

2. 将版本信息放在请求头中。
```

### 2.4 路径

> 视网络上任何东西都是资源，均使用名词表示（建议使用复数）

```python
https://api.example.com/v1/zoos

https://api.example.com/v1/animals

https://api.example.com/v1/employees
```

### 2.5 method

```python
GET     ：获取一项或者多项资源

POST    ：新建资源

PUT     ：更新资源(完整更新)

PATCH  	：更新资源(不完整更新)

DELETE 	：删除资源
```

### 2.6 过滤

> 通过在url上传参的形式传递搜索条件

```python
https://api.example.com/v1/zoos?limit=10	# 指定返回记录的数量

https://api.example.com/v1/zoos?offset=10	# 指定返回记录的开始位置

https://api.example.com/v1/zoos?page=2&per_page=100 # 指定第几页，以及每页的记录数

https://api.example.com/v1/zoos?sortby=name&order=asc # 指定返回结果按照哪个属性排序，以及排序顺序

https://api.example.com/v1/zoos?animal_type_id=1 # 指定筛选条件
```

### 2.7 状态码

> 我一般没用状态码，而是在返回信息中使用code标识，例如
>
> {code:200,msg:"",data:[]}

```python
200 OK - [GET] #服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
201 CREATED - [POST/PUT/PATCH] # 用户新建或修改数据成功。
202 Accepted - [*] # 表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE] # 用户删除数据成功。
400 INVALID REQUEST - [POST/PUT/PATCH] # 用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
401 Unauthorized - [*] # 表示用户没有权限（令牌、用户名、密码错误）。
403 Forbidden - [*] # 表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 NOT FOUND - [*] # 用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
406 Not Acceptable - [GET] # 用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
410 Gone -[GET] # 用户请求的资源被永久删除，且不会再得到的。
422 Unprocesable entity - [POST/PUT/PATCH] # 当创建一个对象时，发生一个验证错误。
500 INTERNAL SERVER ERROR - [*] # 服务器发生错误，用户将无法判断发出的请求是否成功。
```

### 2.8 错误处理

> 状态码是4xx时，应返回错误信息，error当做key。

```
{
	code:xxxx
	error: "Invalid API key"
}
```

### 2.9 返回结果

> 针对不同操作，服务器向用户返回的结果应该符合以下规范

```python
GET /books 			# 返回book的所有信息列表（数组）
GET /books/1  	# 返回单个资源对象
POST /books   	# 返回新生成的资源对象
PUT /books/1    # 返回完整的资源对象
PATCH /books/1  # 返回完整的资源对象
DELETE /books/1 # 返回一个空文档
```

### 3.0 Hypermedia API

> RESTful API最好做到Hypermedia，即返回结果中提供链接，连向其他API方法，使得用户不查文档，也知道下一步应该做什么。

```
{"link": {
  "rel":   "collection https://www.example.com/zoos",
  "href":  "https://api.example.com/zoos",
  "title": "List of zoos",
  "type":  "application/vnd.yourformat+json"
}}
```

# 三、设计示例

## 3.1 接口设计规范

```python
GET    127.0.0.1:8000/books/        # 获取所有数据
GET    127.0.0.1:8000/books/{id}/   # 获取单条数据
POST   127.0.0.1:8000/books/        # 添加一条数据
DELETE 127.0.0.1:8000/books/{id}/   # 删除一条数据
PUT    127.0.0.1:8000/books/{id}/   # 修改一条数据
```

## 3.2 返回数据规范

```python
GET    127.0.0.1:8000/books/        # 获取所有数据 [{}, {}, {}, {}, {}]
GET    127.0.0.1:8000/books/{id}/   # 获取单条数据 {}为单条数据
POST   127.0.0.1:8000/books/        # 添加一条数据 {}为新增的数据
DELETE 127.0.0.1:8000/books/{id}/   # 删除一条数据 返回空
PUT    127.0.0.1:8000/books/{id}/   # 修改一条数据 {}为修改后的数据