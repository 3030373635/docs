---
title: IO模型
---


# 一、IO模型

## 1.1 介绍

> 为了更好地了解IO模型，我们需要事先回顾下：同步、异步、阻塞、非阻塞
>
> http://www.cnblogs.com/linhaifeng/articles/7430066.html
>
> 本文讨论的背景是`Linux环境下的network IO`。

> Stevens在文章中一共比较了五种IO Model：
> \* blocking IO
> \* nonblocking IO
> \* IO multiplexing
> \* signal driven IO
> \* asynchronous IO
> 注意：signal driven IO（信号驱动IO）在实际中并不常用，所以主要介绍其余四种IO Model。

> 再说一下IO发生时涉及的对象和步骤。
>
> 对于一个network IO (这里我们以recv举例)，它会涉及到两个系统对象，
>
> 当一个recv操作发生时，该操作会经历两个阶段：
>
> 1. wait data：等待客户端发送数据到服务端操作系统缓存中（客户端产生数据->客户端OS->通过网络->服务端os）
> 2. copy data：将数据从操作系统缓存中拷贝到应用程序内存中
>
> **记住这两点很重要，因为这些IO模型的区别就是在两个阶段上各有不同的情况。**
>
> **补充**
>
> ```python
> #1、输入操作：read、readv、recv、recvfrom、recvmsg共5个函数，如果会阻塞状态，则会经历wait data和copy data两个阶段，如果设置为非阻塞则在wait 不到data时抛出异常
> 
> #2、输出操作：write、writev、send、sendto、sendmsg共5个函数，只经历copydata阶段。在发送缓冲区满了会阻塞在原地，如果设置为非阻塞，则会抛出异常
> 
> #3、接收外来链接：accept，与输入操作类似
> 
> #4、发起外出链接：connect，与输出操作类似
> ```

## 1.2  阻塞IO

> 在linux中，默认情况下所有的socket都是blocking，一个典型的读操作流程大概是这样

<img src="https://cos.liuqm.cc/1-1-20221010230745802.png" style="width:100%">

<img src="https://cos.liuqm.cc/%E9%98%BB%E5%A1%9EIO-20221010230753549.jpg">

> 当用户进程调用了recvfrom这个系统调用，kernel就开始了IO的第一个阶段：准备数据。对于network io来说，很多时候数据在一开始还没有到达（比如，还没有收到一个完整的UDP包），这个时候kernel就要等待足够的数据到来。
>
> 而在用户进程这边，整个进程会被阻塞。当kernel一直等到数据准备好了，它就会将数据从kernel中拷贝到用户内存，然后kernel返回结果，用户进程才解除block的状态，重新运行起来。
> **所以，blocking IO的特点就是在IO执行的两个阶段（等待数据和拷贝数据两个阶段）都被block了。**
>
> 几乎所有的程序员第一次接触到的网络编程都是从listen()、send()、recv() 等接口开始的，使用这些接口可以很方便的构建服务器/客户机的模型。然而大部分的socket接口都是阻塞型的。如下图
>
> ps：所谓阻塞型接口是指系统调用（一般是IO接口）不返回调用结果并让当前线程一直阻塞，只有当该系统调用获得结果或者超时出错时才返回。

<img src="https://cos.liuqm.cc/1-2-20221010230755819.png" style="width:100%">

> 实际上，除非特别指定，几乎所有的IO接口 ( 包括socket接口 ) 都是阻塞型的。这给网络编程带来了一个很大的问题，如在调用recv(1024)的同时，线程将被阻塞，在此期间，线程将无法执行任何运算或响应任何的网络请求。

**简单解决方案**

> \#在服务器端使用多线程（或多进程）。多线程（或多进程）的目的是让每个连接都拥有独立的线程（或进程），这样任何一个连接的阻塞都不会影响其他的连接。

**该方案的问题是**

> 开启多进程或都线程的方式，在遇到要同时响应成百上千路的连接请求，则无论多线程还是多进程都会严重占据系统资源，降低系统对外界响应效率，而且线程与进程本身也更容易进入假死状态。

**改进方案**

