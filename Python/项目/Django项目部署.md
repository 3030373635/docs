---
title: django项目部署
---


# 一、图解

> 图上的配置为最基本的搭建，没有处理静态资源，也没有优化，
> 下面会说怎么处理静态资源与优化

<img src="https://cos.liuqm.cc/01-20221008135216847.jpeg" style="width:100%">

<img src="https://cos.liuqm.cc/%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6.jpg">

# 二、部署文档

**简介**

```
python3 

uwsgi    wsgi(web服务网关接口，就是一个实现了python web应用的协议)

virtualenvwrapper # 虚拟环境

drf的代码

vue的代码

nginx (一个是nginx对静态文件处理的优秀性能，一个是nginx的反向代理功能，以及nginx的默认80端口，访问nginx的80端口，就能反向代理到应用的8000端口)

mysql 


supervisor 进程管理工具(可选)
```

## 2.1 环境准备

```
1.部署环境准备，准备python3和虚拟环境解释器,virtualenvwrapper

    1.1 安装python3
        https://www.cnblogs.com/liuqimeng/articles/11284465.html ==> 1.3 小节

    1.2 安装virtualenvwrapper
        pip3 install -i https://pypi.douban.com/simple virtualenvwrapper 
        注意：直接安装virtualenvwrapper，同时也会下载virtualenv

2.修改python3的环境变量，写入到/etc/profile中
    PATH=/opt/python36/bin:$PATH
    source /etc/profile

3.virtualenvwrapper相关配置

    3.1 ~/.bashrc中
        export WORKON_HOME=~/Envs                                      #设置virtualenv的统一管理目录
        export VIRTUALENVWRAPPER_VIRTUALENV_ARGS='--no-site-packages'  #生成干净隔绝的环境
        export VIRTUALENVWRAPPER_PYTHON=/opt/python36/bin/python3      #指定python解释器
        source /opt/python36/bin/virtualenvwrapper.sh                  #执行virtualenvwrapper安装脚本

    3.2 修改完后
        退出后重新进，可以看到virtualenvwrapper提示的一些信息


4.新建一个虚拟环境 
    mkvirtualenv  blog
```

## 2.2 前端准备

```
5.准备前后端代码
    将前后端代码下载到linux中

6.解压缩代码


7.前端准备
    1. 准备node打包环境
        wget https://nodejs.org/download/release/v9.9.0/node-v9.9.0-linux-x64.tar.xz

    2. 解压缩node包，配置环境变量
        PATH=/opt/node9/bin:$PATH

    3. 检测node和npm
        node -v 
        npm  -v 

    4. 安装vue项目所需的包
        第一步 cd 前端代码目录
        第二步 npm install  
        第三部 npm run build  
        温馨提示：这两条都正确配置之后，就会生成一个 dist 静态文件目录，整个项目的前端内容都在这里了

```

## 2.3 后端准备

```
1.激活虚拟环境
        workon blog 

2.通过一条命令，导出本地的所有软件包依赖 
    pip3 freeze >  requirements.txt 

3.安装依赖库
    pip3 install -r  requirements.txt  
```

## 2.4 开始部署

> http/https协议使用的uwsgi服务器
>
> ws/wss协议使用的daphne 服务器

### 2.4.1 uwsgi配置与使用

