---
title: django配置文件
---

# 一、静态资源相关

## 1.1 static

### STATIC_ROOT

> **这个常量在开发模式中不会用到，在部署的时候才会用到。**
>
> 部署的时候执行python manage.py collectstatic，django会把所有App下的static文件都复制到STATIC_ROOT文件夹下

```python

# BASE_DIR 一般是项目地址
STATIC_ROOT = os.path.join(BASE_DIR, 'xxxxxx')
```

### STATICFILES_DIRS

> **开发模式中需要用到**。静态文件一般放在两个地方：
>
> - 每个App下面的static目录
> - 项目根目录下的static目录（因为有些静态文件不是某个app独有的）

```python
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'common_static'),
)
"""
STATICFILES_DIRS告诉django,首先到STATICFILES_DIRS里面寻找静态文件，其次再到各个注册的app的static文件夹里面找

注意：django查找静态文件是惰性查找,查找到第一个,就停止查找了
"""
```

### STATIC_URL

```python
STATIC_URL = '/st/' # 一般为static，这里我为了弄清机制
"""
django利用STATIC_URL来让浏览器可以直接访问静态文件
这样假如你项目根目录/static/下有一个123.png的图片
那么就可以直接通过浏览器http://IP：端口号/st/123.png来访问你的图片了
可以看出我们用st代替了static，为了安全起见，django给static文件夹取了别名
"""

```

## 1.2 template

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')]
        ,
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
# 其中DIRS为django默认先去找template的路径，如果找不到，其次再到各个注册的app中找template
```

# 二、ALLOWED_HOSTS

```
主要是为了安全，只允许你设置的域名(hosts)访问，如果不加， 或者设置为"*"，那么有可能别人随便把一个域名的解析指向你的服务器， 都能访问
```

# 三、APPEND_SLASH

> APPEND_SLASH默认为True，如果访问过来的url路径后面没有'/'，就会自动加'/'