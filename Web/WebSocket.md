## 1.1 轮询

> 客户端不断向服务端发起请求，服务端不断询问消息，回复客户端

### 优势

> 数据实时性

### 劣势

> 前后端资源浪费严重，带宽/流量的浪费

## 1.2 长轮询

> 浏览器发送ajax请求，服务器为每个请求设置一个队列，并且队列设置一个超时时间，这样浏览器取不到数据，就会在超时时间内阻塞住，直到超时时间结束，随之连接也断开了
>
> 这样重复的隔一段时间发一次请求，可以减轻服务器请求压力

### 优势

> 节省带宽

### 劣势

> 不能保证数据实时性

## 1.3 websocket

> 我们一直使用的http协议只能由客户端发起，服务端无法直接进行推送，这就导致了如果服务端有持续的变化客户端想要获知就比较麻烦。WebSocket协议就是为了解决这个问题应运而生。
>
> WebSocket协议，客户端和服务端都可以主动的推送消息，可以是文本也可以是二进制数据。而且没有同源策略的限制，不存在跨域问题。协议的标识符就是`ws`。像https一样如果加密的话就是`wxs`。

### websocket群聊

**客户端**

```html
<!doctype html>
<html>
<head>
   <meta charset="utf-8">
   <title>网站标题</title>
   <meta name="keywords" content="网站关键字">
   <meta name="description" content="网站描述信息">
   <meta name="author" content="Meng">
</head>
<body>
<div id="chat"></div>

<script type="text/javascript">
   var ws = new WebSocket("ws://127.0.0.1:5000/ws")
   ws.onmessage = function (data) {
       var p = document.createElement("p")
      p.innerText = data.data
      document.getElementById("chat").appendChild(p)
    }
</script>
</body>
</html>
```

**服务端**

```python
from flask import Flask,request
from geventwebsocket.handler import WebSocketHandler
from gevent.pywsgi import WSGIServer
from geventwebsocket.websocket import WebSocket # 做语法提示用
import json
app = Flask(__name__)
client_list = []
@app.route("/ws")
def ws():
    client = request.environ.get("wsgi.websocket") # type:WebSocket
    print(f"有一个连接{client}")
    client_list.append(client)
    print(f"总连接{client_list}")
    while True:
        try:
            print(1)
            msg = client.receive()
            print(2)
            for client_ in client_list:
                client_.send(msg)
        except Exception:
            client_list.remove(client)

if __name__ == '__main__':
    http_server = WSGIServer(("0.0.0.0",5000),app,handler_class=WebSocketHandler)
    http_server.serve_forever()
```

### 单聊

**客户端**

```html
<!doctype html>
<html>
<head>
	<meta charset="utf-8">
	<title>网站标题</title>
	<meta name="keywords" content="网站关键字">
	<meta name="description" content="网站描述信息">
	<meta name="author" content="Meng">
</head>
<body>
<input type="text" id="username" placeholder="请输入用户名">
<button onclick="open_ws()">连接服务器</button>
<p>给<input type="text" id="to_user"></p>
<p>发送消息: <input type="text" id="msg"><button onclick="send_msg()">发送消息</button></p>
<div id="chat"></div>

<script type="text/javascript">
	var ws = null;
    function open_ws() {
        let username = document.getElementById("username").value
        ws = new WebSocket("ws://127.0.0.1:5000/ws/" + username)
        ws.onmessage = function (response) {

            msg = JSON.parse(response.data)
			print(msg)

			from_user = msg.from_user
			msg = msg.msg

			var p = document.createElement("p")
            p.innerText = from_user + "：" + msg
            document.getElementById("chat").appendChild(p)
        }
    }

    function send_msg() {
		let msg = document.getElementById("msg").value
		let to_user = document.getElementById("to_user").value
		let send_msg = {"to_user":to_user,"msg":msg}
		ws.send(JSON.stringify(send_msg))
    }

</script>
</body>
</html>
```

**服务端**

```python
from flask import Flask,request,render_template
from geventwebsocket.handler import WebSocketHandler
from gevent.pywsgi import WSGIServer
import json
from geventwebsocket.websocket import WebSocket # 做语法提示用
import json
app = Flask(__name__)
client_list = []
user_dict = {}
@app.route("/ws/<username>")
def ws(username):
    client = request.environ.get("wsgi.websocket") # type:WebSocket
    user_dict[username] = client
    print(user_dict)
    while True:
        try:
            res = json.loads(client.receive())
            print(res)
            to_user = res["to_user"]
            send_msg = json.dumps({"from_user":username,"msg":res["msg"]})
            user_dict.get(to_user).send(send_msg)
        except Exception as e:
            pass
@app.route("/chat")
def chat():
    return render_template("websocket单聊.html")
if __name__ == '__main__':
    http_server = WSGIServer(("0.0.0.0",5000),app,handler_class=WebSocketHandler)
    http_server.serve_forever()
```

### 握手

**客户端**