```
4. 准备uwsgi 支持高并发的启动python项目(uwsgi不支持静态文件的解析，必须用nginx去处理静态文件)
        1. 安装uwsgi 
            pip3 install -i https://pypi.douban.com/simple uwsgi

        2. uwsgi的使用方法

            2.1 命令行方式uwsgi启动应用
                2.1.1 启动一个web文件
                    uwsgi --http :8000 --wsgi-file   test.py
                        --http 指定http协议 
                        --wsgi-file  指定一个python文件

                    test.py内容如下
                        def application(env, start_response):
                            start_response('200 OK', [('Content-Type','text/html')])
                            return [b"Hello World"]

                2.1.2 启动一个django应用，并且支持热加载项目，不重启项目，自动生效
      
                        uwsgi --http  :8000 --module luffy.wsgi   --py-autoreload=1
              
                            --module 指定找到django项目的wsgi.py文件

            2.2 配置文件方式启动应用          
                2.2.1 创建一个uwsgi.ini配置文件，写入参数信息  
                    注意：uwsgi.ini最好放在项目的根目录下，方便管理，配置。

                    [uwsgi]

                    # 指定项目的绝对路径
                    chdir           = /opt/luffy/luffyapi/  


                    # django项目的wsgi文件的位置(写相对于chdir的相对路径)
                    module          = luffyapi.wsgi  


                    # 写入虚拟环境解释器的绝对路径
                    home            = /root/env/luffy  

                    master          = true


                    # 指定uwsgi启动的进程个数          
                    processes       = 1    


                    #uwsgi启动一个socket连接，当你使用nginx+uwsgi的时候，应该使用socket参数
                    socket          = 0.0.0.0:9000

                    # 这个参数是uwsgi启动一个http连接，当你不用nginx只用uwsgi的时候，使用这个参数
                    http  =  0.0.0.0:9000  


                    vacuum          = true


                    # 后台运行uwsgi(使用supervisor进程管理工具的时候，不要使用这个参数，不过我一般都不会用)
                    daemonize=yes

            2.3 使用uwsgi配置文件启动项目
                uwsgi --ini  uwsgi.ini
          
          
```

**uwsgi docker线上配置**

```shell
[uwsgi]
chdir           = /home/blog_api/
module          = blog_api.wsgi
master          = true
processes       = 3
socket            = 0.0.0.0:9000
vacuum          = true
```

### 2.4.2 daphne配置与使用

```python
daphne -b 0.0.0.0 -p 8001 blog_api.asgi:application
pip install daphne
配置asgi.py文件
    import os
    import django
    from channels.routing import get_default_application

    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "blog_api.settings.dev")  # joyoo 为项目名，需要修改成你自己的
    django.setup()
    application = get_default_application()
    # 跟wsgi.py在同一个目录下，当然还有路由那些就不写出来了，这里只写重点
```

### 2.4.3 supervisor进程管理工具(可选)

> 详情见http://www.liuqm.cc/article/158

```
1. 将linux进程运行在后台的方法有哪些
            1.1 命令后面加上&
                python  manage.py  runserver & 
      
            1.2 使用nohup命令

            1.3 使用进程管理工具

    2. 安装supervisor，使用python2的包管理工具 easy_install（注意，此时要退出虚拟环境！！！）
        如果没有easy_install
        yum install python-setuptools
  
    3. easy_install supervisor

    4. 通过命令，生成一个配置文件（这个文件就是写入你要管理的进程任务）
        echo_supervisord_conf > /etc/supervisor.conf

    5. 编辑这个配置文件，写入操作  django项目的 命令 
            vim /etc/supervisor.conf  
            直接到最底行，写入以下配置
                [program:luffy]       # 任务名，自定义
                command=/root/env/luffy/bin/uwsgi  --ini  /opt/luffy/luffyapi/uwsgi.ini  # 本质就是uwsgi通过uwsgi.ini配置文件启动了项目

    6. 指定配置文件启动supervisord服务端
            supervisord -c  /etc/supervisor.conf

    7.通过supervisorctl创建管理任务
        supervisorctl -c /etc/supervisor.conf 

        7.1 管理进程任务
            >  stop   luffy 
            >  start  luffy 
            >  status luffy 

    8.启动luffy的后端代码
        >stop luffy
```

### 2.4.4 配置nginx

```nginx
1.编译安装nginx
            安装完成后，配置环境变量
            /etc/profile文件添加一行
                PATH=/opt/nginx112/sbin:$PATH

2.nginx.conf配置如下
			#第一个server虚拟主机是为了找到vue的dist文件， 找到路飞的index.html
      server {
          listen       80;
          server_name  www.lqmblog,com;#当请求来自于 www.liuqm.cc/的时候，直接进入以下location，然						后找到vue的dist/index.html 
       		location / {
              root   /opt/luffy/luffycity/dist;
              index  index.html;
              #这一条参数确保vue页面刷新时候，不会出现404页面
              try_files $uri $uri/ /index.html;
					}  
      }
          
            #由于vue发送的接口数据地址是 api.liuqm.cc:8000，我们还得再准备一个入口server
                server {
                    listen 8000;
                    server_name  api.liuqm.cc;
              
                    #当接收到接口数据时，请求url是 api.liuqm.cc:8000 就进入如下location
                    location /  {
                        #这里是nginx将请求转发给,uwsgi启动的,9000端口
                        uwsgi_pass  api.liuqm.cc:9000;
                        # include  就是一个“引入的作用”，就是将外部一个文件的参数，导入到当前的nginx.conf中生效
                        include /opt/nginx112/conf/uwsgi_params;
                    }
                }

        3.	# 启动nginx
        		# nginx 启动
        		# nginx -s reload 重启
        		# nginx -s stop 停止
        		# 不加配置文件参数，默认以nginx.conf启动
            # 此时可以访问 www.liuqm.cc 查看页面结果
```

