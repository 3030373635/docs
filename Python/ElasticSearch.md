## 1.1 安装

```python
pip install elasticsearch
# 注意elasticsearch模块版本尽量和elasticsearch对应，不然可能会报错，可能....
```

## 1.2 连接

```python
from elasticsearch import  Elasticsearch
# es = Elasticsearch()    # 默认连接本地elasticsearch
# es = Elasticsearch(['127.0.0.1:9200'])  # 连接本地9200端口
es = Elasticsearch(
    ["192.168.1.10", "192.168.1.11", "192.168.1.12"], # 连接集群，以列表的形式存放各节点的IP地址
    sniff_on_start=True,    # 连接前测试
    sniff_on_connection_fail=True,  # 节点无响应时刷新节点
    sniff_timeout=60    # 设置超时时间
)
```

## 1.3 索引操作

```python
- es.indices.delete_alias，删除特定别名。
- es.indices.exists，返回一个布尔值，指示给定的索引是否存在。
- es.indices.exists_type，检查索引/索引中是否存在类型/类型。
- es.indices.flus，明确的刷新一个或多个索引。
- es.indices.get_field_mapping，检索特定字段的映射。
- es.indices.get_template，按名称检索索引模板。
- es.indices.open，打开一个封闭的索引以使其可用于搜索。
- es.indices.close，关闭索引以从群集中删除它的开销。封闭索引被阻止进行读/写操作。
- es.indices.clear_cache，清除与一个或多个索引关联的所有缓存或特定缓存。
- es.indices.put_alias，为特定索引/索引创建别名。
- es.indices.get_uprade，监控一个或多个索引的升级程度。
- es.indices.put_mapping，注册特定类型的特定映射定义。
- es.indices.put_settings，实时更改特定索引级别设置。
- es.indices.put_template，创建一个索引模板，该模板将自动应用于创建的新索引。
- es.indices.rollove，当现有索引被认为太大或太旧时，翻转索引API将别名转移到新索引。API接受单个别名和条件列表。别名必须仅指向单个索引。如果索引满足指定条件，则创建新索引并切换别名以指向新别名。
- es.indices.segments，提供构建Lucene索引（分片级别）的低级别段信息。
```

### 创建索引

**es.index**

```python
# 向指定索引添加或更新文档，如果索引不存在，首先会创建该索引，然后再执行添加或者更新操作
print(es.index(index='w2', doc_type='doc', id='4', body={"name":"可可", "age": 18}))    # 正常
print(es.index(index='w2', doc_type='doc', id=5, body={"name":"卡卡西", "age":22}))     # 正常
print(es.index(index='w2', id=6, body={"name": "鸣人", "age": 22}))  # 会报错，TypeError: index() missing 1 required positional argument: 'doc_type'
print(es.index(index='w2', doc_type='_doc', body={"name": "鸣人", "age": 22}))  # 可以不指定id，默认生成一个id
```

**es.create**

```python
# 创建索引（索引不存在的话）并新增一条数据，索引存在仅新增（只能新增，重复执行会报错）,本质还是调用了es.index
print(es.create(index='py3', doc_type='doc', id='1', body={"name": '王五', "age": 20}))
print(es.get(index='py3', doc_type='doc', id='3'))
```

**es.indices.create**

```python
# 在Elasticsearch中创建索引，用的最多。也是常用的创建自定义索引的方式。
body = {
    "mappings": {
            "dynamic": "strict",
            "properties": {
                "title": {
                    "type": "text",
                    "analyzer": "ik_max_word"
                },
                "url": {
                    "type": "text"
                },
                "action_type": {
                    "type": "text"
                },
                "content": {
                    "type": "text"
                }
            }
    }
}
es.indices.create('py4', body=body)
```

### 删除索引

```python
print(es.indices.delete(index='py4'))
print(es.indices.delete(index='w3'))    # {'acknowledged': True}
```

### 查询索引定义

```python
# 检索一个或多个（或所有）索引的设置
print(es.indices.get_settings(index='py4'))
```

### 查询索引是否存在

