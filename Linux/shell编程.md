## 命令的查找顺序

```shell
==> alias
　　==> Compound Commands(for if)
　　　　==> function
　　　　　　==> build_in
　　　　　　　　==> hash
　　　　　　　　　　==> $PATH
　　　　　　　　　　　　==> error: command not found
　　　　　　　　　　　　
　　　　　　　　　　　　
alias(别名)
    设置别名：ali as test="ls -l"
    取消别名：unalias test
   
　　　　　　　　
built-in commands(内置命令)
    cd pwd....


Compound Commands
    例如 for if while等
    参见 man bash


hash(使用的命令会缓存，方便下次查找)：
    [meng@mycentos /]$ hash
    命中	命令
       1	/usr/bin/ls
```

## 元字符

```shell
#1、~：家目录
[meng@mycentos ~]$ cd ~


#2、*：所有
[meng@mycentos ~]$ ls /opt/*


#3、``和$()：取命令结果，建议使用$()，它弥补了``嵌套无法识别的问题；${}:取变量的值
[meng@mycentos ~]$ x=$(ls)
[meng@mycentos ~]$ echo $x

#4、!：取反
[meng@mycentos ~]$

#5、 ;间隔的命令按顺序依次执行。
	例如：a=2;b=3;a=`expr $a \* $b`;echo $a

#6、 && 前后命令的执行存在“逻辑与关系”，只有&&前的命令执行成功，后面的才执行。
	例如： var1=5;var2=4;var3=3;echo $((var1>var2&&var2>var3))

#7、 || 前后命令的执行存在“逻辑或关系”，只有||前的命令执行失败，后面的才执行。
	例如： var1=5;var2=4;var3=3;echo $((var1<var2||var2>var3))

#8、 & 将命令挂在后台执行，不过shell一关就没了
	例如：python3 manage.py runserver &

#9、 nohup 将命令挂在后台执行，相比&，关掉shell，任然还在
	例如：nohup python3 manage.py runserver





0: 标准输入(stdin)
1: 标准输出(stdout)
2: 标准错误(stderr)


#10、 > 表示重定向（默认为标准输出重定向，也可以表示为，"1>"，会覆盖写）
[root@localhost ~]$ echo 'World' > test.txt


#11、2>&1 表示将标准错误重定向到标准输出里面，为什么加&？&起转移作用，不加&，就表示将标准错误重定向到文件名为1里面了

#12、1>&2 表示将标准输出重定向到标准错误里面，为什么加&？&起转移作用，不加&，就表示将标准输出重定向到文件名为1里面了

#13、 &> or >& 将标准输出与标准错误，输出到某个地方
[meng@centos ~]$ ls
honeypot  test.txt  tpp-honeypot  vulhub-master  web
[meng@centos ~]$ ls test.txt aaaa.txt &> tmp.txt
[meng@centos ~]$ cat tmp.txt
ls: 无法访问aaaa.txt: 没有那个文件或目录
test.txt

#14、 >> 表示内容追加写。
[root@localhost ~]$ echo 'World' >> test.txt


```

## 变量

### 创建

```shell
创建变量：<变量名>=<值>	（“=”前后不能有空格）
[root@aliyun ~]# name=tom or [root@aliyun ~]# name="tom"
# 注意：加不加引号都不所谓，都是按照字符串处理
```

### 赋值

```shell
shell允许用户从键盘中输入一个值给变量，如：
read var
例子：
   #!/bin/bash
　　echo “Enter the name of the customer.”
　　read name
　　echo “Enter the mobile phone number.”
　　read number
   echo “$name:$number” >> customerdata
# read命令可以提示用户输入数据。相当于python的input
# 另外在命令行直接赋值的比如  name="tom"
```

### 删除

```shell
unset VARNAME

例子：
[root@aliyun ~]# name=tom
[root@aliyun ~]# unset name
[root@aliyun ~]# echo $name

[root@aliyun ~]# 
```

### 全局与局部变量

