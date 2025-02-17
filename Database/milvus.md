#### 数据模型基础概念

- **Collection**: 类似数据库的表，包含多个字段
- **Schema**: 定义字段类型和约束
- **Partition**: 数据分区，用于优化查询性能
- **Index**: 加速向量搜索的索引结构

#### 创建集合（Collection）

```python
from pymilvus import (
    FieldSchema, CollectionSchema, DataType,
    Collection
)

# 定义字段
fields = [
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True, auto_id=True),
    FieldSchema(name="embedding", dtype=DataType.FLOAT_VECTOR, dim=128),
    FieldSchema(name="age", dtype=DataType.INT32)
]

# 创建Schema
schema = CollectionSchema(fields, description="人脸特征向量库")

# 创建Collection
collection = Collection(name="face_db", schema=schema)
"""
参数说明：
    auto_id: 是否自动生成主键
    dim: 向量维度（必须与后续插入数据维度一致）
"""
```