**nginx docker线上配置**

```nginx
worker_processes  1;
events {
    worker_connections  1024;
}
http {
        # 前端
        server {
                listen       80;
                server_name  www.liuqm.cc;
                location / {
                    root   /opt/project/blog/blog/dist;
                    index  index.html;
                    try_files $uri $uri/ /index.html;
                }
        }
        # 后端：nginx代理8000端口，实际服务运行在9000端口
        server {
            listen 8000;
            server_name  api.liuqm.cc;
            location /  {
                uwsgi_pass  api.liuqm.cc:9000;
                include /opt/nginx112/conf/uwsgi_params;
            }
            location /static/{
                    alias /opt/project/blog/blog_api/blog_api/static/;
            }
            #websocket(ssh系统)
            location /webssh/ {
                proxy_pass http://127.0.0.1:8001;  #对应channel启动端口
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_redirect off;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Host $server_name;
            }

        }
        # 优化
        gzip on;
        gzip_static on;
        gzip_min_length 1k;
        gzip_buffers    4 32k;
        gzip_http_version 1.1;
        gzip_comp_level 6;
        gzip_types text/plain text/css text/javascriptapplication/json application/javascript application/x-javascriptapplication/xml image/jpeg image/gif image/png font/ttf font/x-woff;
        gzip_vary on;
        gzip_proxied any;
        include       mime.types;
        default_type  application/octet-stream;

}
```

### 2.4.5 静态资源与优化

#### 处理静态资源

> 由于我们的项目跑在uwsgi上，刚好uwsgi不处理静态资源，
> 所以我们需要nginx来处理。

```
nginx.conf
		server {
					listen 8000;
					server_name  api.liuqm.cc;
					location /  {
							uwsgi_pass api.liuqm.cc:9000;
							include /opt/nginx112/conf/uwsgi_params;
					}
					location /static/{
							alias /opt/blog/blog_api/blog_api/static/;
					}
			}
```

#### 访问速度优化

> 这里我们可以开启前后端的压缩功能，前后端满足一定条件都压缩成.gz文件
> 返回给浏览器，浏览器自己解包

**1. 前端**

```
vue.config.js
		const CompressionPlugin = require("compression-webpack-plugin")
		module.exports = {
			devServer: {
				host: "www.liuqm.cc",
				port: 1024,
				// disableHostCheck: true
			},
			chainWebpack: config => {
				// 开启图片压缩
				config.module
					.rule('images')
					.use('image-webpack-loader')
					.loader('image-webpack-loader')
					.options({
						bypassOnDebug: true
					})
					.end()
				// 开启js、css压缩
				if (process.env.NODE_ENV === 'production') {
					config.plugin('compressionPlugin')
						.use(new CompressionPlugin({
							test: /\.js$|\.html$|\.css$|\.ttf$|\.json$|\.woff2$|\.woff$/, // 匹配文件名
							threshold: 10240, // 对超过10k的数据压缩
							deleteOriginalAssets: false // 不删除源文件
						}))
				}
			},
		}
```

**2. nginx**

```
nginx.conf
		http {
			gzip on;
			gzip_static on; # 如果请求的文件已经压缩，则返回已压缩的文件，否则，就自己压缩，看gzip on参数
			gzip_min_length 1k;
			gzip_buffers    4 32k;
			gzip_http_version 1.1;
			gzip_comp_level 6;
			gzip_types text/plain text/css text/javascriptapplication/json application/javascript application/x-javascriptapplication/xml image/jpeg image/gif image/png font/ttf font/x-woff;
			gzip_vary on;
			gzip_proxied any;
			include       mime.types;
			default_type  application/octet-stream;
		}
```