```python
print(es.indices.get(index='py2'))    # 查询指定索引是否存在
print(es.indices.get(index=['py2', 'py3']))
```

### 索引别名

**es.indices.put_alias**

```python
"""
为一个或多个索引创建别名，查询多个索引的时候，可以使用这个别名。
put_alias参数说明：
-  index 别名应指向的逗号分隔的索引名称列表（支持通配符），使用_all对所有索引执行操作。
-  name 要创建或更新的别名的名称。
-  body 别名的设置，例如路由或过滤器。
"""
print(es.indices.put_alias(index='py4', name='py4_alias'))  # 为单个索引创建别名
print(es.indices.put_alias(index=['py3', 'py2'], name='py23_alias'))  # 为多个索引创建同一个别名，联查用
```

**es.indices.delete_alias**

```python
# 删除一个或多个别名
print(es.indices.delete_alias(index='alias1'))
print(es.indices.delete_alias(index=['alias1, alias2']))
```

**es.indices.get_alias**

```python
# 检索别名是否存在
print(es.indices.get_alias(index='py2'))
print(es.indices.get_alias(index=['py2', 'py3']))
```

### 查询分词

```python
es.indices.analyze(body={'analyzer': "ik_max_word", "text": "皮特和茱丽当选“年度模范情侣”Brad Pitt and Angelina Jolie"})
```

### 查询映射的定义

```python
print(es.indices.get_mapping(index='py4'))
```

### 检索字段映射信息

```python
print(es.indices.get_field_mapping(fields='url', index='py4', doc_type='doc'))
print(es.indices.get_field_mapping(fields=['url', 'title'], index='py4', doc_type='doc'))
```

## 1.4 文档操作

### 插入文档

```python
data = {
     "id":10000,
     "is_top":1,
     "comment_number":20,
     "read_number":20,
     "title":"test",
     "content":"这里是正文",
     "img":"http://www.lqmblog.com",
     "created_time":"2010-10-10"
 }
res = es.create(index='blog', doc_type='_doc', id=1000, body=data)# 这里的id其实是es里面的_id
print(res)
```

### 删除文档

**方式一：es.delete**

```python
res = es.delete(index='py3', doc_type='doc', id='4')# 这里的id其实是es里面的_id
print(res) 
```

**方式二：es.delete_by_query**

```python
# 删除与查询匹配的所有文档。
"""
delete_by_query参数说明
- index 要搜索的以逗号分隔的索引名称列表; 使用_all 或空字符串对所有索引执行操作。
- doc_type 要搜索的以逗号分隔的文档类型列表; 留空以对所有类型执行操作。
- body 使用Query DSL的搜索定义。
"""

print(es.delete_by_query(index='py3', doc_type='doc', body={"query": {"match":{"age": 20}}}))
```

### 修改文档

```python
body = {'doc': {'title': 'blog项目'}}# 删除title为blog项目的文档
res = es.update(index='blog', body=body,id="...",doc_type="_doc") # 这里的id其实是es里面的_id
print(res)
```

### 搜索文档

```python
# 执行搜索查询并获取与查询匹配的搜索匹配。这个用的最多，可以跟复杂的查询条件。
"""
search参数说明：
- index要搜索的以逗号分隔的索引名称列表; 使用_all 或空字符串对所有索引执行操作。
- doc_type 要搜索的以逗号分隔的文档类型列表; 留空以对所有类型执行操作。
- body使用Query DSL（QueryDomain Specific Language查询表达式）的搜索定义。
- _source 返回_source字段的true或false，或返回的字段列表，返回指定字段。
- _source_exclude要从返回的_source字段中排除的字段列表，返回的所有字段中，排除哪些字段。
- _source_include从_source字段中提取和返回的字段列表，跟_source差不多。
"""
print(es.search(index='py3', doc_type='_doc', body={"query": {"match":{"age": 20}}}))  # 一般查询
print(es.search(index='py3', doc_type='_doc', body={"query": {"match":{"age": 19}}},_source=['name', 'age']))  # 结果字段过滤
print(es.search(index='py3', doc_type='_doc', body={"query": {"match":{"age": 19}}},_source_exclude  =[ 'age']))
print(es.search(index='py3', doc_type='_doc', body={"query": {"match":{"age": 19}}},_source_include =[ 'age']))
```

