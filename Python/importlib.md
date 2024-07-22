## 1.1 importlib

> importlib通过字符串路径找到对应对象，并且只能到模块

**/aaa/bbb.py**

```python
class ccc():
    def ddd():
        print("这是ddd方法")
```

**/test.py**

```python
import importlib

path = "aaa.bbb.ccc"

module_path,class_name = path.rsplit('.',maxsplit=1)

m = importlib.import_module(module_path)


cls = getattr(m,class_name)


obj = cls()
obj.test()  # 这是ddd方法