> 很多程序员可能会考虑使用“线程池”或“连接池”。“线程池”旨在减少创建和销毁线程的频率，其维持一定合理数量的线程，并让空闲的线程重新承担新的执行任务。“连接池”维持连接的缓存池，尽量重用已有的连接、减少创建和关闭连接的频率。这两种技术都可以很好的降低系统开销，都被广泛应用很多大型系统，如websphere、tomcat和各种数据库等。

**改进后方案其实也存在着问题**

> “线程池”和“连接池”技术也只是在一定程度上缓解了频繁调用IO接口带来的资源占用。而且，所谓“池”始终有其上限，当请求大大超过上限时，“池”构成的系统对外界的响应并不比没有池的时候效果好多少。所以使用“池”必须考虑其面临的响应规模，并根据响应规模调整“池”的大小。

> **对应上例中的所面临的可能同时出现的上千甚至上万次的客户端请求，“线程池”或“连接池”或许可以缓解部分压力，但是不能`解决所有问题`。**
>
> **总结：**
>
> **多线程模型可以方便高效的解决小规模的服务请求，但面对大规模的服务请求，多线程模型也会遇到瓶颈，可以用非阻塞接口来尝试解决这个问题。**

## 1.3 非阻塞IO

> Linux下，可以通过设置socket使其变为non-blocking。当对一个non-blocking socket执行读操作时

**流程如下**

<img src="https://cos.liuqm.cc/1-3-20221010230803500-20221010230808624.png" style="width:100%">

<img src="https://cos.liuqm.cc/%E9%9D%9E%E9%98%BB%E5%A1%9EIO.jpg">

> 从图中可以看出，当用户进程发出read操作时，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个error。从用户进程角度讲 ，它发起一个read操作后，并不需要等待，而是马上就得到了一个结果。用户进程判断结果是一个error时，它就知道数据还没有准备好，于是用户就可以在本次到下次再发起read询问的时间间隔内做其他事情，或者直接再次发送read操作。一旦kernel中的数据准备好了，并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存（这一阶段仍然是阻塞的），然后返回。
>
> 也就是说非阻塞的recvform系统调用调用之后，进程并没有被阻塞，内核马上返回给进程，如果数据还没准备好，此时会返回一个error。进程在返回之后，可以干点别的事情，然后再发起recvform系统调用。重复上面的过程，循环往复的进行recvform系统调用。这个过程通常被称之为轮询。轮询检查内核数据，直到数据准备好，再拷贝数据到进程，进行数据处理。需要注意，拷贝数据整个过程，进程仍然是属于阻塞的状态。
>
> **所以，在非阻塞式IO中，用户进程其实是需要不断的主动询问kernel数据准备好了没有。**

**服务端**

```python
"""
# 强调强调强调：！！！非阻塞IO的精髓在于完全没有阻塞！！！

"""
import socket
server = socket.socket()
server.setsockopt(socket.SOL_SOCKET,socket.SO_REUSEADDR,1) #关闭端口

server.bind(("127.0.0.1",8080))
server.listen(5) # 限制请求数
server.setblocking(False) # 取消所有的IO阻塞，当询问os没有数据时，会触发BlockingIOError异常
conn_l = [] # 里面存放建立的连接
while True:
    try:
        conn,client_addr = server.accept() # 向os询问有没有链接请求，没有就触发BlockingIOError
        conn_l.append(conn)
        print("当前列表有[%s]个链接" % len(conn_l))
    except BlockingIOError:
        del_l = []
        for conn in conn_l:
            try:
                data = conn.recv(1024) # 向os询问这一个链接有没有数据，没有数据触发BlockingIOError异常
                if(len(data)) == 0:
                    del_l.append(conn)
                    continue
                conn.send(data.upper())
            except BlockingIOError:
                continue
            except ConnectionResetError:
                # 当客户端断开连接时，触发
                del_l.append(conn)
                continue
        for conn in del_l:
            # 删除断开的连接
            conn_l.remove(conn)


```

**客户端**

```python
import socket

client = socket.socket()
client.connect(("127.0.0.1",8080))


while True:# 通信循环
    msg = input(">>：").strip()
    client.send(msg.encode("utf-8"))
    data = client.recv(1024)
    print(data)

client.close()


```

