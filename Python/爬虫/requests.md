---
title: requests
---


## 1.1介绍

> 使用requests可以模拟浏览器的请求，比起之前用到的urllib，requests模块的api更加便捷（本质就是封装了urllib3）
>
> requests库发送请求将网页内容下载下来以后，并不会执行js代码，这需要我们自己分析目标站点然后发起新的request请求

## 1.2 get请求

> 1. 没有请求体
> 2. 数据必须在1K之内！
> 3. GET请求数据会暴露在浏览器的地址栏中
> 4. GET请求携带参数如果含有中文或者特殊字符需要进行url编码
>
> **get请求分为带参数与不带参数，带参数的话，参数在url后面，例如`http://host:port/?id=1&name="西游记"`**

**请求地址中拼接GET参数**

```python
#在请求头内将自己伪装成浏览器，否则百度不会正常返回页面内容
import requests
response=requests.get('https://www.baidu.com/s?wd=python&pn=1',
                      headers={
                        'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36',
                      })
print(response.text)


#如果查询关键词是中文或者有其他特殊符号，则不得不进行url编码
from urllib.parse import urlencode
wd='egon老师'
encode_res=urlencode({'k':wd},encoding='utf-8')
keyword=encode_res.split('=')[1]
print(keyword)
# 然后拼接成url
url='https://www.baidu.com/s?wd=%s&pn=1' %keyword

response=requests.get(url,
                      headers={
                        'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36',
                      })
res1=response.text

```

**params拼接GET参数**

> 本质还是调用urlencode

```python
response=requests.get('https://www.baidu.com/s',
                      params={
                          'wd':wd,
                          'pn':pn
                      },
                      headers={
                        'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36',
                      })
res2=response.text
```

**带参数的GET请求->headers**

> 通常我们在发送请求时都需要带上请求头，请求头是将自身伪装成浏览器的关键，常见的有用的请求头如下
> Host
> Referer ：大型网站通常都会根据该参数判断请求的来源
> User-Agent ：客户端
> Cookie ：Cookie信息虽然包含在请求头里，但requests模块有单独的参数来处理他，headers={}内就不要放它了

```python
#添加headers(浏览器会识别请求头,不加可能会被拒绝访问,比如访问https://www.zhihu.com/explore)
import requests
response=requests.get('https://www.zhihu.com/explore')
response.status_code #500


#自己定制headers
headers={
    'User-Agent':'Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.76 Mobile Safari/537.36',

}
respone=requests.get('https://www.zhihu.com/explore',
                     headers=headers)
print(respone.status_code) #200
```

**带参数的GET请求->cookies**

```python
#登录github，然后从浏览器中获取cookies，以后就可以直接拿着cookie登录了，无需输入用户名密码
#用户名:egonlin 邮箱378533872@qq.com 密码lhf@123

import requests

Cookies={'user_session':'wGMHFJKgDcmRIVvcA14_Wrt_3xaUyJNsBnPbYzEL6L0bHcfc',
}

response=requests.get('https://github.com/settings/emails',
             cookies=Cookies) #github对请求头没有什么限制，我们无需定制user-agent，对于其他网站可能还需要定制


print('378533872@qq.com' in response.text) #True
```

## 1.3 post请求

> - 数据不会出现在地址栏中
> - 数据的大小没有上限
> - 有请求体
> - 请求体中如果含有中文或者特殊字符需要进行url编码
>
> **requests.post()用法与requests.get()完全一致，特殊的是requests.post()有一个data参数，用来存放请求体数据**

### 1.3.1 自动登录github

> 自己处理cookie信息

```python
'''
一 目标站点分析
    浏览器输入https://github.com/login
    然后输入错误的账号密码，抓包
    发现登录行为是post提交到：https://github.com/session
    而且请求头包含cookie
    而且请求体包含：
        commit:Sign in
        utf8:✓
        authenticity_token:lbI8IJCwGslZS8qJPnof5e7ZkCoSoMn6jmDTsL1r/m06NLyIbw7vCrpwrFAPzHMep3Tmf/TSJVoXWrvDZaVwxQ==
        login:egonlin
        password:123


二 流程分析
    先GET：https://github.com/login拿到初始cookie与authenticity_token
    返回POST：https://github.com/session， 带上初始cookie，带上请求体（authenticity_token，用户名，密码等）
    最后拿到登录cookie

    ps：如果密码时密文形式，则可以先输错账号，输对密码，然后到浏览器中拿到加密后的密码，github的密码是明文
'''

import requests
import re

#第一次请求
r1=requests.get('https://github.com/login')
r1_cookie=r1.cookies.get_dict() #拿到初始cookie(未被授权)
authenticity_token=re.findall(r'name="authenticity_token".*?value="(.*?)"',r1.text)[0] #从页面中拿到CSRF TOKEN

#第二次请求：带着初始cookie和TOKEN发送POST请求给登录页面，带上账号密码
data={
    'commit':'Sign in',
    'utf8':'✓',
    'authenticity_token':authenticity_token,
    'login':'317828332@qq.com',
    'password':'alex3714'
}
r2=requests.post('https://github.com/session',
             data=data,
             cookies=r1_cookie
             )


login_cookie=r2.cookies.get_dict()


#第三次请求：以后的登录，拿着login_cookie就可以,比如访问一些个人配置
r3=requests.get('https://github.com/settings/emails',
                cookies=login_cookie)

print('317828332@qq.com' in r3.text) #True

```