```html
<!doctype html>
<html>
<head>
	<meta charset="utf-8">
	<title>网站标题</title>
	<meta name="keywords" content="网站关键字">
	<meta name="description" content="网站描述信息">
	<meta name="author" content="Meng">
</head>
<body>
<script type="text/javascript">
	var ws = new WebSocket("ws://127.0.0.1:9527")
	ws.onmessage = function (data) {
		console.log(JSON.parse(data.data))
    }
</script>
</body>
</html>
```

**服务端**

```python
import socket, base64, hashlib

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
sock.bind(('127.0.0.1', 9527))
sock.listen(5)
# 获取客户端socket对象
conn, address = sock.accept()
# 获取客户端的【握手】信息
data = conn.recv(1024)
# data =
"""
b'GET / HTTP/1.1\r\n
Host: 127.0.0.1:9527\r\n
Connection: Upgrade\r\n
Pragma: no-cache\r\n
Cache-Control: no-cache\r\n
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.97 Safari/537.36\r\n
Upgrade: websocket\r\n
Origin: http://localhost:63342\r\n
Sec-WebSocket-Version: 13\r\n
Accept-Encoding: gzip, deflate, br\r\n
Accept-Language: zh-CN,zh;q=0.9\r\n
Cookie: csrftoken=SWtt8cglETIC7Kla9hVadbOnwLRSOBpFwFeYsgVchd3mij9ucPmZirapMkZ8of9U\r\n
Sec-WebSocket-Key: tDXVE3a5pXSFEKGPT01PWQ==\r\n
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits\r\n\r\n'
tDXVE3a5pXSFEKGPT01PWQ==258EAFA5-E914-47DA-95CA-C5AB0DC85B11
"""

magic_string = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11' # 永不变


def get_headers(data):
    header_dict = {}
    header_str = data.decode("utf8")
    for i in header_str.split("\r\n"):
        if str(i).startswith("Sec-WebSocket-Key"):
            header_dict["Sec-WebSocket-Key"] = i.split(":")[1].strip()
    return header_dict


def get_header(data):
    """
     将请求头格式化成字典
     :param data:
     :return:
     """
    header_dict = {}
    data = str(data, encoding='utf-8')

    header, body = data.split('\r\n\r\n', 1)
    header_list = header.split('\r\n')
    for i in range(0, len(header_list)):
        if i == 0:
            if len(header_list[i].split(' ')) == 3:
                header_dict['method'], header_dict['url'], header_dict['protocol'] = header_list[i].split(' ')
        else:
            k, v = header_list[i].split(':', 1)
            header_dict[k] = v.strip()
    return header_dict


headers = get_headers(data)  # 提取请求头信息
# 对请求头中的sec-websocket-key进行加密
response_tpl = "HTTP/1.1 101 Switching Protocols\r\n" \
               "Upgrade:websocket\r\n" \
               "Connection: Upgrade\r\n" \
               "Sec-WebSocket-Accept: %s\r\n" \
               "WebSocket-Location: ws://127.0.0.1:9527\r\n\r\n"

value = headers['Sec-WebSocket-Key'] + magic_string

ac = base64.b64encode(hashlib.sha1(value.encode('utf-8')).digest())
response_str = response_tpl % (ac.decode('utf-8'))
# 响应【握手】信息
conn.send(response_str.encode("utf8"))

while True:
    msg = conn.recv(8096)
    print(msg) # b'\x81\x83\xceH\xb6\x85\xffz\x85'
```

### 解密

```python
msg = b'\x81\x83\xceH\xb6\x85\xffz\x85'
data = 0
mask = ""

# 将第二个字节（\x83第9-16位） 进行与127进行位运算
payload = msg[1] & 127 # 131 & 127 = 3

if payload == 127:
    extend_payload_len = msg[2:10]
    mask = msg[10:14]
    data = msg[14:]
# 当位运算结果等于127时,则第3-10个字节为数据长度
# 第11-14字节为mask 解密所需字符串
# 第15字节至结尾为数据


if payload == 126:
    extend_payload_len = msg[2:4]
    mask = msg[4:8]
    data = msg[8:]
# 当位运算结果等于126时
# 第3-4个字节为数据长度
# 第5-8字节为mask 解密所需字符串
# 第9字节至结尾为数据


if payload <= 125:
    extend_payload_len = None
    mask = msg[2:6]
    data = msg[6:]

# 当位运算结果小于等于125时
# 这个数字就是数据的长度
# 第3-6字节为mask 解密所需字符串
# 第7字节至结尾为数据


str_byte = bytearray()
for i in range(len(data)):
    byte = data[i] ^ mask[i % 4]
    str_byte.append(byte)

print(str_byte.decode("utf8"))
```

### 加密

```python
import struct
msg_bytes = "hello".encode("utf8")
token = b"\x81"
length = len(msg_bytes)

if length < 126:
    token += struct.pack("B", length)
elif length == 126:
    token += struct.pack("!BH", 126, length)
else:
    token += struct.pack("!BQ", 127, length)

msg = token + msg_bytes

print(msg)