> **但是非阻塞IO模型绝不被推荐。**
>
> 优点：够在等待任务完成的时间里干其他活了（包括提交其他任务，也就是 “后台” 可以有多个任务在""同时""执行）
>
> 缺点：
>
> 1. 对CPU的无效占用率过高
> 2. 任务完成的响应延迟增大了，因为每过一段时间才去轮询一次read操作，而任务可能在两次轮询之间的任意时间完成。这会导致整体数据吞吐量的降低。
> 3. 链接太多，会导致链接列表循环太慢

> **此外，在这个方案中recv()更多的是起到检测“操作是否完成”的作用，实际操作系统提供了更为高效的检测“操作是否完成“作用的接口，例如select()多路复用模式，可以一次检测多个连接是否活跃。**

## 1.4 多路复用IO

> IO multiplexing这个词可能有点陌生，但是如果我说select/epoll，大概就都能明白了。有些地方也称这种IO方式为**事件驱动IO**(event driven IO)。我们都知道，select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。它的基本原理就是select/epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。非阻塞IO中我们自己去管理和询问数据是否准备好，而在select中，它会帮我们管理和询问数据，无需我们自己管理了

<img src="https://cos.liuqm.cc/1-4-20221010230812709.png" style="width:100%">

<img src="https://cos.liuqm.cc/IO%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8-20221010230817294.jpg">

> 当用户进程调用了select，那么整个进程会被block，而同时，kernel会“监视”所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程。
> 这个图和blocking IO的图其实并没有太大的不同，事实上还更差一些。因为这里需要使用两个系统调用(select和recvfrom)，而blocking IO只调用了一个系统调用(recvfrom)。但是，用select的优势在于它可以同时处理多个connection。

**强调**

> **1. 如果处理的连接数不是很高的话，使用select/epoll的web server不一定比使用multi-threading + blocking IO的web server性能更好，可能延迟还更大。select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。**
>
> **2. 在多路复用模型中，对于每一个socket，一般都设置成为non-blocking，但是，如上图所示，整个用户的process其实是一直被block的。只不过process是被select这个函数block，而不是被socket IO给block。**
>
> **结论: select的优势在于可以处理多个连接，不适用于单个连接**

### 1.4.1 select实现

**服务端**

```python
from socket import *
import select
server = socket(AF_INET,SOCK_STREAM)
server.setsockopt(SOL_SOCKET,SO_REUSEADDR,1)
server.bind(("127.0.0.1",8080))
server.listen(5)
read_list = [server,]
write_list = []
data_dic = {} # 将通信套接字与数据绑定 {conn1:"hello",conn2:"nihao"}
while True:
    rl,wl,xl = select.select(read_list,write_list,[])
    """
    read_list：需要监听的套接字，并且套接字是用来读消息的
    rl：哪些套接字可以读消息了，就把那些放进去
    write_list：需要监听的套接字，并且套接字是用来发消息的，其实就是通信套接字
    wl：哪些套接字可以发消息了，就把那些放进去
  

    select 发送一个系统调用，操作系统遍历这几个列表，当列表里面的套接字有数据时，就把对应套接字返回
    比如，read_list里面的server有数据了，也就是可以server.accept了，就会把server添加到rl中
    并且程序往下运行
 
 		timeout = x：为可选项，意义不大，阻塞xs,没有消息的话xs后则往下执行(有消息就返回),不设置超时时间的话，如果没有数据，就一直		阻塞住
    """
    print("read_list:%s r1:%s wl:%s" % (len(read_list),len(rl),len(wl)))
    for socket in rl:
        if socket == server:
            conn,addr = socket.accept()
            read_list.append(conn)
        else:
            data = socket.recv(1024)
            write_list.append(socket)
            data_dic[socket] = data

    for socket in wl:
        socket.send(data_dic[socket].upper())
        data_dic.pop(socket)
        write_list.remove(socket) # 通信完了，通信套接字没有意义了
    
    
"""
IO多路复用:
select 
		发送一个系统调用，操作系统遍历这个列表,有消息就拿出来（select模型特点）
		管理一堆套接字(conn,....)
		如果只管理一个conn,那效率还不如阻塞IO
		发动系统调用去问有没有消息到操作系统的缓存里面

		之前是conn自己去问(conn.recv(1024))，现在是select把conn管理起来了，一块去问操作系统

epoll 
		不是操作系统去遍历了,而是列表里面的套接字自己返回信息,就像是绑定了回调函数(windows不支持)
		跟非阻塞IO是差不多的,实现本质都一样


多路复用是像操作系统发送请求，有就没有就拿数据，在原地等操作系统有数据,
而不是随时都在像操作系统发请求

缺点:
1 如果客户端非常非常多，for循环那里很慢

"""
```

