---
title: gitignore文件
sidebar_position: 4
---



# 一、为什么使用

> 在一些项目中，我们不想让本地仓库的所有文件都上传到远程仓库中，而是有选择的上传，比如：一些依赖文件（node_modules下的依赖）、bin 目录下的文件、测试文件等。一方面将一些依赖、测试文件都上传到远程传输量很大，另一方面，一些文件对于你这边是可用的，在另一个人那可能就不可用了，比如：本地配置文件。
>
> 为了解决上述问题，git 引入了 .gitignore 文件，使用该文件来选择性的上传文件。

# 二、使用规则

## 2.1 注释

> 注释使用 **#** 开头，后面跟注释内容。如下所示：

```shell
linuxy@linuxy:~/linuxGit$ cat .gitignore 
# this is .gitignore file.
out
*.exe
```

## 2.2 忽略文件和目录

### 2.2.1 忽略文件和目录

例如：**folderName :** 表示忽略 folderName 文件和 folderName 目录，会自动搜索多级目录，比如：`*/*/folderName`

来看一个简单的例子，本地仓库的目录结构如下所示：

```shell
linuxy@linuxy:~/linuxGit$ tree
├── folder
│   └── file1
└── src
    ├── folder
    └── utils
        └── folder

```

其中，.gitignore 文件内容如下所示：

```shell
linuxy@linuxy:~/linuxGit$ cat .gitignore 
# this is .gitignore file.
# 以下是忽略的文件
folder
```

故在本地仓库中，同名的 folder 目录、src/folder 文件、src/utils/folder 文件都会被忽略

### 2.2.2 仅忽略文件

仅忽略 folderName 文件，而不忽略 folderName 目录，其中，感叹号**“!”**表示反向操作。

```shell
folderName
!folderName/
```

来看一个简单的例子，本地仓库的目录结构如下所示：

```shell
linuxy@linuxy:~/linuxGit$ tree
├── folder
│   └── file1
└── src
    ├── folder
    └── utils
        └── folder
```

其中，.gitignore 文件内容如下所示：

```shell
linuxy@linuxy:~/linuxGit$ cat .gitignore 
# this is .gitignore file.
# 以下是忽略的文件
folder
!folder/
```

故在本地仓库中，src/folder 文件、src/utils/folder 文件会被忽略，而同名的 folder 目录不会被忽略。

### 2.2.3 仅忽略目录

语法如下：

```
folderName/
```

来看一个简单的例子，本地仓库的目录结构如下所示：

```shell
linuxy@linuxy:~/linuxGit$ tree
├── folder
│   └── file1
└── src
    ├── folder
    └── utils
        └── folder

```

其中，.gitignore 文件内容如下所示：

```shell
linuxy@linuxy:~/linuxGit$ cat .gitignore 
# this is .gitignore file.
# 以下是忽略的文件
folder/
```

故在本地仓库中，folder 目录会被忽略，而同名的 src/folder 文件和 src/utils/folder 文件不会被忽略。

# 三、通配符

## 3.1 *和?和[]

```
1）星号“*” ：匹配多个字符
2）问号“?”：匹配除 ‘/’外的任意一个字符
3）方括号“[xxxx]”：匹配多个列表中的字符
```

来看一个简单的例子，本地仓库的目录结构如下所示：

```shell
linuxy@linuxy:~/linuxGit$ tree
├── src
│   ├── add.c
│   ├── add.i
│   └── add.o
├── test.c
├── test.i
└── test.o
```

其中，.gitignore 文件内容如下所示：

```shell
linuxy@linuxy:~/linuxGit$ cat .gitignore 
# this is .gitignore file.
# 以下是忽略的文件
*.[io]
```

故在本地仓库中，test.i文件、test.o文件、src/add.o文件、src/add.i文件会被忽略，而 test.c文件和add.c 文件不会被忽略。注意：这里忽略的匹配模式是多级目录的。

## 3.2 !号

> ！表示反向操作，表示仅忽略 folderName 文件，而不忽略 folderName 目录：

```
folderName
!folderName/
```

## 3.3 **号

斜杠后紧跟两个连续的星号"**"，表示多级目录。

来看一个简单的例子，.gitignore文件的内容如下所示：

```shell
linuxy@linuxy:~/linuxGit$ cat .gitignore 
# this is .gitignore file.
# 以下是忽略的文件
src/**/file
```

可以表示忽略 src/folder1/file 、src/folder1/folder2/***/foldern/file 等。

# 四、常见问题

```
1）空行不匹配任何文件

2）git 跟踪文件，而不是目录

3）在 .gitignore 文件中，每行表示一种模式

4）如果本地仓库文件已被跟踪，那么即使在 .gitignore 中设置了忽略，也不起作用。
    其实真正的原因是.gitignore只能忽略那些尚未被被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。一个简单的解决方法就是先把本地缓存删除（改变成未track状态），然后再提交。
    git rm -r --cached .
    git add .
    git commit -m 'update .gitignore'
  
5）.gitignore 文件也会被上传的到远程仓库，所以，同一个仓库的人可以使用同一个.gitignore 文件。