```shell
[root@MiWiFi-R3-srv ~]# gender='male' #在爹这个位置定义一个局部变量gender
[root@MiWiFi-R3-srv ~]# export money=1000 #在爹这个位置定义一个全局变量money
[root@MiWiFi-R3-srv ~]#
[root@MiWiFi-R3-srv ~]#
[root@MiWiFi-R3-srv ~]# bash #切换到子bash
[root@MiWiFi-R3-srv ~]# echo $gender #在儿子这里看它爹的局部变量gender吗？，结果为空->看不到

[root@MiWiFi-R3-srv ~]# echo $money #在儿子这里看它爹的全局变量money，可以看到
1000
[root@MiWiFi-R3-srv ~]#
[root@MiWiFi-R3-srv ~]# export hobby='piao' #在儿子这里定义一个全局变量hobby
[root@MiWiFi-R3-srv ~]# exit #退出，进入爹的bash环境
exit
[root@MiWiFi-R3-srv ~]# echo $hobby #爹是看不到儿子的export的，儿子的儿子可以看到
# 这里有个问题，如果export传给后面的子shell后，在子shell里面改了值，父亲的值会变吗，其实不会变的，只是把值复制了过去，我自己的理解是一个子shell是一个线程，开线程数据是备份过去的，并不共享

# 记一个问题   执行bash和执行sh有啥区别
```

### 常用系统变量

```shell
HOME变量：存在当前用户主目录的位置
PATH变量：存储可执行程序的搜索路径
PS1和PS2变量：Linux下的bash shell有两级用户提示符PS1和PS2
SHLVL变量：存储当前工作的shell level
SHELL变量：存储了用户的缺省shell
```

### 内部变量

> 在shell执行之前定义；并根据shell的定义来使用这些变量，不能重复定义；由$和另一个字符组成

```shell
$#：位置参数的数量
$*：所有位置参数的内容
$?：命令执行后返回的状态
$$：当前进程的进程号
$!：后台运行的最后一个进行号
```

```shell
向脚本传递参数
$0：当前执行的程序名
$1	第一个参数
$2	第二个参数
$3	第二个参数
$N	第N个参数

例如：p1.sh 1 2 3 4....
    echo "Program name is $0"
    echo "There are totally $# parameters passed to this program"
    echo "The last is $?"
    echo "The parameters are $*"

```

## EOF

```shell
# 执行脚本的时候，需要往一个文件里自动输入N行内容。如果是少数的几行内容，还可以用echo追加方式，但如果是很多行，那么单纯用echo追加的方式就显得愚蠢之极了！这个时候，就可以使用EOF结合cat命令进行行内容的追加或者覆盖了。
下面就对EOF的用法进行梳理：
EOF是END Of File的缩写,表示自定义终止符.既然自定义,那么EOF就不是固定的,可以随意设置别名
EOF一般会配合cat能够多行文本输出.
其用法如下:
<<EOF        //开始
....
EOF            //结束

还可以自定义，比如自定义：
<<BBB        //开始
....
BBB              //结束

# 追加
cat >> tmp.txt <<EOF
echo "123"
ls -l 
...
EOF

# 覆盖
cat > tmp.txt <<EOF
echo "123"
ls -l 
...
EOF
```

## 别名

```shell
alias la='ls -a'
用户可以将这些设置放入一个系统环境配置文件中，使其长期生效。如将其加入到用户环境配置文件~/.bashrc中，则每次用户登录时都可以自动生效了
```

## 算数运算