**客户端**

```python
import socket

client = socket.socket()
client.connect(("127.0.0.1",8080))


while True:# 通信循环
    msg = input(">>：").strip()
    client.send(msg.encode("utf-8"))
    data = client.recv(1024)
    print(data)

client.close()


```

**基于select和非阻塞的爬虫**

```python
"""
事件驱动，也叫事件循环
该框架从调用的方式来看是异步IO，但是内部却不是
"""

import socket
import select

class HttpRequest:
    def __init__(self,sk,host,callback):
        self.socket = sk
        self.host = host
        self.callback = callback

    def fileno(self):
        return self.socket.fileno()

class HttpResponse:
    def __init__(self,recv_data):
        self.recv_data = recv_data
        self.header_dic = {}
        self.body = ""
        self.initialize()
    def initialize(self):
        headers,body = self.recv_data.split(b'\r\n\r\n',1)
        self.body = body
        head_list = headers.split(b'\r\n')
        for h in head_list:
            # print(h)
            h_str = h.decode("utf-8")
            v = h_str.split(":",1)
            if(len(v) == 2):
                self.header_dic[v[0]] = v[1]
class AsyncioRequest:
    def __init__(self):
        self.conn = []
        self.connection = []

    def add_request(self,host,callback):
        sk = socket.socket()
        sk.setblocking(False)
        try:
            sk.connect((host,80))
        except BlockingIOError:
            pass

        request = HttpRequest(sk,host,callback)

        self.conn.append(request)
        self.connection.append(request)

    def run(self):
        # 事件循环
        while True:
            # self.conn:用来读的套接字列表，self.connection：用来写的套接字列表，当链接建立后，就可以写了
            rlist,wlist,elist = select.select(self.conn,self.connection,[])
            for w in wlist:
                # 只要循环到，表示客户端和服务器已经建立连接
                print(w.host,"连接成功")
                tpl = "GET / HTTP/1.0\r\nHost:%s\r\nUser-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36\r\n\r\n" % w.host
                w.socket.send(tpl.encode("utf-8"))
                self.connection.remove(w)
            for r in rlist:
                recv_data = b""
                while True:
                    try:
                        chunk = r.socket.recv(1024)
                        recv_data += chunk
                    except Exception as e:
                        break

                response = HttpResponse(recv_data)
                r.callback(response)
                r.socket.close()
                self.conn.remove(r)
            if(len(self.conn) == 0):break

def f1(response_obj):
    print("响应头：",response_obj.header_dic)
    print("响应体：",response_obj.body)
def f2(response_obj):
    print("响应头：",response_obj.header_dic)
    print("响应体：",response_obj.body)
def f3(data):
    print(data)

if __name__ == '__main__':


    url_list = [
        {"host":"www.baidu.com","callback":f1},
        {"host":"picsum.photos","callback":f2},
        #{"host":"cnblogs.com","callback":f3},

    ]
    req = AsyncioRequest()
    for item in url_list:
        req.add_request(item["host"],item["callback"])


    req.run()
```

### 1.4.2 poll实现

> 只在linux下有，poll和select都可以监管套接字，但是poll监管的更多

### 1.4.3 epoll实现

> 不是操作系统去遍历了,而是列表里面的套接字自己返回信息,就像是绑定了回调函数(windows不支持)
> 跟非阻塞IO是差不多的,实现本质都一样

## 1.5 异步IO

> 所有模型中，效率最高的，也是使用最多的
>
> 模块：asyncio模块
>
> 框架：sanic tronado twisted
>
> 这些都是支持异步IO的

<img src="https://cos.liuqm.cc/1-5-20221010230822180.png" style="width:100%">

<img src="https://cos.liuqm.cc/%E5%BC%82%E6%AD%A5IO.jpg">