**requests.session()**

> 自动帮我们保存cookie信息

```python
import requests
import re

session=requests.session()
#第一次请求
r1=session.get('https://github.com/login')
authenticity_token=re.findall(r'name="authenticity_token".*?value="(.*?)"',r1.text)[0] #从页面中拿到CSRF TOKEN

#第二次请求
data={
    'commit':'Sign in',
    'utf8':'✓',
    'authenticity_token':authenticity_token,
    'login':'317828332@qq.com',
    'password':'alex3714'
}
r2=session.post('https://github.com/session',
             data=data,
             )

#第三次请求
r3=session.get('https://github.com/settings/emails')

print('317828332@qq.com' in r3.text) #True

```

### 1.3.2 传输json数据

```python
requests.post(url='xxxxxxxx',
              data={'xxx':'yyy'}) #没有指定请求头,#默认的请求头:application/x-www-form-urlencoed

#如果我们自定义请求头是application/json,并且用data传值, 则服务端取不到值
requests.post(url='',
              data={'name':'tom',},
              headers={
                  'content-type':'application/json'
              })


requests.post(url='',
              json={'name':'tom',},
              ) #默认的请求头:application/json
```

## 1.4 响应Response

### 1.4.1 response属性

```python
import requests
respone=requests.get('http://www.jianshu.com')
# respone属性
print(respone.text)
print(respone.content)

print(respone.status_code) # 响应状态码
print(respone.headers) # 响应头
print(respone.cookies) # cookies
print(respone.cookies.get_dict()) # 获取cookie的字典形式 {"id":1,'user':"meng"}
print(respone.cookies.items()) # [('id',1),('user','meng')] cookie字典形式转为列表

print(respone.url) # 请求的url
print(respone.history) # [放的是重定向之前的地址]
print(respone.encoding) # 响应的编码方式
  

#关闭：response.close()
from contextlib import closing
with closing(requests.get('xxx',stream=True)) as response:
    for line in response.iter_content():
    pass
```

### 1.4.2 编码问题

```python
#编码问题
import requests
response=requests.get('http://www.autohome.com/news')
# response.encoding='gbk' #汽车之家网站返回的页面内容为gb2312编码的，而requests的默认编码为ISO-8859-1，如果不设置成gbk则中文乱码
print(response.text)
```

### 1.4.3 获取二进制数据

```python
import requests

response=requests.get('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1509868306530&di=712e4ef3ab258b36e9f4b48e85a81c9d&imgtype=0&src=http%3A%2F%2Fc.hiphotos.baidu.com%2Fimage%2Fpic%2Fitem%2F11385343fbf2b211e1fb58a1c08065380dd78e0c.jpg')

with open('a.jpg','wb') as f:
    f.write(response.content)
```

```python
#stream参数:一点一点的取,比如下载视频时,如果视频100G,用response.content然后一下子写到文件中是不合理的

import requests

response=requests.get('https://gss3.baidu.com/6LZ0ej3k1Qd3ote6lo7D0j9wehsv/tieba-smallvideo-transcode/1767502_56ec685f9c7ec542eeaf6eac93a65dc7_6fe25cd1347c_3.mp4',
                      stream=True)

with open('b.mp4','wb') as f:
    for line in response.iter_content():
        f.write(line)
```

### 1.4.4 解析json

```python
#解析json
import requests
response=requests.get('http://httpbin.org/get')

import json
res1=json.loads(response.text) #太麻烦

res2=response.json() #直接获取json数据


print(res1 == res2) #True
```

### 1.4.5 Redirection and History

> allow_redirects参数
>
> 默认为True：响应头遇到location就跳转对应网址
>
> False：响应头即使遇到了location也不跳转

