---
title: pip
sidebar_position: 1
---


# 一、国内镜像

```python
新版ubuntu要求使用https源，要注意。

清华        https://pypi.tuna.tsinghua.edu.cn/simple

阿里云      https://mirrors.aliyun.com/pypi/simple/

中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/

华中理工大学 https://pypi.hustunique.com/

山东理工大学 https://pypi.sdutlinux.org/

豆瓣        https://pypi.douban.com/simple/ 
```

# 二、临时使用

```python
可以在使用pip的时候加参数-i    https://pypi.tuna.tsinghua.edu.cn/simple

例如：pip install -i https://pypi.tuna.tsinghua.edu.cn/simple/ requests
例如：pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

这样就会从清华源去安装requests库
```

# 三、永久使用

**配置文件内容**

```python
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple/
[install]
use-mirrors =true
mirrors =https://pypi.tuna.tsinghua.edu.cn/simple/
trusted-host =pypi.tuna.tsinghua.edu.cn
```

## 3.1 windows

```python
1. 文件管理器文件路径地址栏敲：%APPDATA% 回车，快速进入 C:\Users\电脑用户\AppData\Roaming 文件夹中
2. 新建 pip 文件夹并在文件夹中新建 pip.ini 配置文件
3. 新增 pip.ini 配置文件内容
```

## 3.2 MacOs、Linux

```python
1. 在用户根目录下 ~ 下创建 .pip 隐藏文件夹，如果已经有了可以跳过
    -- mkdir ~/.pip
2. 进入 .pip 隐藏文件夹并创建 pip.conf 配置文件
    -- cd ~/.pip && touch pip.conf
3. 启动 Finder(访达) 按 cmd+shift+g 来的进入，输入 ~/.pip 回车进入
4. 新增 pip.conf 配置文件内容
```

# 四、打包项目依赖包

```python
(env1) [root@master myenv]# pip3 freeze
backcall==0.1.0
Click==7.0
decorator==4.4.0
Django==1.11.1
Flask==1.1.1
ipython==7.7.0
ipython-genutils==0.2.0
itsdangerous==1.1.0
jedi==0.15.1
Jinja2==2.10.1
MarkupSafe==1.1.1
parso==0.5.1
pexpect==4.7.0
pickleshare==0.7.5
prompt-toolkit==2.0.9
ptyprocess==0.6.0
Pygments==2.4.2
pytz==2019.2
six==1.12.0
traitlets==4.3.2
wcwidth==0.1.7
Werkzeug==0.15.5
```

```python
我们可以将pip3 freeze的结果重定向到一个txt文件。我们约定俗成为requirements.txt，文件里面是项目依赖包以及版本

当我们在另外一个项目中执行下面语句时，就会自动安装文件里面的库
pip3 install -r requirements.txt
```

# 五、制作包

> **本教程带你制作专属自己的开源模块，以后所有人都可方便的使用pip安装你的模块了，如：pip install 模块名字**

![image](assets/images/1665122610.png)

```
对于模块开发者本质上需要做3件事：

- 编写模块
- 将模块进行打包
- 上传到PyPI（需要先注册PyPI账号）
  - 注册PyPI账号
  - 安装上传工具
  - 基于工具进行上传

对于模块的使用者来说，只需要做2件事：

- 通过pip install 模块名字 去安装模块
- 调用模块

假设，现在我们要做一个名称叫 fucker 的模块，跟着下面的步骤一步一步的操作。
```

## 5.1 项目结构

```
fucker
├── LICENSE           # 声明，给模块使用者看，说白了就是使用者是否可以免费用于商业用途等。
├── README.md         # 模块介绍
├── demos             # 使用案例
├── fucker            # 模块代码目录
│   └── __init__.py
└── setup.py          # 给setuptools提供信息的脚本(名称、版本等)
```

### 5.1.1 License