> 用户进程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从kernel的角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，kernel会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，kernel会给用户进程发送一个signal，告诉它read操作完成了。

**栗子**

> http://www.liuqm.cc/article/99 的高性能章节的asyncio模块部分

## 1.6 各IO模型比较

> blocking和non-blocking的区别在哪，synchronous IO和asynchronous IO的区别在哪

**blocking IO（阻塞IO） VS  non-blocking IO（非阻塞IO）**

```python
前面的介绍中其实已经很明确的说明了这两者的区别。调用blocking IO会一直block住对应的进程直到操作完成，而non-blocking IO在kernel还准备数据的情况下会立刻返回。
```

**synchronous IO（同步IO）VS asynchronous IO（异步IO）**

> synchronous IO做`IO operation`的时候会将process阻塞
>
> 四个IO模型可以分为两大类，之前所述的blocking IO，non-blocking IO，IO multiplexing都属于synchronous IO这一类，而 asynchronous I/O属于后一类 。
>
> ```html
> 有人可能会说，non-blocking IO并没有被block啊。这里有个非常“狡猾”的地方，定义中所指的”IO operation”是指真实的IO操作，就是例子中的recvfrom这个system call。non-blocking IO在执行recvfrom这个system call的时候，如果kernel的数据没有准备好，这时候不会block进程。但是，当kernel中数据准备好的时候，recvfrom会将数据从kernel拷贝到用户内存中，这个时候进程是被block了，在这段时间内，进程是被block的。而asynchronous IO则不一样，当进程发起IO 操作之后，就直接返回再也不理睬了，直到kernel发送一个信号，告诉进程说IO完成。在这整个过程中，进程完全没有被block。
> ```

<img src="https://cos.liuqm.cc/1-6.png" style="width:100%">

<img src="https://cos.liuqm.cc/877318-20160731161330028-1449419644.png">

> 经过上面的介绍，会发现`non-blocking IO`和`asynchronous IO`的区别还是很明显的。
>
> 在non-blocking IO中，虽然进程大部分时间都不会被block，但是它仍然要求进程去主动的check，并且当数据准备完成以后，也需要进程主动的再次调用recvfrom来将数据拷贝到用户内存。
>
> 而asynchronous IO则完全不同。它就像是用户进程将整个IO操作交给了他人（kernel）完成，然后他人做完后发信号通知。在此期间，用户进程不需要去检查IO操作的状态，也不需要主动的去拷贝数据。

## 1.7 selectors模块