```python
import requests
import re

#第一次请求
r1=requests.get('https://github.com/login')
r1_cookie=r1.cookies.get_dict() #拿到初始cookie(未被授权)
authenticity_token=re.findall(r'name="authenticity_token".*?value="(.*?)"',r1.text)[0] #从页面中拿到CSRF TOKEN

#第二次请求：带着初始cookie和TOKEN发送POST请求给登录页面，带上账号密码
data={
    'commit':'Sign in',
    'utf8':'✓',
    'authenticity_token':authenticity_token,
    'login':'317828332@qq.com',
    'password':'alex3714'
}






#测试一：没有指定allow_redirects=False,则响应头中出现Location就跳转到新页面，r2代表新页面的response
r2=requests.post('https://github.com/session',
             data=data,
             cookies=r1_cookie
             )

print(r2.status_code) #200
print(r2.url) #看到的是跳转后的页面
print(r2.history) #看到的是跳转前的response
print(r2.history[0].text) #看到的是跳转前的response.text


#测试二：指定allow_redirects=False,则响应头中即便出现Location也不会跳转到新页面，r2代表的仍然是老页面的response
r2=requests.post('https://github.com/session',
             data=data,
             cookies=r1_cookie,
             allow_redirects=False
             )


print(r2.status_code) #302
print(r2.url) #看到的是跳转前的页面https://github.com/session
print(r2.history) #[]

利用github登录后跳转到主页面的例子来验证它
```

## 1.5 高级用法

### 1.5.1 SSL Cert Verification

```python
#证书验证(大部分网站都是https)
import requests
respone=requests.get('https://www.12306.cn') #如果是ssl请求,首先检查证书是否合法,不合法则报错,程序终端


#改进1:去掉报错,但是会报警告
import requests
respone=requests.get('https://www.12306.cn',verify=False) #不验证证书,报警告,返回200
print(respone.status_code)


#改进2:去掉报错,并且去掉警报信息
import requests
from requests.packages import urllib3
urllib3.disable_warnings() #关闭警告
respone=requests.get('https://www.12306.cn',verify=False)
print(respone.status_code)

#改进3:加上证书
#很多网站都是https,但是不用证书也可以访问,大多数情况都是可以携带也可以不携带证书
#知乎\百度等都是可带可不带
#有硬性要求的,则必须带，比如对于定向的用户,拿到证书后才有权限访问某个特定网站
import requests
respone=requests.get('https://www.12306.cn',
                     cert=('/path/server.crt',
                           '/path/key'))
print(respone.status_code)
```

### 1.5.2 使用代理

```python
#官网链接: http://docs.python-requests.org/en/master/user/advanced/#proxies

#代理设置:先发送请求给代理,然后由代理帮忙发送(封ip是常见的事情)
# 免费代理：https://www.kuaidaili.com/free/
# 高匿和透明代理：高匿：后端无论如何都拿不到你的IP，透明：后端可以拿到你的IP
# 如何拿到透明代理的IP，后端的x-Forwarded-For
import requests
proxies={
    'http':'http://egon:123@localhost:9743',#带用户名密码的代理,@符号前是用户名与密码
    'http':'http://localhost:9743',
    'https':'https://localhost:9743',
}
respone=requests.get('https://www.12306.cn',
                     proxies=proxies)

print(respone.status_code)



#支持socks代理,安装:pip install requests[socks]
import requests
proxies = {
    'http': 'socks5://user:pass@host:port',
    'https': 'socks5://user:pass@host:port'
}
respone=requests.get('https://www.12306.cn',
                     proxies=proxies)

print(respone.status_code)
```

### 1.5.3 超时设置

```python
#超时设置
#两种超时:float or tuple
#timeout=0.1 #代表接收数据的超时时间
#timeout=(0.1,0.2)#0.1代表链接超时  0.2代表接收数据的超时时间

import requests
respone=requests.get('https://www.baidu.com',
                     timeout=0.0001)
```

### 1.5.4 认证设置

```python
#官网链接：http://docs.python-requests.org/en/master/user/authentication/

#认证设置:登陆网站是,弹出一个框,要求你输入用户名密码（与alter很类似），此时是无法获取html的
# 但本质原理是拼接成请求头发送
#         r.headers['Authorization'] = _basic_auth_str(self.username, self.password)
# 一般的网站都不用默认的加密方式，都是自己写
# 那么我们就需要按照网站的加密方式，自己写一个类似于_basic_auth_str的方法
# 得到加密字符串后添加到请求头
#         r.headers['Authorization'] =func('.....')

#看一看默认的加密方式吧，通常网站都不会用默认的加密设置
import requests
from requests.auth import HTTPBasicAuth
r=requests.get('xxx',auth=HTTPBasicAuth('user','password'))
print(r.status_code)

#HTTPBasicAuth可以简写为如下格式
import requests
r=requests.get('xxx',auth=('user','password'))
print(r.status_code)
```

### 1.5.5 异常处理

```python
#异常处理
import requests
from requests.exceptions import * #可以查看requests.exceptions获取异常类型

try:
    r=requests.get('http://www.baidu.com',timeout=0.00001)
except ReadTimeout:
    print('===:')
# except ConnectionError: #网络不通
#     print('-----')
# except Timeout:
#     print('aaaaa')

except RequestException:
    print('Error')
```

### 1.5.6 上传文件

```python
import requests
files={'file':open('a.jpg','rb')}
respone=requests.post('http://httpbin.org/post',files=files)
print(respone.status_code)