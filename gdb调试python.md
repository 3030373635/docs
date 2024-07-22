- gcc (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0
- Ubuntu 20.04 LTS
- python 3.8.1
- GNU Make 4.2.1
- GNU gdb (Ubuntu 9.2-0ubuntu1~20.04) 9.2

## 一种方法

查找*gdb文件

```shell
lqm@ubuntu:~/Desktop$ find / -name "*gdb.py"
```



```
udo gdb /usr/bin/python3 -p 7516 -x /usr/share/gdb/auto-load/usr/bin/python3.8-gdb.py
```



如果报错

> (gdb) py-list
> Unable to open fib.py: [Errno 2] No such file or directory: b'fib.py'

在fib.py文件所在路径执行gdb

## 另外一种方法

1. 安装gdb

2. 下载python解释器对应的[libpython](https://github.com/python/cpython/blob/3.9/Tools/gdb/libpython.py)文件，貌似包含针对py调试的插件，可以使用py-bt、py-list等

3. 这里使用python3.10运行的程序，调试也使用python3.10

   ```shell
   lqm@ubuntu:~/Desktop$ gdb /data/software/miniconda3/envs/baizhanai/bin/python3.10 进程id
   ```

4. 加载libpython

   ```shell
   (gdb) python
   >import sys
   >sys.path.insert(0,libpython.py所在目录)
   >import libpython
   >end
   ```

5. 测试是否成功

   ```shell
   gdb) py-bt
   Traceback (most recent call first):
     (unable to read python frame information)
     (unable to read python frame information)
     (unable to read python frame information)
   
   # 如果报错了，应该是python解释器版本与libpython版本不一样。
   ```

   



