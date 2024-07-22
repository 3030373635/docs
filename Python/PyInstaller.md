> 当我们需要把项目给别人运行时，别人需要安装python环境，于是我们可以打包项目为可执行程序

# 常用方式

```python
# 安装：pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pyinstaller
pyinstaller -D start.py(入口文件)
pyinstaller -F start.py(入口文件)
```

```
# 当项目依赖非py文件时，需要使用spec文件标记需要引入哪些资源
# 创建spec文件
方式一：pyi-makespec -F demo01.py		pyinstaller demo01.spec
方式二：pyinstaller -F demo01.py		pyinstaller demo01.spec


*.spec常用字段
1. hiddenimports：pyinstaller无法引入动态引入的模块，例如程序中使用了importlib来动态引入模块，
hiddenimports = ["server.core","server.utils"]

2. binaries：当程序引入了二进制文件，需要标记
hiddenimports = [("busybox-12",".")]

3. datas：其他文件，例如静态文件
datas = [("statuc","static")]


Nots：python2 单文件只有动态引入资源，不可以打包成真正的单文件


单文件
	- 动态资源（如果有外部资源，需要把可执行文件和外部资源放在一块）

		Nots：打包单文件，本质其实是打包成一个压缩包，运行可执行文件时，会解压在临时目录，如果此时用else里面的代码，PROJ_PATH会得到临时目录的路径，所以需要区分一下。

		if getattr(sys, "frozen", False):
		    PROJ_PATH = PROJ_PATH = os.path.normpath(os.path.join(
		        sys.executable, # 可执行文件所在路径
		        # os.path.abspath(sys.argv[0]) # 不建议使用，如果执行文件时，设置了工作目录可能会导致sys.argv计算出错
		        os.pardir,  # 上一级目录(..)
		    ))
		else:
		    PROJ_PATH = os.path.normpath(os.path.join(
		        os.path.abspath(__file__),  # 脚本所在路径
		        os.pardir,  # 上一级目录(..)
		    ))


	- 静态资源（外部资源和py代码全打包在一个可执行文件里面）

		PROJ_PATH = os.path.normpath(os.path.join(
		        os.path.abspath(__file__),
		        os.pardir,  # 上一级目录(..)
		))

		不推荐，因为外部资源更新，需要重新打包
		程序引入资源在打包时使用--add-data命令或者在*.spec->Analysis->datas里面填写，例如，需要将static目录加进去，这样写datas = [("static","static")]
		如果要将文件加进去，这样写datas = [("aaa.txt",".")]

		Nots1：打包单文件，本质其实是打包成一个压缩包，运行那个单文件的时候，会解压在临时目录，PROJ_PATH会得到临时目录的路径，在此之前，我们已经手动将外部资源给加到了临时目录下，所以，可以直接引用
		Nots2：因为是在临时目录运行的代码，所以程序如果有输出文件，都会在临时目录，如果有输出文件，不建议使用该方式打包。

多文件
.......
```

# 通用参数


| 参数名           | 描述                                                     | 说明                                                         |
| :--------------- | :------------------------------------------------------- | :----------------------------------------------------------- |
| -h               | 显示帮助                                                 | 无                                                           |
| -v               | 显示版本号                                               | 无                                                           |
| –distpath        | 生成文件放在哪里                                         | 默认：当前目录的dist文件夹内                                 |
| –workpath        | 生成过程中的中间文件放在哪里                             | 默认：当前目录的build文件夹内                                |
| -y               | 如果dist文件夹内已经存在生成文件，则不询问用户，直接覆盖 | 默认：询问是否覆盖                                           |
| –upx-dir UPX_DIR | 指定upx工具的目录                                        | 默认：execution path                                         |
| -a               | 不包含unicode支持                                        | 默认：尽可能支持unicode                                      |
| –clean           | 在本次编译开始时，清空上一次编译生成的各种文件           | 默认：不清除                                                 |
| –log-level LEVEL | 控制编译时pyi打印的信息                                  | 一共有6个等级，由低到高分别为TRACE DEBUG INFO(默认) WARN ERROR CRITICAL。也就是默认清空下，不打印TRACE和DEBUG信息 |