# 三、相关东西

## 3.1 前端

### 文章图片

> 文章的图片均来自https://picsum.photos/

### 语法高亮

```
使用https://github.com/markedjs/marked解析markdown语法
使用https://github.com/highlightjs/highlight.js高亮markdown语法
```

### 图片懒加载

```javascript
cnpm install vue-lazyload --save-dev
```

### **main.js**

```javascript
// 图片懒加载
import VueLazyLoad from 'vue-lazyload'
Vue.use(VueLazyLoad,{
    error:require('@/assets/img/error.jpeg'),    # 出错时展示的图片
    loading:require('@/assets/img/loading.gif'), # 加载中展示的图片
    // attempt: 1
})
```

### ***.vue**

```javascript
<img v-lazy="article_obj.img" :key="article_obj.img">
//一定要加key，别问为什么，我也不直到
       
```

### ajax

> 前端向后台拿数据用的axios

```javascript
cnpm install axios --save
```

### marked

> 因为后端传来的是markdown格式的字符串，所以前端要展示的话，需要转换

### **安装**

```javascript
cnpm install marked --save
```

### **使用**

```javascript
import marked from "marked"; //markdown语法解析的组件

this.article_content = marked(this.article_obj.content)
```

### element-ui

> 前端很多地方用的element-ui，比如消息弹窗，input组件啥的
>
> cnpm install element-ui --save
>
> cnpm install babel-plugin-component -D

### 压缩优化

> 项目可以进行压缩优化，不然加载很慢

```python
cnpm install  compression-webpack-plugin --save-dev
cnpm install  image-webpack-loader --save-dev
```

```python
const CompressionPlugin = require("compression-webpack-plugin")

module.exports = {
 devServer: {
     host: "www.liuqm.cc",
     port: 1024,
     // disableHostCheck: true
 },
 chainWebpack: config => {
     // 开启图片压缩
     config.module
         .rule('images')
         .use('image-webpack-loader')
         .loader('image-webpack-loader')
         .options({
             bypassOnDebug: true
         })
         .end()
     // 开启js、css压缩
     if (process.env.NODE_ENV === 'production') {
         config.plugin('compressionPlugin')
             .use(new CompressionPlugin({
                 test: /\.js$|\.html$|\.css$|\.ttf$|\.json$|\.woff2$|\.woff$/, // 匹配文件名
                 threshold: 10240, // 对超过10k的数据压缩
                 deleteOriginalAssets: false // 不删除源文件
             }))
     }
 },
}
```

### 文章目录

> 文章里面的目录菜单用的github上开源项目
>
> https://github.com/KELEN/katelog

```python
用了三部分
  Takelog.min.js文件
  标记文章容器，目录容器
  css样式
```

### 评论

> https://too.pub/utterances，但是我没用成功

### 301问题

```python
如果后端有一条路由规则
    re_path(r"^articles/$", views.ArticlesAPIView.as_view()),
  
如果你前端是这样访问的http://127.0.0.1:8000/articles,末尾没有'/'，那么django会给你的url末尾加'/'，然后跳转到http://127.0.0.1:8000/articles/，在你的前端就会有两个请求，一个是301状态码的请求，一个应该是200状态码的请求
  
如果不想自动在末尾加'/'，只需要在settings.py中APPEND_SLASH=False
 
```

## 3.2 后端

### django-mdeditor

> 后端要写文章，我用的是markdown编辑器django-mdeditor
>
> github地址：https://github.com/pylixm/django-mdeditor

### 文件系统

> 文件系统的目录树采用ayui框架写的
>
> 文件系统目录树有个BUG，就是当目录下没有文件时，该目录会被识别成文件，因为用的是人家的的东西，所以我不知道怎么改这个bug

### CDN

> 阿里云cdn加速静态资源访问
>
> 原理是，创建了一个**加速域名cdn.liuqm.cc**，然后通过域名解析把**加速域名cdn.liuqm.cc**，指向了**源站api.liuqm.cc**
>
> **访问静态资源时就会这样访问：cdn.liuqm.cc/static/....如果没有就去源站(俗称回源):api.liuqm.cc/static里面找到，然后返回，并且cdn复制一份回它那里**
>
> 这玩意在开发环境不好调试，因为cdn回源的话，源站必须是公网ip或者域名。所以我只上传生产环境的代码配置