> LICENSE文件就是咱们模块的许可证，给模块使用者看，说白了就是使用者是否可以免费用于商业用途等。一般开源的软件会选择相对宽泛许可证 MIT，即：作者保留版权，无其他任何限制。
>
> 更多其他许可证参见：[https://choosealicense.com/](https://choosealicense.com/)

```
Copyright (c) 2018 The Python Packaging Authority

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### 5.1.2 readme

> readme就是对于当前模块的描述信息，一般用于markdown格式编写
>
> 此文件也可以写模块的简单使用手册，如果手册太复杂建议再创建一个doc文件夹存放使用手册。

```
fucker是一个吊炸天的工具...
```

### 5.1.3 demos目录

> demos一般会写一些该模块使用的示例，用于使用者快速可以将模块应用到生成中。

### 5.1.4 setup.py

> setup.py文件其实是一个配置文件，用于给setuptools提供一些模块相关的信息，如：模块名称、模块版本、适用的python版本、作者、github地址等。
>
> **注意：setuptools是一个包管理工具，可用于打包和安装模块。**

```python
import setuptools

with open("README.md", "r") as fh:
    long_description = fh.read()

setuptools.setup(
    name="fucker",  # 模块名称
    version="1.0",  # 当前版本
    author="wupeiqi",  # 作者
    author_email="wupeiqi@live.com",  # 作者邮箱
    description="一个非常NB的包",  # 模块简介
    long_description=long_description,  # 模块详细介绍
    long_description_content_type="text/markdown",  # 模块详细介绍格式
    # url="https://github.com/wupeiqi/fucker",  # 模块github地址
    packages=setuptools.find_packages(),  # 自动找到项目中导入的模块
    # 模块相关的元数据
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    # 依赖模块
    install_requires=[
        'pillow',
    ],
    python_requires='>=3',
)
```

### 5.1.5 fucker目录

> 插件内容，可以将相关代码在此处编写，如：wp.py

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-


def func():
    print("一个神奇的包")

```

**最后的文件夹如下：**

```
fucker
├── LICENSE
├── README.md
├── demos
├── fucker
│   ├── __init__.py
│   └── wp.py
└── setup.py
```

## 5.2 代码打包上传

> 上述步骤中将文件夹和代码编写好之后，就需要对代码进行打包处理。

### 5.2.1 安装打包工具

> 打包代码需先安装setuptools和wheel两个工具，可以单独安装，也可以安装pip，从而自动安装这两个工具。
>
> - Securely Download [get-pip.py](https://bootstrap.pypa.io/get-pip.py) [[1\]](https://packaging.python.org/tutorials/installing-packages/#id7)
> - Run `python get-pip.py`. [[2\]](https://packaging.python.org/tutorials/installing-packages/#id8) This will install or upgrade pip. Additionally, it will install [setuptools](https://packaging.python.org/key_projects/#setuptools) and [wheel](https://packaging.python.org/key_projects/#wheel) if they’re not installed already.
>   详见：[https://packaging.python.org/tutorials/installing-packages/](https://packaging.python.org/tutorials/installing-packages/)
> - 注意：已安装用户，如要更新setuptools和wheel两个工具，可通过如下命令：

```
python -m pip install --upgrade setuptools wheel
```

### 5.2.2 打包代码

```
python setup.py sdist bdist_wheel
```

```
fucker
├── LICENSE
├── README.md
├── fucker
│   ├── __init__.py
│   └── wp.py
├── fucker.egg-info
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   ├── dependency_links.txt
│   └── top_level.txt
├── build
│   ├── bdist.macosx-10.6-intel
│   └── lib
│       └── fucker
│           ├── __init__.py
│           └── wp.py
├── demos
├── dist
│   ├── fucker-0.0.1-py3-none-any.whl
│   └── fucker-0.0.1.tar.gz
└── setup.py
```

### 5.2.3 发布模块（上传）

> 文件打包完毕后，需要将打包之后的文件上传到PyPI，如果想要上传是需要先去 [https://pypi.org/](https://pypi.org/) 注册一个账号。

**安装用于发布模块的工具：twine**

```
python -m pip install --upgrade twine
或
pip install --upgrade twine

# 提示：python -m 的作用是 run library module as a script (terminates option list)
```

**发布（上传）**

```
python -m twine upload --repository-url https://upload.pypi.org/legacy/  dist/*
或
twine upload --repository-url https://upload.pypi.org/legacy/  dist/*
```

**上传时，提示需要输入PyPI的用户名和密码.**

## 5.3 安装使用

### 5.3.1 安装

```
pip install 模块
```

### 5.3.2 应用

```。python
from fucker import wp

wp.func()
```

# 六、离线安装包

1. 将环境列出到requirements.txt

   ```shell
   lqm@lqmdeMacBook-Pro ~ % pip freeze > requirements.txt
   ```

2. 在有网的环境下下载`requirements.txt`的包到`D:\Users\packagesdir`

   ```shell
   lqm@lqmdeMacBook-Pro ~ % pip download -d D:\Users\packagesdir  -r requirements.txt
   ```

3. 安装离线包

   ```shell
   lqm@lqmdeMacBook-Pro ~ % pip install --no-index --find-links=D:\Users\packagesdir -r requirements.txt
   ```

   