# 与生成结果有关的参数


| 参数名    | 描述                          | 说明                                                         |
| :-------- | :---------------------------- | :----------------------------------------------------------- |
| -D        | 生成one-folder的程序（默认）  | 生成结果是一个目录，各种第三方依赖、资源和exe同时存储在该目录 |
| -F        | 生成one-file的程序            | 生成结果是一个exe文件，所有的第三方依赖、资源和代码均被打包进该exe内 |
| –specpath | 指定.spec文件的存储路径       | 默认：当前目录                                               |
| -n        | 生成的.exe文件和.spec的文件名 | 默认：用户脚本的名称，即main.py和main.spec                   |

# 指定打包哪些资源、代码


| 参数名                | 描述                                       | 说明                                                         |
| :-------------------- | :----------------------------------------- | :----------------------------------------------------------- |
| –add-data             | 打包额外资源                               | 用法：pyinstaller[main.py](http://main.py/) --add-data=src;dest。windows以;分割，linux以:分割 |
| –add-binary           | 打包额外的代码                             | 用法：同–add-data。与–add-data不同的是，用binary添加的文件，pyi会分析它引用的文件并把它们一同添加进来 |
| -p                    | 指定额外的import路径，类似于使用PYTHONPATH | 参见PYTHONPATH                                               |
| –hidden-import        | 打包额外py库                               | pyi在分析过程中，有些import没有正确分析出来，运行时会报import error，这时可以使用该参数 |
| –additional-hooks-dir | 指定用户的hook目录                         | hook用法参见其他，系统hook在PyInstaller\hooks目录下          |
| –runtime-hook         | 指定用户runtime-hook                       | 如果设置了此参数，则runtime-hook会在运行main.py之前被运行    |
| –exclude-module       | 需要排除的module                           | pyi会分析出很多相互关联的库，但是某些库对用户来说是没用的，可以用这个参数排除这些库，有助于减少生成文件的大小 |
| –key                  | pyi会存储字节码，指定加密字节码的key       | 16位的字符串                                                 |

# 生成参数


| 参数名 | 描述                                                 | 说明                              |
| :----- | :--------------------------------------------------- | :-------------------------------- |
| -d     | 执行生成的main.exe时，会输出pyi的一些log，有助于查错 | 默认：不输出pyi的log              |
| -s     | 优化符号表                                           | 原文明确表示不建议在windows上使用 |
| –noupx | 强制不使用upx                                        | 默认：尽可能使用。                |

# 其他


| 参数名          | 描述                 | 说明                   |
| :-------------- | :------------------- | :--------------------- |
| –runtime-tmpdir | 指定运行时的临时目录 | 默认：使用系统临时目录 |

# Windows和Mac特有的参数


| 参数名 | 描述               | 说明                                                |
| :----- | :----------------- | :-------------------------------------------------- |
| -c     | 显示命令行窗口     | 与-w相反，默认含有此参数                            |
| -w     | 不显示命令行窗口   | 编写GUI程序时使用此参数有用。                       |
| -i     | 为main.exe指定图标 | pyinstaller -i beauty.ico[main.py](http://main.py/) |

# Windows特有的参数


| 参数名         | 描述             | 说明                               |
| :------------- | :--------------- | :--------------------------------- |
| –version-file  | 添加版本信息文件 | pyinstaller --version-file ver.txt |
| -m, --manifest | 添加manifest文件 | pyinstaller -m main.manifest       |
| -r RESOURCE    | 请参考原文       |                                    |
| –uac-admin     | 请参考原文       |                                    |
| –uac-uiaccess  | 请参考原文       |                                    |