---
title: PKI
---

# 一、概述

名称：puklic key infra....，公钥基础设施，是一个**安全框架和体系结构**，用于**建立、管理和分发公钥**，并提供**安全和可信赖的通信和数据传输**。它是基于**非对称加密算法**的一种广泛应用的加密技术框架，通过**数字证书**和**证书颁发机构**（Certificate Authority, CA）来实现身份验证和信任。

作用：通过加密技术和数字签名保证信息安全

组成：

- 公钥加密技术
- 数字证书
- CA（数字证书颁发机构，公正公钥的合法性）、RA（数字颁发机构的分局，公正公钥的合法性）

信息安全三要素：

- 机密性
- 完整性
- 身份验证

> 如何解决机密性：将数据通过私钥加密，即可解决机密性，没有解决完整性与身份验证
>
> 如何解决完整性：私钥加密数据后，对密文进行hash，生成的串叫做摘要，将摘要附在加密数据后面然后发出去，对方使用公钥解密拿到数据后，对密文部分hash后，跟传过来的摘要作对比，方可知道数据是否完整，但没有解决身份验证
>
> 如何解决身份验证：再用私钥加密摘要（**此时叫做数字签名**），对方收到后，先用公钥对签名部分解密，如果能解开，表示身份正确
>
> 但是有个问题：双方交流前，需要交换公钥，但是交换公钥是明文交换，并且可能会被中间人攻击。如何解决？通过数字证书解决

## 1.1 公钥加密技术

作用：实现对信息加密、数字签名等安全保障

**常用对称加密算法：**

> 加密和解密时使用的是同一个秘钥
>
> 优点：速度快
>
> 缺点：双方都需要保存秘钥，传递秘钥过程中间容易被中间人攻击获取

- DES
- AES

**常用非对称加密算法：**

> 而非对称加密算法需要两个秘钥来进行加密和解密，分别是公开密钥（public key，简称公钥）和私有密钥（private key，简称私钥）。其实任何一个用来加密的话，要解密只能用另外一个
>
> 优点：安全性高
>
> 缺点：速度慢

- RSA

- DHE
- DSA
- ECDHE

**常用hash算法（一般验证数据的完整性）**

- MD5
- SHA

## 1.2 数字证书

> 用于保证公钥的合法性，遵循x.509标准
>
> 包含信息：
>
> - 使用者的公钥值
> - 使用者标识信息（如名称和电子邮件地址）
> - 有效期（证书的有效时间）
> - 颁发者标识信息
> - 颁发者的数字签名

## 1.3 应用领域

    - SSL/HTTPS
    - IPsecVPV
    - 部分远程访问VPN

