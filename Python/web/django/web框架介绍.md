---
title: web框架介绍
---

## 1.1 自己写网站

```python
a. socket服务端
b. 路由系统，一个url对应一个视图函数
c. 模板系统，HTML充当模板，自己填充，返回字符串

# django：b,c都是自己写的，a用的第三方的wsgiref
# tornado:a,b,c都是自己写的
# flask:a,c用的第三方，b是自己写的
```

**第三方的socket服务端**

```python
'cgi': CGIServer,
'flup': FlupFCGIServer,
'wsgiref': WSGIRefServer,
'waitress': WaitressServer,
'cherrypy': CherryPyServer,
'paste': PasteServer,
'fapws3': FapwsServer,
'tornado': TornadoServer,
'gae': AppEngineServer,
'twisted': TwistedServer,
'diesel': DieselServer,
'meinheld': MeinheldServer,
'gunicorn': GunicornServer,
'eventlet': EventletServer,
'gevent': GeventServer,
'geventSocketIO':GeventSocketIOServer,
'rocket': RocketServer,
'bjoern' : BjoernServer,
'auto': AutoServer,
```

**wsgiref模块**

> 最简单的Web应用就是先把HTML用文件保存好，用一个现成的HTTP服务器软件，接收用户请求，从文件中读取HTML，返回。
>
> 如果要动态生成HTML，就需要把上述步骤自己来实现。不过，接受HTTP请求、解析HTTP请求、发送HTTP响应都是苦力活，如果我们自己来写这些底层代码，还没开始写动态HTML呢，就得花个把月去读HTTP规范。
>
> **正确的做法是底层代码由专门的服务器软件实现，我们用Python专注于生成HTML文档。因为我们不希望接触到TCP连接、HTTP原始请求和响应格式，所以，需要一个统一的接口协议来实现这样的服务器软件，让我们专心用Python编写Web业务。这个接口就是WSGI：Web Server Gateway Interface。而wsgiref模块就是python基于wsgi协议开发的服务模块。**

```python
from wsgiref.simple_server import make_server

def run_server(environ, start_response):
    """
    :param environ:请求头的各种数据
    :param start_response:响应头
    :return:
    """
    print(environ)
    start_response('200 OK', [('Content-Type', 'text/html')])
    # django逻辑开始了
    if environ.get('PATH_INFO') == '/index':
        with open('index.html','rb') as f:
            data=f.read()

    elif environ.get('PATH_INFO') == '/login':
        with open('login.html', 'rb') as f:
            data = f.read()
    else:
        data=b'<h1>Hello, web!</h1>'
    # django逻辑结束了
    return [data]

if __name__ == '__main__':
    server = make_server('127.0.0.1', 8000, run_server)
    server.serve_forever()
```

## 1.2 web框架本质

**py文件**

```python
import socket

import pymysql
def index(request):
    """
    :param request:http响应报文
    :return:
    """
    return "index"

def login(request):
    """

    :param request: http响应报文
    :return:
    """
    with open('login.html','r',encoding='utf-8') as f :
        data=f.read()
    return data

def time(request):
    """
    :param request: http响应报文
    :return:
    """
    import datetime
    now=datetime.datetime.now().strftime('%Y-%m-%d %X')
    with open('time.html','r',encoding='utf-8') as f :
        data=f.read()
    data=data.replace('@@time@@',now)
    return data



def role_list(request):
    """
    自己做模板渲染
    :param request: http响应报文
    :return:
    """
    # 创建连接
    conn = pymysql.connect(host='127.0.0.1', port=3306, user='root', passwd='lqm576576', db='blog')
    cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
    cursor.execute("select * from rbac_role")
    user_list = cursor.fetchall()
    cursor.close()
    conn.close()
    tr_list=[]
    for row in user_list:
        tr='<tr><td>%s</td><td>%s</td></tr>'%(row['id'],row['title'])
        tr_list.append(tr)


    with open('role_list.html', 'r', encoding='utf-8') as f:
        data=f.read()
    data=data.replace('@@body@@',''.join(tr_list))
    return data

def jinja2_role_list(request):
    """
    基于第三方工具jinja2实现模板渲染工作
    :param request: http响应报文
    :return:
    """
    # 创建连接
    conn = pymysql.connect(host='127.0.0.1', port=3306, user='root', passwd='lqm576576', db='blog')
    cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
    cursor.execute("select * from rbac_role")
    role_list = cursor.fetchall()
    cursor.close()
    conn.close()
    with open('jinja2_role_list.html','r',encoding='utf-8') as f:
        data=f.read()
    from jinja2 import Template
    template=Template(data)
    response=template.render(role_list=role_list) # # response=template.render({'role_list':role_list})
    return response


urlpatterns = [
    ('/index', index),
    ('/login', login),
    ('/time', time),
    ('/role_list', role_list),
    ('/jinja2_role_list', jinja2_role_list),
]


def run():
    soc = socket.socket()
    soc.setsockopt(socket.SOL_SOCKET,socket.SO_REUSEADDR,1) #就是它，在bind前加
    soc.bind(('127.0.0.1', 8080))
    soc.listen(5)
    while True:
        conn, port = soc.accept()
        data = conn.recv(8096)
        data = str(data, encoding='utf-8')
        request_list = data.split('\r\n\r\n')
        head_list = request_list[0].split('\r\n')

        method, url, protocol = head_list[0].split(' ')

        conn.send(b'HTTP/1.1 200 OK \r\n\r\n')
        func_name = None
        for u_tuple in urlpatterns:
            if url == u_tuple[0]:
                func_name = u_tuple[1]
                break
        if func_name:
            response = func_name(data)
        else:
            response = '404 not found'

        conn.send(response.encode('utf-8'))
        conn.close()


if __name__ == '__main__':
    run()
```

**jinja2_role_list.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>角色列表</title>
</head>
<body>
<table border="1">
    <thead>
    <tr>
        <th>id</th>
        <th>title</th>
    </tr>
    </thead>
    <tbody>

    {% for role in role_list%}
    <tr>
        <td>{{role.id}}</td>
        <td>{{role.title}}</td>
    </tr>
    {%endfor%}


    </tbody>


</table>

</body>
</html>
```

**login.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登陆</title>
</head>
<body>

<form action="">
    <p>用户名：<input type="text"></p>
    <p>密码：<input type="password"></p>


</form>
</body>
</html>
```

**role_list.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>角色列表</title>
</head>
<body>
  <table>
    <th>
      <td>ID</td>
      <td>title</td>
    </th>
    @@body@@
  </table>
</body>
</html>
```

**time.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>角色列表</title>
</head>
<body>
  <table>
    <th>
      <td>ID</td>
      <td>title</td>
    </th>
    @@body@@
  </table>
</body>
</html>