### 字段过滤

```python
print(es.search(index='py2', filter_path=['hits.total', 'hits.hits._source']))    # 可以省略type类型
print(es.search(index='w2', doc_type='_doc'))        # 可以指定type类型
print(es.search(index='w2', doc_type='_doc', filter_path=['hits.total']))
```

> `filter_path`参数用于减少elasticsearch返回的响应，比如仅返回`hits.total`和`hits.hits._source`内容。
> 除此之外，`filter_path`参数还支持`*`通配符以匹配字段名称、任何字段或者字段部分：

```python
print(es.search(index='py2', filter_path=['hits.*']))
print(es.search(index='py2', filter_path=['hits.hits._*']))
print(es.search(index='py2', filter_path=['hits.to*']))  # 仅返回响应数据的total
print(es.search(index='w2', doc_type='_doc', filter_path=['hits.hits._*']))        # 可以加上可选的type类型
```

### 查询指定文档

**es.get**

```python
# 查询索引中指定文档
print(es.get(index='w2', doc_type='doc', id=5))  # 正常
print(es.get(index='w2', doc_type='doc'))  # TypeError: get() missing 1 required positional argument: 'id'
print(es.get(index='w2',  id=5))  # TypeError: get() missing 1 required positional argument: 'doc_type'
```

**es.get_source**

```python
# 通过索引、类型和ID获取文档的来源，其实，直接返回想要的字典
print(es.get_source(index='py3', doc_type='doc', id='1'))  # {'name': '王五', 'age': 19}
```

### 统计文档数量

```python
# 执行查询并获取该查询的匹配数。比如查询年龄是18的文档
body = {
    "query": {
        "match": {
            "age": 18
        }
    }
}
print(es.count(index='py2', doc_type='doc', body=body))  # {'count': 1, '_shards': {'total': 5, 'successful': 5, 'skipped': 0, 'failed': 0}}
print(es.count(index='py2', doc_type='doc', body=body)['count'])  # 1
print(es.count(index='w2'))  # {'count': 6, '_shards': {'total': 5, 'successful': 5, 'skipped': 0, 'failed': 0}}
print(es.count(index='w2', doc_type='doc'))  # {'count': 6, '_shards': {'total': 5, 'successful': 5, 'skipped': 0, 'failed': 0}}
```

### 查询文档是否存在

```python
# 查询elasticsearch中是否存在指定的文档，返回一个布尔值。
print(es.exists(index='py3', doc_type='doc', id='1'))
```

## 1.5 集群

- es.cluster.get_settigns，获取集群设置。

```
print(es.cluster.get_settings())
```

- es.cluster.health，获取有关群集运行状况的非常简单的状态。

```
print(es.cluster.health())
```

- es.cluster.state，获取整个集群的综合状态信息。

```
print(es.cluster.state())
```

- es.cluster.stats，返回群集的当前节点的信息。

```
print(es.cluster.stats())
```

## 1.6 节点

- es.nodes.info，返回集群中节点的信息。

```
print(es.nodes.info())  # 返回所节点
print(es.nodes.info(node_id='node1'))   # 指定一个节点
print(es.nodes.info(node_id=['node1', 'node2']))   # 指定多个节点列表
```

- es.nodes.stats，获取集群中节点统计信息。

```
print(es.nodes.stats())
print(es.nodes.stats(node_id='node1'))
print(es.nodes.stats(node_id=['node1', 'node2']))
```

- es.nodes.hot_threads，获取指定节点的线程信息。

```
print(es.nodes.hot_threads(node_id='node1'))
print(es.nodes.hot_threads(node_id=['node1', 'node2']))
```

- es.nodes.usage，获取集群中节点的功能使用信息。

```
print(es.nodes.usage())
print(es.nodes.usage(node_id='node1'))
print(es.nodes.usage(node_id=['node1', 'node2']))
```

## 1.7 Cat（一种查询方式）

