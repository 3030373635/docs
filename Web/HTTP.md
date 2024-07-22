## 1.1 上网流程

```tex
用户从打开浏览器输入网址到看到页面经历了什么？

1.首先必须web服务器运行

2.客户端运行浏览器软件

3.用户输入http://www.sina.com.cn/

4.客户端浏览器处理http://www.sina.com.cn/,发起查询本地DNS操作，将www.sina.com.cn->202.103.0.33

5.客户端浏览器发送http请求http://202.103.0.33:80/index.html   (注意：80是web服务器的默认端口,index.html是默认的请求的资源)

6.服务端web服务收到该http的request请求头，从请求头中获取客户端的方法GET/POST.../index.html这个路径，及客户端请求的其他相关信息

7.服务端web服务根据取得的信息,回复respone响应头

响应头中包含：

响应代码：200表示成功，3xx表示重定向，4xx表示客户端发送的请求有错误，5xx表示服务器端处理时发生了错误；

响应类型：由Content-Type指定；

以及其他相关的Header；

通常服务器的HTTP响应会携带内容，也就是有一个Body（响应体），包含响应的内容，网页的HTML源码就在Body中，压缩后返回给客户端。

8.客户端浏览器收到服务端发来的数据,解压后解析html内容，用户就看到网页内容了

9.html内可能嵌套其他的链接，比方说图片、视频、javascript脚本，flash等，客户端浏览器会继续发起http请求来获取它们。这样来自图片和视频的压力就被分散到各个服务器，一个站点由无数个站点相互连接起来，就形成了World Wide Web，简称WWW。

综上，其实就是一次http请求-响应的流程
```

## 1.2 http协议特性

> - HTTP叫超文本传输协议，基于请求/响应模式的！
> - HTTP是无状态协议，身不对请求和响应之间的通信状态进行保存。也就是说在HTTP这个级别,协议对于发送过的请求或响应都不做持久化处理
> - 可是,随着Web的不断发展,因无状态而导致业务处理变得棘手 的情况增多了。比如,用户登录到一家购物网站,即使他跳转到该站的 其他页面后,也需要能继续保持登录状态。针对这个实例,网站为了能 够掌握是谁送出的请求,需要保存用户的状态。HTTP/1.1虽然是无状态协议,但为了实现期望的保持状态功能, 于是引入了**Cookie技术。有了Cookie再用HTTP协议通信,就可以管 理状态了。有关Cookie的详细内容稍后讲解**

## 1.3 请求协议

```python
请求协议的格式如下：

请求首行；  // 请求方式 请求路径 协议和版本，例如：GET /index.html HTTP/1.1
请求头信息；// 请求头名称:请求头内容，即为key:value格式，例如：Host:localhost
空行；     // 用来与请求体分隔开
请求体。   // GET没有请求体，只有POST有请求体。
# 浏览器发送给服务器的内容就这个格式的，如果不是这个格式服务器将无法解读！在HTTP协议中，请求有很多请求方法，其中最为常用的就是GET和POST。不同的请求方法之间的区别，后面会一点一点的介绍。
```

<img src="http://oss.liuqm.cc/article/%E5%8D%8F%E8%AE%AE/http/%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87.png" style="width:100%">

<img src="http://oss.liuqm.cc/article/%E5%8D%8F%E8%AE%AE/http/download.jpg">

### 1.3.1 GET请求