```html
IO复用：为了解释这个名词，首先来理解下复用这个概念，复用也就是共用的意思，这样理解还是有些抽象，为此，咱们来理解下复用在通信领域的使用，在通信领域中为了充分利用网络连接的物理介质，往往在同一条网络链路上采用时分复用或频分复用的技术使其在同一链路上传输多路信号，到这里我们就基本上理解了复用的含义，即公用某个“介质”来尽可能多的做同一类(性质)的事，那IO复用的“介质”是什么呢？为此我们首先来看看服务器编程的模型，客户端发来的请求服务端会产生一个进程来对其进行服务，每当来一个客户请求就产生一个进程来服务，然而进程不可能无限制的产生，因此为了解决大量客户端访问的问题，引入了IO复用技术，即：一个进程可以同时对多个客户请求进行服务。也就是说IO复用的“介质”是进程(准确的说复用的是select和poll，因为进程也是靠调用select和poll来实现的)，复用一个进程(select和poll)来对多个IO进行服务，虽然客户端发来的IO是并发的但是IO所需的读写数据多数情况下是没有准备好的，因此就可以利用一个函数(select和poll)来监听IO所需的这些数据的状态，一旦IO有数据可以进行读写了，进程就来对这样的IO进行服务。

  

理解完IO复用后，我们在来看下实现IO复用中的三个API(select、poll和epoll)的区别和联系

select，poll，epoll都是IO多路复用的机制，I/O多路复用就是通过一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知应用程序进行相应的读写操作。但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。三者的原型如下所示：

int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);

int poll(struct pollfd *fds, nfds_t nfds, int timeout);

int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);



 1.select的第一个参数nfds为fdset集合中最大描述符值加1，fdset是一个位数组，其大小限制为__FD_SETSIZE（1024），位数组的每一位代表其对应的描述符是否需要被检查。第二三四参数表示需要关注读、写、错误事件的文件描述符位数组，这些参数既是输入参数也是输出参数，可能会被内核修改用于标示哪些描述符上发生了关注的事件，所以每次调用select前都需要重新初始化fdset。timeout参数为超时时间，该结构会被内核修改，其值为超时剩余的时间。

 select的调用步骤如下：

（1）使用copy_from_user从用户空间拷贝fdset到内核空间

（2）注册回调函数__pollwait

（3）遍历所有fd，调用其对应的poll方法（对于socket，这个poll方法是sock_poll，sock_poll根据情况会调用到tcp_poll,udp_poll或者datagram_poll）

（4）以tcp_poll为例，其核心实现就是__pollwait，也就是上面注册的回调函数。

（5）__pollwait的主要工作就是把current（当前进程）挂到设备的等待队列中，不同的设备有不同的等待队列，对于tcp_poll 来说，其等待队列是sk->sk_sleep（注意把进程挂到等待队列中并不代表进程已经睡眠了）。在设备收到一条消息（网络设备）或填写完文件数 据（磁盘设备）后，会唤醒设备等待队列上睡眠的进程，这时current便被唤醒了。

（6）poll方法返回时会返回一个描述读写操作是否就绪的mask掩码，根据这个mask掩码给fd_set赋值。

（7）如果遍历完所有的fd，还没有返回一个可读写的mask掩码，则会调用schedule_timeout是调用select的进程（也就是 current）进入睡眠。当设备驱动发生自身资源可读写后，会唤醒其等待队列上睡眠的进程。如果超过一定的超时时间（schedule_timeout 指定），还是没人唤醒，则调用select的进程会重新被唤醒获得CPU，进而重新遍历fd，判断有没有就绪的fd。

（8）把fd_set从内核空间拷贝到用户空间。

总结下select的几大缺点：

（1）每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大

（2）同时每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大

（3）select支持的文件描述符数量太小了，默认是1024

 

2．  poll与select不同，通过一个pollfd数组向内核传递需要关注的事件，故没有描述符个数的限制，pollfd中的events字段和revents分别用于标示关注的事件和发生的事件，故pollfd数组只需要被初始化一次。

 poll的实现机制与select类似，其对应内核中的sys_poll，只不过poll向内核传递pollfd数组，然后对pollfd中的每个描述符进行poll，相比处理fdset来说，poll效率更高。poll返回后，需要对pollfd中的每个元素检查其revents值，来得指事件是否发生。

 

3．直到Linux2.6才出现了由内核直接支持的实现方法，那就是epoll，被公认为Linux2.6下性能最好的多路I/O就绪通知方法。epoll可以同时支持水平触发和边缘触发（Edge Triggered，只告诉进程哪些文件描述符刚刚变为就绪状态，它只说一遍，如果我们没有采取行动，那么它将不会再次告知，这种方式称为边缘触发），理论上边缘触发的性能要更高一些，但是代码实现相当复杂。epoll同样只告知那些就绪的文件描述符，而且当我们调用epoll_wait()获得就绪文件描述符时，返回的不是实际的描述符，而是一个代表就绪描述符数量的值，你只需要去epoll指定的一个数组中依次取得相应数量的文件描述符即可，这里也使用了内存映射（mmap）技术，这样便彻底省掉了这些文件描述符在系统调用时复制的开销。另一个本质的改进在于epoll采用基于事件的就绪通知方式。在select/poll中，进程只有在调用一定的方法后，内核才对所有监视的文件描述符进行扫描，而epoll事先通过epoll_ctl()来注册一个文件描述符，一旦基于某个文件描述符就绪时，内核会采用类似callback的回调机制，迅速激活这个文件描述符，当进程调用epoll_wait()时便得到通知。

 

epoll既然是对select和poll的改进，就应该能避免上述的三个缺点。那epoll都是怎么解决的呢？在此之前，我们先看一下epoll 和select和poll的调用接口上的不同，select和poll都只提供了一个函数——select或者poll函数。而epoll提供了三个函 数，epoll_create,epoll_ctl和epoll_wait，epoll_create是创建一个epoll句柄；epoll_ctl是注 册要监听的事件类型；epoll_wait则是等待事件的产生。

　　对于第一个缺点，epoll的解决方案在epoll_ctl函数中。每次注册新的事件到epoll句柄中时（在epoll_ctl中指定 EPOLL_CTL_ADD），会把所有的fd拷贝进内核，而不是在epoll_wait的时候重复拷贝。epoll保证了每个fd在整个过程中只会拷贝 一次。

　　对于第二个缺点，epoll的解决方案不像select或poll一样每次都把current轮流加入fd对应的设备等待队列中，而只在 epoll_ctl时把current挂一遍（这一遍必不可少）并为每个fd指定一个回调函数，当设备就绪，唤醒等待队列上的等待者时，就会调用这个回调 函数，而这个回调函数会把就绪的fd加入一个就绪链表）。epoll_wait的工作实际上就是在这个就绪链表中查看有没有就绪的fd（利用 schedule_timeout()实现睡一会，判断一会的效果，和select实现中的第7步是类似的）。

　　对于第三个缺点，epoll没有这个限制，它所支持的FD上限是最大可以打开文件的数目，这个数字一般远大于2048,举个例子, 在1GB内存的机器上大约是10万左右，具体数目可以cat /proc/sys/fs/file-max察看,一般来说这个数目和系统内存关系很大。

总结：

（1）select，poll实现需要自己不断轮询所有fd集合，直到设备就绪，期间可能要睡眠和唤醒多次交替。而epoll其实也需要调用 epoll_wait不断轮询就绪链表，期间也可能多次睡眠和唤醒交替，但是它是设备就绪时，调用回调函数，把就绪fd放入就绪链表中，并唤醒在 epoll_wait中进入睡眠的进程。虽然都要睡眠和交替，但是select和poll在“醒着”的时候要遍历整个fd集合，而epoll在“醒着”的 时候只要判断一下就绪链表是否为空就行了，这节省了大量的CPU时间，这就是回调机制带来的性能提升。

（2）select，poll每次调用都要把fd集合从用户态往内核态拷贝一次，并且要把current往设备等待队列中挂一次，而epoll只要 一次拷贝，而且把current往等待队列上挂也只挂一次（在epoll_wait的开始，注意这里的等待队列并不是设备等待队列，只是一个epoll内 部定义的等待队列），这也能节省不少的开销。
```