- es.cat.aliases，返回别名信息。
  - `name`要返回的以逗号分隔的别名列表。
  - `format`Accept标头的简短版本，例如json，yaml

```
print(es.cat.aliases(name='py23_alias'))
print(es.cat.aliases(name='py23_alias', format='json'))
```

- es.cat.allocation，返回分片使用情况。

```
print(es.cat.allocation())
print(es.cat.allocation(node_id=['node1']))
print(es.cat.allocation(node_id=['node1', 'node2'], format='json'))
```

- es.cat.count，Count提供对整个群集或单个索引的文档计数的快速访问。

```
print(es.cat.count())  # 集群内的文档总数
print(es.cat.count(index='py3'))  # 指定索引文档总数
print(es.cat.count(index=['py3', 'py2'], format='json'))  # 返回两个索引文档和
```

- es.cat.fielddata，基于每个节点显示有关当前加载的fielddata的信息。有些数据为了查询效率，会放在内存中，fielddata用来控制哪些数据应该被放在内存中，而这个`es.cat.fielddata`则查询现在哪些数据在内存中，数据大小等信息。

```
print(es.cat.fielddata())
print(es.cat.fielddata(format='json', bytes='b'))
```

`bytes`显示字节值的单位，有效选项为：`'b'，'k'，'kb'，'m'，'mb'，'g'，'gb'，'t'，'tb' ，'p'，'pb'`
`format`Accept标头的简短版本，例如json，yaml

- es.cat.health，从集群中`health`里面过滤出简洁的集群健康信息。

```
print(es.cat.health())
print(es.cat.health(format='json'))
```

- es.cat.help，返回`es.cat`的帮助信息。

```
print(es.cat.help())
```

- es.cat.indices，返回索引的信息；也可以使用此命令进行查询集群中有多少索引。

```
print(es.cat.indices())
print(es.cat.indices(index='py3'))
print(es.cat.indices(index='py3', format='json'))
print(len(es.cat.indices(format='json')))  # 查询集群中有多少索引
```

- es.cat.master，返回集群中主节点的IP，绑定IP和节点名称。

```
print(es.cat.master())
print(es.cat.master(format='json'))
```

- es.cat.nodeattrs，返回节点的自定义属性。

```
print(es.cat.nodeattrs())
print(es.cat.nodeattrs(format='json'))
```

- es.cat.nodes，返回节点的拓扑，这些信息在查看整个集群时通常很有用，特别是大型集群。我有多少符合条件的节点？

```
print(es.cat.nodes())
print(es.cat.nodes(format='json'))
```

- es.cat.plugins，返回节点的插件信息。

```
print(es.cat.plugins())
print(es.cat.plugins(format='json'))
```

- es.cat.segments，返回每个索引的Lucene有关的信息。

```
print(es.cat.segments())
print(es.cat.segments(index='py3'))
print(es.cat.segments(index='py3', format='json'))
```

- es.cat.shards，返回哪个节点包含哪些分片的信息。

```
print(es.cat.shards())
print(es.cat.shards(index='py3'))
print(es.cat.shards(index='py3', format='json'))
```

- es.cat.thread_pool，获取有关线程池的信息。

```
print(es.cat.thread_pool())
```

## 1.8 Snapshot（快照相关）

- es.snapshot.create，在存储库中创建快照。
  - `repository` 存储库名称。
  - `snapshot`快照名称。
  - `body`快照定义。
- es.snapshot.delete，从存储库中删除快照。
- es.snapshot.create_repository。注册共享文件系统存储库。
- es.snapshot.delete_repository，删除共享文件系统存储库。
- es.snapshot.get，检索有关快照的信息。
- es.snapshot.get_repository，返回有关已注册存储库的信息。
- es.snapshot.restore，恢复快照。
- es.snapshot.status，返回有关所有当前运行快照的信息。通过指定存储库名称，可以将结果限制为特定存储库。
- es.snapshot.verify_repository，返回成功验证存储库的节点列表，如果验证过程失败，则返回错误消息。

## 1.9 Task（任务相关）

- es.tasks.get，检索特定任务的信息。
- es.tasks.cancel，取消任务。
- es.tasks.list，任务列表。