> 所有计算器：$[] (()) $(()) expr bc bc -l
>
> - expr
>
> ```shell
> expr 4 + 5 （运算符前后有空格）
> expr支持的运算符可以是+，-，*，/。使用*时，前面加反斜杠；
> expr不支持小数，例如：expr 5 / 2 算下来是整数
> ```
>
> - $(()
>
> ```shell
> shell 提供了完整的算数运算能力，而且使用与c相同运算符与优先级。
> 语法：$((表达式1,表达式2…))
> 特点：
>   在双括号结构中，所有表达式可以像c语言一样，如：a++,b--等。
>   在双括号结构中，所有变量可以不加入“$”符号前缀。
>   支持多个表达式运算，各个表达式之间用“,”分开
>   双括号可以进行逻辑运算，四则运算
>   双括号结构扩展了for，while,if条件测试运算
> 例如：
> echo $((4+5))
> var =1
> echo $((var+1))
> 
> ```

## 测试

> test，[]， [[ ]] ，(( ))
>
> 命令执行后会返回到一个系统变量中 **$?**
> 如果$?值为0 表示命令执行成功 否则为失败
>
> **[]两边必须有空格**

### 逻辑测试

```shell
 	-eq 等于
  -ne 不等于
  -gt 大于
  -lt 小于
  -ge 大于等于
  -le 小于等于
  [root@MiWiFi-R3-srv ~]# [ 10000 -gt 250 ] 
```

### 串测试


| **选项**            | **值** | **含义**       |
| ------------------- | ------ | -------------- |
| string              | True   | 该串非空       |
| -z  string          | True   | 串的长度是零   |
| -n  string          | True   | 串的长度是非零 |
| string1  = string2  | True   | 两串相等       |
| string1  != string2 | True   | 两串不相等     |

```shell
#!/bin/bash
echo -n "Please enter your name:"
read name
if [ -z $name ]
then echo "You have not enter your name."
else echo "Your name is:$name."
fi

[root@MiWiFi-R3-srv ~]# var1='abc'
[root@MiWiFi-R3-srv ~]# var2='123'
[root@MiWiFi-R3-srv ~]# [ $var1 == $var2 ]
[root@MiWiFi-R3-srv ~]# echo $?
1
```

### 文件测试


| **命令格式**      | **返回值** | **状态**                 |
| ----------------- | ---------- | ------------------------ |
| test  –e filename | True       | 该文件存在               |
| test  –f filename | True       | 该文件存在并且是一般文件 |
| test  –d filename | True       | 该文件存在并且是目录文件 |
| test  –r filename | True       | 该文件存在并且可读的     |
| test  –w filename | True       | 该文件存在并且可写的     |
| test  –x filename | True       | 该文件存在并且可执行的   |
| test  –s filename | True       | 该文件存在并且不空       |
| test  –b filename | True       | 该文件存在并且是特殊块   |

```shell
#!/bin/bash
echo -n "Please enter a file name:"
read file_name
if [ -d $file_name ]
then echo "$file_name is a directory file."
elif [ -b $file_name ]
then echo "$file_name is a block device file."
elif [ -c $file_name ]
then echo "$file_name is a character device file."
elif [ -L $file_name ]
then echo "$file_name is a link file."
elif [ -f $file_name ]
then echo "$file_name is an ordinary file."
else echo "$file_name is other type file."
fi

```

## 条件语句

### 选择语句

```shell
语法1						语法2
  if[条件]			if[条件]
  then 命令			then 命令
  else 命令		  elif 条件
  fi					 then 命令
  						 ...
  						 else 命令
  						 fi
  						 
例子：test.sh
  #!/bin/zsh
  echo “Enter a number ”
  read num1
  if [ $num1 -ge 50 ]
  then echo “The number is greater than or equal to 50.”
  else echo “The number is less than 50.”
  fi
  
	#有多个条件的情况下，可以使用：
		-a 表示and选项
		-o 表示or选项
	例如：
    echo “Enter a number.”
    read no
    if [ $no –ge 1 –a $no –le 100 ]
    then echo “The number is between 1 and 100”
    else echo “The number is not between 1 and 100”
    fi

```

### 分支语句

> **case...esac**，相当于C语言的switch...case

```shell
#!/bin/bash
read  -p  "press a key ,then press return :"   key
case $key in 
[a-zA-Z])
	echo "It's a letter."
  echo "$key";;
[0-9]) 
	echo "It's a digit.";;
*)
	echo "It's function keys、Spacebar or other keys."
esac

```

### 循环语句

**while**

> 语法：
>
> 　　while<条件>
>
> 　　do
>
> 　　 <命令>
>
> 　　done
>
> 只有条件为真时，才执行do和done之间的命令。
>
> 该语句支持while true命令，创建一个无限循环。

```shell
#!/bin/bash
read  -p  "Please enter a file name(n used to exit):"  file_name
while [ $file_name != 'n' ]
do
	echo "The $file_name's content is :"
	cat  $file_name
	read  -p  "Please enter a file name(n used to exit):"  file_name
done

```

**for循环**

> **语法1：**
>
> 　　for 变量名 in <值表>
>
> 　　do
>
> 　　 …
>
> 　　 …
>
> 　　done
>
> **语法2：**
>
> 　　for((表达式1；表达式2；表达式3))
>
> 　　do
>
> 　　 …
>
> 　　 …
>
> 　　done

```shell
输出1到20之间的奇数
 	#!/bin/bash
	for i in `seq 1 2 20`
	do
    		echo -n "$i " 
	done
	echo
```

**until循环**

> uuntil循环语句的求值模式与while循环相反。
>
> uuntil循环一直执行直到求值的条件为真才退出。
>
> 两个语句仅仅在循环实现的求值的条件上不同

```shell
#!/bin/bash
	a=1
	until [ $a -gt 10 ]
	do
 		  echo -n "$a " 
  		 ((a=a+1))
	done
	echo
```

**break和continue**

> ubreak和contine命令用在循环语句中：
>
> break命令表示终止循环。
>
> continue命令的作用是强制结束当前循环而转入下一个新的循环。

```shell
a=0
while [ $a -le 10 ]
do
    ((a=a+1))
    if [ $a -eq 5 ]
    then 	continue
    elif [ $a -eq 8 ]
    then 	break
    fi
    echo -n  "$a "
done
echo