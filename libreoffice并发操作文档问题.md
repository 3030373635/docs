## 问题背景

使用libreoffice(version==5.3.6)在[linux](https://so.csdn.net/so/search?q=linux&spm=1001.2101.3001.7020)端做word文档转pdf，在使用多线程进行word转pdf时，出现程序卡死、超时等问题，并且程序也不会抛出异常

```python
# word 转 pdf
cmd = f"libreoffice --headless --convert-to pdf {doc_path} --outdir {pdf_path}"
 
# 运行命令
subprocess.run(cmd, shell=True, timeout=15)
```

## 问题分析

libreoffice在运行过程中，会使用UserInstallation目录共享相同的空间，因此[多线程](https://so.csdn.net/so/search?q=多线程&spm=1001.2101.3001.7020)操作时会出现问题。

## 解决方案

在转化过程中，为每个[word文档](https://so.csdn.net/so/search?q=word文档&spm=1001.2101.3001.7020)定义一个UserInstallation工作空间

```python
cmd = f"libreoffice -env:UserInstallation=file:{pdf_path} --headless --convert-to pdf {doc_path} --outdir {pdf_path}"
```

## 其他问题

1. 命令 libreoffice --headless --convert-to pdf *.doc 也可以批量转pdf，不过是串行操作
2. 将libreoffice版本从5.3升级到7.5之后，不用单独为每个文档定义工作空间也可以做转化，但是会出现个别文档没有转化的情况

## 测试脚本

```python
from threading import Thread
import subprocess
import time
thread_num = 5
 
 
def doc_to_pdf(doc_path, pdf_path, i):
    # cmd = f"libreoffice --headless --convert-to pdf {doc_path} --outdir {pdf_path}"
    cmd = f"libreoffice -env:UserInstallation=file:{pdf_path} --headless --convert-to pdf {doc_path} --outdir {pdf_path}"
    try:
        subprocess.run(cmd, shell=True, timeout=15)
    except Exception as e:
        print(f"error:{e}")
        raise
 
 
if __name__ == '__main__':
    thread_list = []
    for i in range(thread_num):
        thread = Thread(target=doc_to_pdf, args=(f'/data/test_files/{i}.doc', f'/data/test_files/test_pdf/{i}', i))
        thread_list.append(thread)
 
    for th in thread_list:
        th.start()
 
    for th in thread_list:
        th.join()
```


