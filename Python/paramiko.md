---
title: paramiko
---


# 一、简介

> 像xshell一样，paramiko提供接口远程服务器
>
> 环境：
>
> python解释器 3.9
>
> paramiko 2.7.2

# 二、连接

## 2.1 用户名密码连接

```python
import paramiko

# 创建SSH对象
ssh = paramiko.SSHClient()
# 允许连接不在know_hosts文件中的主机
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
# 连接服务器
ssh.connect(hostname='192.168.31.178', port=22, username='meng', password='lqm576576')

# 执行命令
stdin, stdout, stderr = ssh.exec_command('ls')

# 获取命令结果
result = stdout.read()

"""
交互
stdin, stdout, stderr = ssh.exec_command('rm -i test.py')
-i 删除的时候询问
stdin.write("y") 这条指令就是交互用的
"""


print(result.decode("utf-8"))

# 关闭连接
ssh.close()

```

## 2.2 公钥密钥连接

```python
import paramiko

private_key = paramiko.RSAKey.from_private_key_file('/Users/meng/.ssh/id_rsa')

# 创建SSH对象
ssh = paramiko.SSHClient()
# 允许连接不在know_hosts文件中的主机
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
# 连接服务器
ssh.connect(hostname='192.168.31.178', port=22, username='meng', pkey=private_key)

# 执行命令
stdin, stdout, stderr = ssh.exec_command('ls')
# 获取命令结果
result = stdout.read()
print(result.decode("utf-8"))

# 关闭连接
ssh.close()
```

# 三、上传下载

## 3.1 用户名密码上传下载

```python
import paramiko
 
transport = paramiko.Transport(('hostname',22))
transport.connect(username='wupeiqi',password='123')
 
sftp = paramiko.SFTPClient.from_transport(transport)
# 将location.py 上传至服务器 /tmp/test.py
sftp.put('/tmp/location.py', '/tmp/test.py')
# 将remove_path 下载到本地 local_path
sftp.get('remove_path', 'local_path')
```

## 3.2 公钥密钥上传下载

```python
import paramiko
 
private_key = paramiko.RSAKey.from_private_key_file('/home/auto/.ssh/id_rsa')
 
transport = paramiko.Transport(('hostname', 22))
transport.connect(username='', pkey=private_key )
 
sftp = paramiko.SFTPClient.from_transport(transport)
# 将location.py 上传至服务器 /tmp/test.py
sftp.put('/tmp/location.py', '/tmp/test.py')
# 将remove_path 下载到本地 local_path
sftp.get('remove_path', 'local_path')
 
transport.close()
```

# 四、封装类

## 4.1 账号密码版

```python
import paramiko
class SSHProxy:
    def __init__(self,host_name,port,user_name,password):
        self.host_name = host_name
        self.port = port
        self.user_name = user_name
        self.password = password
        self.transport = None
    def open(self):
        self.transport = paramiko.Transport((self.host_name,self.port))
        self.transport.connect(username=self.user_name,password=self.password)

    def command(self,cmd):
        ssh = paramiko.SSHClient()
        ssh._transport = self.transport
        stdin,stdout,stderr = ssh.exec_command(cmd)
        result = stdout.read()
        return result

    def upload(self,local_path,remote_path):
        sftp = paramiko.SFTPClient.from_transport(self.transport)
        sftp.put(local_path, remote_path)

    def close(self):
        self.transport.close()

    def __enter__(self):
        self.open()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()

if __name__ == '__main__':
    with SSHProxy("lqmblog.com",22,"meng","lqm576576") as f:
        print(f.command("ls").decode("utf-8"))
        f.upload("test1.txt","/home/meng/test1.txt")
        print(f.command("ls").decode("utf-8"))


"""
这里又长见识了
with上下文管理的使用机制
1）with运行时
        例如：with obj as f
        调用__enter__函数
        这个f就是__enter__的返回值
2）with结束时
        就会执行__exit__函数
"""
```

## 4.2 密匙版

```python
import paramiko
from io import StringIO

class SSHProxy(object):

    def __init__(self, hostname, port, username, private_key_string):
        self.hostname = hostname
        self.port = port
        self.username = username

        self.private_key = paramiko.RSAKey(file_obj=StringIO(private_key_string))

        self.transport = None

    def open(self):
        self.transport = paramiko.Transport(self.hostname, self.port)
        self.transport.connect(username=self.username, pkey=self.private_key)

    def close(self):
        self.transport.close()

    def command(self, cmd):
        ssh = paramiko.SSHClient()
        ssh._transport = self.transport
        stdin, stdout, stderr = ssh.exec_command(cmd)
        result = stdout.read()
        # ssh.close()
        return result

    def upload(self, local_path, remote_path):
        sftp = paramiko.SFTPClient.from_transport(self.transport)
        sftp.put(local_path, remote_path)
        sftp.close()

    def __enter__(self):
        self.open()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()


if __name__ == '__main__':
    with SSHProxy('10.211.55.25', 22, 'root', 'asdasdfasdfasdfasdfasdf') as proxy:
        proxy.upload('xx','xx')
        proxy.command('ifconfig')
        proxy.command('ifconfig')
        proxy.upload('xx', 'xx')
    with SSHProxy('10.211.55.26', 22, 'root', 'aasdfasdfa') as proxy:
        proxy.upload('xx','xx')
        proxy.command('ifconfig')
        proxy.command('ifconfig')
        proxy.upload('xx', 'xx')