```python
settings.py
    STATIC_URL = 'http://cdn.liuqm.cc/static/'
  # 至于STATIC_URL可以为url我也不知道为什么，以前我一直认为它的作用就是取别名
  
nginx.conf
     location /static/{
                alias /opt/blog/blog_api/blog_api/static/;
        }
  
```

**跨域配置**

> 不知道咋回事，用字体图标font-awesome跨域了，进行了以下配置

<img src="https://cos.liuqm.cc/%E8%B7%A8%E5%9F%9F.png" style="width:100%">

## 3.3 服务器

### 3.3.1 mysql数据转sqlite

> 将mysql的数据导成txt文件，然后连接sqlite，我这里是直接连接的django里面的sqlite文件，然后在把txt文件导入即可

<img src="https://cos.liuqm.cc/image-20210817164515004.png">

<img src="https://cos.liuqm.cc/image-20210817164536440.png">

### 3.3.2 sqlite转mysql

**1 创建mysql表**

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': "blog",
        "USER": "root",
        "PASSWORD": "xxxx",
        "HOST": "127.0.0.1",
        "PORT": 3306
    }
}
# python manage.py makemigrations
# python manage.py migrate
```

**2 导出sqlite数据**

```python
# DATABASES = {
#     'default': {
#         'ENGINE': 'django.db.backends.mysql',
#         'NAME': "blog",
#         "USER": "root",
#         "PASSWORD": "lqm576576",
#         "HOST": "127.0.0.1",
#         "PORT": 3306
#     }
# }

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3')
    }
}
# python manage.py dumpdata > data.json

```

**3 将数据拷贝进mysql**

```python
# python manage.py loaddata data.json
```

# 四、常见问题

**问题1**

> functionName is defined but never used'

**解决**

```json
package.json中
"rules": {
  "no-unused-vars": "off"
}
```

**问题2**

> RuntimeError: Model class app_anme.models.Ad doesn't declare an explicit app_label and isn't in an application in INSTALLED_APPS.

由于已在dev.py中配置apps路径，所以我们这里app之间互相导model，直接使用

```
from app_name.models import *
from app_name import models
```

**问题3**

> django.db.utils.InternalError: (1060, "Duplicate column name 'user_id'")

```python
# 删除掉migration下的文件(除了__init__.py)再：
python manage.py makemigrations
pytohn manage.py migrate
```

**问题4**

路径放在数组里面批量解析不生效，解决方法如下

```python
other_data:[
        {
        "name":"github",
        "src":require("@/assets/img/login-component/github-logo.jpeg"),
        "class":"github-logo"
      },
      {
        "name":"qq",
        "src":require("@/assets/img/login-component/qq-logo.jpeg"),
        "class":"qq-logo"
      },
      ]
```

**问题五5**

```
eslint语法检测，如何关闭

vue.config.js中加一行lintOnSave:false
```

**问题6**

> 项目中如果要使用相对导入，因为我的blog_api的项目已经改过结构了，所以相对导入有很多问题，为什么这么说，因为我试过在不同app之间使用项目导入，我的app都在apps的包下，还是报错了。不过我可以在每个包内部使用相对导入，这样不会报错

**问题7**

> channels相关问题
>
> python解释器：3.9
>
> channels：3.0.3
>
> Django:3.2.1

routing.py中

> 这里需要判断一下python解释器的版本号，大于3.6或者小于3.6需要额外设置

```python
from channels.routing import ProtocolTypeRouter, URLRouter
# from django.conf.urls import url
from app01 import consumers
from django.urls import path,re_path
import sys
if float(sys.version[:3]) > 3.6:
    ChatConsumer = consumers.ChatConsumer.as_asgi()
else:
    ChatConsumer = consumers.ChatConsumer
application = ProtocolTypeRouter({
    'websocket': URLRouter([
        re_path(r'^app01/$', ChatConsumer),
    ])
})

```

**问题8**

> 当刷新或者进入有滚轮的页面时，会造成loading组件的元素抖动，解决方法

```css
html{
    overflow-y: scroll;
}