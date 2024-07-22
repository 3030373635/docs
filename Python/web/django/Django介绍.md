---
title: django介绍
---


# 一、MVC与MTV模型

## 1.1 MVC

> Web服务器开发领域里著名的MVC模式，所谓MVC就是把Web应用分为模型(M)，控制器(C)和视图(V)三层，他们之间以一种插件式的、松耦合的方式连接在一起，模型负责业务对象与数据库的映射(ORM)，视图负责与用户的交互(页面)，控制器接受用户的输入调用模型和视图完成用户的请求，其示意图如下所示：

<img src="https://cos.liuqm.cc/download.png">

## 1.2 MTV

> Django的MTV模式本质上和MVC是一样的，也是为了各组件间保持松耦合关系，只是定义上有些许不同，Django的MTV分别是值：
>
> - M 代表模型（Model）： 负责业务对象和数据库的关系映射(ORM)。
> - T 代表模板 (Template)：负责如何把页面展示给用户(html)。
> - V 代表视图（View）：  负责业务逻辑，并在适当时候调用Model和Template。
>
> 除了以上三层之外，还需要一个URL分发器，它的作用是将一个个URL的页面请求分发给不同的View处理，View再调用相应的Model和Template，MTV的响应模式如下所示：

<img src="https://cos.liuqm.cc/download-1.png">

# 二、django下载与基本使用

**安装**

```
pip install django
```

**创建一个django项目**

```
django-admin.py startproject mysite(mysite是我的项目名)
```

**目录结构**

> manage.py ----- Django项目里面的工具，通过它可以调用django shell和数据库等。
>
> settings.py ---- 包含了项目的默认设置，包括数据库信息，调试标志以及其他一些工作的变量。
>
> urls.py ----- 负责把URL模式映射到应用程序。

<img src="https://cos.liuqm.cc/download-2.png">

**在项目根目录下创建app应用**

```python
python manage.py startapp blog
```

**启动django项目**

```
python manage.py runserver 8000
```

# 三、django请求生命周期

<img src="https://cos.liuqm.cc/download-3.png">