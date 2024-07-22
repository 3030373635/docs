> https://juejin.cn/post/6844904034181070861
>
> http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html

# 一、cookie

> 服务端生成存储在客户端的一组组键值对

# 二、session

> 服务端生成存储在服务端的一组组键值对

```python
客户端cookie中：
	{session_id:随机字符串}

服务端session中：
	{随机字符串:{"id":xxxx,useranme":xxxxx}}


# 流程：客户端带着随机字符串访问服务端，服务端拿到随机字符串后去匹配，看看是否可以根据随机字符串拿到东西
```

# 三、token

## 3.1 传统token

> 用户登录成功后，服务端生成一个随机token给用户，并且在服务端(数据库或缓存)中保存一份token，以后用户再来访问时需携带token，服务端接收到token之后，去数据库或缓存中进行校验token的是否超时、是否合法。

## 3.2 jwt

> jwt（JSON Web Tokens），是一种开发的行业标准 [RFC 7519](https://tools.ietf.org/html/rfc7519) ，用于安全的表示双方之间的声明。目前，jwt广泛应用在系统的用户认证方面，特别是现在前后端分离项目。
>
> 用户登录成功后，服务端通过jwt生成一个随机token给用户（服务端无需保留token），以后用户再来访问时需携带token，服务端接收到token之后，通过jwt对token进行校验是否超时、是否合法。

### 3.2.1 流程

<img src="http://cos.liuqm.cc/1665119190.png">

### 3.2.2 token组成

jwt的生成token格式如下，即：由 `.` 连接的三段字符串组成。

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

- 第一段header部分，固定包含算法和token类型，对此json进行base64加密，这就是token的第一段。

```python
{
  "alg": "HS256", # 第三段加密使用的算法
  "typ": "JWT"
}
```

- 第二段payload部分，包含一些自定义数据，对此json进行base64加密，这就是token的第二段。

```python
{
  "id": "1234567890",
  "name": "John Doe",
  "exp": 1516239022 # 超时时间
  ...
}
```

- 第三段signature部分，把前两段的base64后的密文通过`.`拼接起来，然后对其进行`HS256`加密，再然后对`HS256`密文进行base64加密，最终得到token的第三段。

```
base64url(
    HMACSHA256(
      base64UrlEncode(header) + "." + base64UrlEncode(payload),
      your-256-bit-secret (秘钥加盐)
    )
)
```

**最后将三段字符串通过 `. `拼接起来就生成了jwt的token。**

**注意：base64加密是先做base64加密，然后再将 `-` 替代 `+` 及 `_`  替代 `/` 。**

### 3.2.3 代码实现

> 基于Python的pyjwt模块创建jwt的token。

**安装**

```
pip3 install pyjwt
```

**实现**

```python
import jwt
import datetime
from jwt import exceptions

SALT = 'iv%x6xo7l7_u9bf_u!9#g#m*)*=ej@bek5)(@u3kh*72+unjv='

def create_token():
    # 构造header
    headers = {
        'typ': 'jwt',
        'alg': 'HS256'
    }
    # 构造payload
    payload = {
        'user_id': 1, # 自定义用户ID
        'username': 'wupeiqi', # 自定义用户名
        'exp': datetime.datetime.utcnow() + datetime.timedelta(minutes=5) # 超时时间
    }

    result = jwt.encode(payload=payload, key=SALT, algorithm="HS256", headers=headers).decode('utf-8')
    return result

if __name__ == '__main__':
    token = create_token()
    print(token)
```

### 3.2.4 原理检验

> 一般在登录成功后，把jwt生成的token返回给用户，以后用户再次访问时候需要携带token，此时jwt需要对token进行`超时`及`合法性`校验。

第一步：将token分割成 `header_segment`、`payload_segment`、`crypto_segment` 三部分，对header_segment、payload_segment、 crypto_segment，base64url解密

第二步：判断是否超时，超时就终止了

- 提取exp字段判断

第三步：当未超时时，判断第三部分的合法性

- 从第一段明文中获取加密算法，默认：`HS256`
- 将解密后的前两段密文拼接，即：`signing_input`
- 使用`HS256`+盐 对`signing_input` 进行加密，将得到的结果和第三段进行比较。

```python
import jwt
import datetime
from jwt import exceptions
SALT = 'iv%x6xo7l7_u9bf_u!9#g#m*)*=ej@bek5)(@u3kh*72+unjv='
def get_payload(token):
    try:
        # True表示，第二段时间校验与第三段合法性校验
        payload = jwt.decode(token, SALT, True)
        return payload
    except exceptions.ExpiredSignatureError:
        print('token失效')
    except jwt.DecodeError:
        print('token认证失败')
    except jwt.InvalidTokenError:
        print('非法的token')

if __name__ == '__main__':
    token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NzM1NTU1NzksInVzZXJuYW1lIjoid3VwZWlxaSIsInVzZXJfaWQiOjF9.xj-7qSts6Yg5Ui55-aUOHJS4KSaeLq5weXMui2IIEJU"
    payload = get_payload(token)