> 1. 没有请求体
> 2. 数据必须在1K之内！(因为浏览器对URL的长度有限制）
> 3. GET请求数据会暴露在浏览器的地址栏中
> 4. GET请求携带参数如果含有中文或者特殊字符需要进行url编码

```
GET / HTTP/1.1
Host: www.sina.com.cn
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8
Cookie: SINAGLOBAL=223.71.229.3_1484011627.259786; Apache=223.71.229.3_1484011627.259788
```

### 1.3.2 POST请求

> - 数据不会出现在地址栏中
> - 数据的大小没有上限
> - 有请求体
> - 请求体如果含有中文或者特殊字符需要进行url编码

```python
POST /?name=lqz&age=18 HTTP/1.1
Host: 127.0.0.1:8008
Connection: keep-alive
Content-Length: 21
Cache-Control: max-age=0
Origin: http://127.0.0.1:8008
nUpgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Referer: http://127.0.0.1:8008/?name=lqz&age=1
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie: csrftoken=7xx6BxQDJ6KB0PM7qS8uTA892ACtooNbnnF4LDwlYk1Y7S7nTS81FBqwruizHsxF

name=lqz&password=123

"""
Content-Type的值有以下几种: 

	application/x-www-form-urlencoded：表单提交，会使用url格式编码数据；url编码的数据都是以“%”为前缀，后面跟随两位的16进制。格式例子：name=tom&age=1，url编码还像是中文才会编码

	application/json：格式例子：'{"name":"tom","age":1}'
"""
```

## 1.4 响应协议

```
响应首行；
响应头信息；
空行；
响应体。
```

<img src="http://oss.liuqm.cc/article/%E5%8D%8F%E8%AE%AE/http/%E5%93%8D%E5%BA%94%E6%8A%A5%E6%96%87.png" style="width:100%">

<img src="http://oss.liuqm.cc/article/%E5%8D%8F%E8%AE%AE/http/download-1.jpg">

## 1.5 状态码


| 状态码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | （成功）服务器已成功处理了请求。                             |
| 201    | （已创建）请求成功并且服务器创建了新的资源。                 |
| 204    | （无内容）服务器成功处理了请求，但没有返回任何内容。         |
| 301    | （永久移动）请求的网页已永久移动到新位置。                   |
| 302    | （临时移动）服务器目前从不同的位置响应请求。                 |
| 400    | （错误请求）服务器不理解请求的语法。                         |
| 401    | （未授权）请求要求身份验证。                                 |
| 403    | （禁止）无权限, 服务器拒绝请求。                             |
| 404    | （未找到） 服务器找不到请求的资源                            |
| 408    | （超时） 请求超时                                            |
| 422    | （验证错误） 请求参数未通过验证                              |
| 429    | （被限制）请求次数过多                                       |
| 500    | （服务器内部错误） 服务器遇到错误，无法完成请求。            |
| 501    | （尚未实施） 服务器不具备完成请求的功能。                    |
| 502    | （错误网关） 服务器作为网关或代理，从上游服务器收到无效响应。 |
| 503    | （服务不可用） 服务器目前无法使用（由于超载或停机维护）。 通常，这只是暂时状态。 |
| 504    | （网关超时） 服务器作为网关或代理，但是没有及时从上游服务器收到请求。 |
| 505    | （HTTP 版本不受支持） 服务器不支持请求中所用的 HTTP 协议版本。 |

## 1.6 为什么要url编码

> 个人理解：
>
> 观点一：url只支持英文，如果含有中文需要编码
>
> 观点二：url中含有特殊字符，不编码会造成数据的混乱，如下

```
我们都知道Http协议中参数的传输是"key=value"这种简直对形式的，如果要传多个参数就需要用“&”符号对键值对进行分割。如"?name1=value1&name2=value2"，这样在服务端在收到这种字符串的时候，会用“&”分割出每一个参数，然后再用“=”来分割出参数值。

 

针对“name1=value1&name2=value2”我们来说一下客户端到服务端的概念上解析过程: 
  上述字符串在计算机中用ASCII吗表示为： 
  6E616D6531 3D 76616C756531 26 6E616D6532 3D 76616C756532。 
   6E616D6531：name1 
   3D：= 
   76616C756531：value1 
   26：&
   6E616D6532：name2 
   3D：= 
   76616C756532：value2 
   服务端在接收到该数据后就可以遍历该字节流，首先一个字节一个字节的吃，当吃到3D这字节后，服务端就知道前面吃得字节表示一个key，再想后吃，如果遇到26，说明从刚才吃的3D到26子节之间的是上一个key的value，以此类推就可以解析出客户端传过来的参数。

   现在有这样一个问题，如果我的参数值中就包含=或&这种特殊字符的时候该怎么办。 
比如说“name1=value1”,其中value1的值是“va&lu=e1”字符串，那么实际在传输过程中就会变成这样“name1=va&lu=e1”。我们的本意是就只有一个键值对，但是服务端会解析成两个键值对，这样就产生了奇异。

如何解决上述问题带来的歧义呢？解决的办法就是对参数进行URL编码 
   URL编码只是简单的在特殊字符的各个字节前加上%，例如，我们对上述会产生奇异的字符进行URL编码后结果：“name1=va%26lu%3De1”，这样服务端会把紧跟在“%”后的字节当成普通的字节，就是不会把它当成各个参数或键值对的分隔符。