> 这三种IO多路复用模型在不同的平台有着不同的支持，而epoll在windows下就不支持，好在我们有selectors模块，帮我们默认选择当前平台下最合适的

```python
#服务端
from socket import *
import selectors

sel=selectors.DefaultSelector()
def accept(server_fileobj,mask):
    conn,addr=server_fileobj.accept()
    sel.register(conn,selectors.EVENT_READ,read)

def read(conn,mask):
    try:
        data=conn.recv(1024)
        if not data:
            print('closing',conn)
            sel.unregister(conn)
            conn.close()
            return
        conn.send(data.upper()+b'_SB')
    except Exception:
        print('closing', conn)
        sel.unregister(conn)
        conn.close()



server_fileobj=socket(AF_INET,SOCK_STREAM)
server_fileobj.setsockopt(SOL_SOCKET,SO_REUSEADDR,1)
server_fileobj.bind(('127.0.0.1',8088))
server_fileobj.listen(5)
server_fileobj.setblocking(False) #设置socket的接口为非阻塞
sel.register(server_fileobj,selectors.EVENT_READ,accept) #相当于网select的读列表里append了一个文件句柄server_fileobj,并且绑定了一个回调函数accept

while True:
    events=sel.select() #检测所有的fileobj，是否有完成wait data的
    for sel_obj,mask in events:
        callback=sel_obj.data #callback=accpet
        callback(sel_obj.fileobj,mask) #accpet(server_fileobj,1)

#客户端
from socket import *
c=socket(AF_INET,SOCK_STREAM)
c.connect(('127.0.0.1',8088))

while True:
    msg=input('>>: ')
    if not msg:continue
    c.send(msg.encode('utf-8'))
    data=c.recv(1024)
    print(data.decode('utf-8'))