---
title: 文件IO
---


# 一、标准I/O

> 标准IO由ANSI C标准定义
>
> 就是一组定义好的来进行IO的函数
>
> 内部通过缓冲机制减少系统调用，其实就是会，读取一部分数据放在缓冲区，拿数据先在缓冲区拿，而不像操作系统发请求，所以效率高
>
> 读写内容都是先存在缓冲区。
>
> 系统调用：使用os提供的接口来访问硬件资源，并不是直接访问硬件资源
>
> **标准IO无法打开设备文件，比如套接字文件**
>
> 标准IO用一个结构体类型来存放打开的文件的相关信息
>
> 标准IO的所有操作都是围绕FILE来进行
>
> FILE 是 <stdio.h> 头文件中的一个结构体，它专门用来保存文件信息。我们不用关心 FILE 的具体结构，只需要知道它的用法就行。

## windows

> 文本流：   换行符==> '\r''\n'
>
> 二进制流:  换行符==> '\n'

## Linux

> linux不考虑文本流和二进制流的区别

## 缓冲类型

```
全缓冲

		当流的缓冲区无数据或者物空间时才执行IO

行缓冲

		当在输入和输出中遇到换行符，进行IO操作

无缓冲

		数据直接写入文件，流不经过缓冲
```

## 创建

> **标准输入文件 stdin（表示键盘）、标准输出文件 stdout（表示显示器）、标准错误文件 stderr（表示显示器）是由系统打开的，可直接使用。**

```c
FILE *fp = NULL;
fp = fopen("./test.txt", "模式");
```

模式：

| r    |            打开一个已有的文本文件，允许读取文件。            |
| ---- | :----------------------------------------------------------: |
| w    | 打开一个文本文件，允许写入文件。如果文件不存在，则会创建一个新文件。在这里，您的程序会从文件的开头写入内容。如果文件存在，则该会被截断为零长度，重新写入。 |
| a    | 打开一个文本文件，以追加模式写入文件。如果文件不存在，则会创建一个新文件。在这里，您的程序会在已有的文件内容中追加内容。 |
| r+   |               打开一个文本文件，允许读写文件。               |
| w+   | 打开一个文本文件，允许读写文件。如果文件已存在，则文件会被截断为零长度，如果文件不存在，则会创建一个新文件。 |
| a+   | 打开一个文本文件，允许读写文件。如果文件不存在，则会创建一个新文件。读取会从文件的开头开始，写入则只能是追加模式。 |

如果处理的是二进制文件，则需使用下面的访问模式来取代上面的访问模式：

```c
"rb", "wb", "ab", "rb+", "r+b", "wb+", "w+b", "ab+", "a+b"
 //linux没有二进制流与文本流区分
```

## 关闭

```c
fclose(fp);
// 关闭后，缓冲区的内容刷到硬盘
```

## 读写

### 以字符的形式

**读**

```c
getc(FILE * stream)
getchar(void)

  

int fgetc(FILE * stream) //推荐使用,fgetc 是 file get char 的缩写，意思是从指定的文件中读取一个字符。
  
  #include<stdio.h>
  int main()
  {
      printf("%c\n",fgetc(stdin));
      return 0;
  }

  


  
 /*
  成功返回读取的字符
	出错或者文件末尾返回EOF(一般是-1)
 */
```

**写**

> int putc(int c,FILE * stream)
> int putchar(int c)
>
> **int fputc(int c,FILE * stream)推荐使用，fputc 是 file output char 的所以，意思是向指定的文件中写入一个字符**
>
> 成功返回写入的字符，出错返回EOF(一般是-1)

```c
#include<stdio.h>
int main()
{
  char a = fgetc(stdin);
  fputc(a,stdout);
  putchar("\n")
    return 0;
}

```

**例子**

> 复制一个文件内容到另一个文件

```c
#include <stdio.h>
int main()
{
    FILE * fp1 = fopen("test1.c","r");
    FILE * fp2 = fopen("test2.c","w");
    char ch;
    if(fp1 == NULL){
        printf("空文件");
        return -1;
    }
    while(1){
        ch = fgetc(fp1);
        if(ch == EOF){
            return -1;
        }
        putc(ch, fp2);
    }
    
    fclose(fp1);
    fclose(fp2);
    
    return 0;
}

```

### 以字符串的形式

**读**

> gets
>
> 	不推荐使用，容易造成缓冲区溢出
>
> fgets
>
> 	成功返回s，到文件末尾或出错返回NULL
> 					
> 	遇到'\n'了，把'\n'读进去，然后末尾加'\0，'或已读到size-1个字符时返回，末尾加'\0'

```c
char * gets(char * s)


char * fgets(char * s,int size,FILE * stream)
      #include <stdio.h>
      #define SIZE 6
      int main()
      {

          char buf[SIZE];
          fgets(buf,SIZE,stdin);
          printf("%s",buf);
          /*
              这里引出一个问题，如果输入的字符串比SIZE还大，那怎么存
              假如键盘输入：abcd回车，那么会存abcd'\n''\0'
              假如键盘输入：abcdef回车，那么会存abcde'\0'
              不管怎么存，都必须保证结尾是'\0'
           */
          return 0;
      }

  
  
  

```

**写**

> int puts(const char * s)
> int fputs(const char * s,FILE * stream)
>
> 成功时返回写入的字符个数，出错时返回EOF
>
> puts将缓冲区s中的字符串写入到stdout，并追加'\n'
>
> fputs将缓冲区s中的字符串写入到stream中

```c
#include <stdio.h>
int main()
{
    puts("hello,world");
    return 0;
}
```

```c
#include <stdio.h>
int main()
{
    FILE * fp;
    char buff[] = "hello,world";
    fp = fopen("test.txt", "a");
    fputs(buff, fp);
}

```

### 以块的形式

> 成功返回读写的对象个数，失败返回EOF

**fread函数**

> fread函数用来读取块数据，啥的，所谓块数据，也就是若干个字节的数据，可以是一个字符，可以是一个字符串，可以是多行数据，结构体啥的

```c
size_t fread ( void *ptr, size_t size, size_t count, FILE *fp );
//参数二：每个块多少字节
//参数三：读取多少个块
//参数一：存在那里
//成功返回：读取对象的个数
//出错时：返回EOF
```

```c
#include <stdio.h>
int main()
{
    int s[10];
    if(fread(s, sizeof(int), 10, fp))
    {
        perror("fread");
        return -1
    }
    return 0;
}
```

**fwrite函数** 

> fwrite函数用来向文件中写入块数据

```c
size_t fwrite ( void * ptr, size_t size, size_t count, FILE *fp );

对参数的说明：

1）ptr 为内存区块的[指针](http://c.biancheng.net/c/80/)，它可以是数组、变量、结构体等。fread() 中的 ptr 用来存放读取到的数据，fwrite() 中的 ptr 用来存放要写入的数据。

2）size：表示每个数据块的字节数。

3）count：表示要读写的数据块的块数。

4）fp：表示文件指针。

5）理论上，每次读写 size*count 个字节的数据。

返回值：

`size_t` 是在 stdio.h 和 stdlib.h 头文件中使用 typedef 定义的数据类型，表示无符号整数，也即非负数，常用来表示数量。

返回成功读写的块数，也即 count。如果返回值小于 count：

对于 fwrite() 来说，肯定发生了写入错误，可以用 ferror() 函数检测。
对于 fread() 来说，可能读到了文件末尾，可能发生了错误，可以用 ferror() 或 feof() 检测。
```

```c
#include <stdio.h>
struct student{
    int no;
    char name[8];
    float score;
}s[] = {{1,"张",10}};
int main()
{
    fwrite(s,sizeof(struct student), 2, fp);
    
    return 0;
}

```

```c
#include<stdio.h>
#define N 2
struct stu{
    char name[10]; //姓名
    int num;  //学号
    int age;  //年龄
    float score;  //成绩
}boya[N], boyb[N], *pa, *pb;
int main(){
    FILE *fp;
    int i;
    pa = boya;
    pb = boyb;
    if( (fp=fopen("d:\\demo.txt", "wb+")) == NULL ){
        puts("Fail to open file!");
        exit(0);
    }
    //从键盘输入数据
    printf("Input data:\n");
    for(i=0; i<N; i++,pa++){
        scanf("%s %d %d %f",pa->name, &pa->num,&pa->age, &pa->score);
    }
    //将数组 boya 的数据写入文件
    fwrite(boya, sizeof(struct stu), N, fp);
    //将文件指针重置到文件开头
    rewind(fp);
    //从文件读取数据并保存到数据 boyb
    fread(boyb, sizeof(struct stu), N, fp);
    //输出数组 boyb 中的数据
    for(i=0; i<N; i++,pb++){
        printf("%s  %d  %d  %f\n", pb->name, pb->num, pb->age, pb->score);
    }
    fclose(fp);
    return 0;
}
```

```c
//复制一个文件内容到另外一个文件



#include <stdio.h>
#define N 60
int main()
{
    
    char buf[N];
    FILE * f1 = fopen("test.c", "r");
    FILE * f2 = fopen("test1.c", "w");
    
    if(f1 == NULL)
    {
        perror("fopen");
    }
    int flag = 1;
    while(flag){
        if(fread(buf, 1, N, f1) > 0)
        {
            fwrite(buf, 1, N, f2);
        }else{
            flag = 0;
        }
    }
    fclose(f1);
    fclose(f2);
    
    return 0;
}

```

### 以格式化的形式

> fscanf() 和 fprintf() 函数与前面使用的 scanf() 和 printf() 功能相似，都是格式化读写函数，两者的区别在于 fscanf() 和 fprintf() 的读写对象不是键盘和显示器，而是磁盘文件。

```c
int fprintf(FILE * stream,const char * fmt,...) 
int sprintf(char * s,const char * fmt,...)
//成功时返回数输出的字符个数，出错时返回EOF
```

```c
#include <stdio.h>
#define N 60
int main()
{
    
    int year,month,day;
    FILE * fp;
    char buf[64];
    
    year = 2021;
    month = 10;
    day = 3;
    fp = fopen("test.txt","w");
    fprintf(fp, "%d-%d-%d\n",year,month,day);
    sprintf(buf, "%d-%d-%d\n",year,month,day);
    printf("%s\n",buf);
    return 0;
}

```



## 流的刷新

```c
int fflush(FILE * stream)
  
成功返回0，出错返回EOF
linux下只能刷新输出缓冲区
  
在我们写入数据到文件时，只有当缓冲区满，或者遇到换行符时，或者关闭文件时，才会写入缓冲区，如果我们想强制刷进去。就需要执行fflush
```

## 流的定位

> 值得说明的是，fseek() 一般用于二进制文件，在文本文件中由于要进行转换，计算的位置有时会出错。

```c
long ftell(FILE * stream)
long fseek(FILE * stream,long offset,int whence)
void rewind(FILE * stream)
  
ftell函数：返回当前流的位置，出错返回EOF
fseek函数：定位一个流，成功返回0，出错返回EOF
  
whence参数：
起始点	常量名	常量值
文件开头	SEEK_SET	0
当前位置	SEEK_CUR	1
文件末尾	SEEK_END	2
```




## 错误信息处理

```c
#include<stdio.h>
int main()
{
    
    FILE * fp;
   
    fp = fopen("./a.c","r");
    if(fp == NULL)
    {
        perror("出错了!");//出错了!: No such file or directory
        
    }
    return 0;
}

```

## 对 EOF 的说明

> EOF 本来表示文件末尾，意味着读取结束，但是很多函数在读取出错时也返回 EOF，那么当返回 EOF 时，到底是文件读取完毕了还是读取出错了？我们可以借助 stdio.h 中的两个函数来判断，分别是 feof() 和 ferror()。

```c
int ferror(FILE * stream)
int feof(FILE * stream)
  
ferror函数：返回1流出错，否则返回0
feof函数：返回1表示文件已到末尾，否则返回0
```

## 应用

### 以块的形式拷贝文件

```c
#include <stdio.h>
#define N 10
typedef char elemType ;
int main(int argc,char * argv[])
{
        FILE *fps,*fpd;
        elemType buf[N];//缓冲区
        int n;//每次读取的对象的个数

        if(argc < 3){
                printf("参数少了!");// ./source source_file destination_file
                return -1;
        }

        //打开源文件
        if((fps = fopen(argv[1],"r")) == NULL){
            perror("打开文件失败");
            return -1;
        }
        //打开目标文件
        if((fpd = fopen(argv[2],"w")) == NULL){
            perror("打开文件失败");
            return -1;
        }
        while((n = fread(buf,sizeof(elemType),N,fps)) > 0 )
        {
            //每次读取N个块大小放入buf，然后读取N个块大小存入fpd所指文件
            fwrite(buf,sizeof(elemType),n,fpd);
        }
        fclose(fps);
        fclose(fpd);
        return 0;
}

```



# 二、文件I/O

> posix标准
>
> 不带缓冲区，每次调用都是系统调用，也就是每次调用都会向os请求
>
> **可以打开设备文件**

## 2.1 文件描述符

> 每个打开的文件都对应一个文件描述符
>
> 文件描述符是一个非负整数，从0开始分配，逐渐递增
>
> 文件IO操作通过文件描述符来完成
>
> 文件描述符：0、1、2分别对应标准标准输入，标准输出，标标错误，因为**通常一个进程启动时都会打开三个文件**

## open函数

> open函数用来打开或者创建一个文件
>
> int open(const char * path,int oflag，**[mode_t mode]**)   **[mode_t mode]：可选的**
>
> **成功返回文件描述符，出错返回EOF**
>
> 打开文件使用两个参数
>
> 创建文件使用三个参数，第三个参数指定文件的权限

**oflag参数**

| **参**  **数** | **参 数 解 释**                                              |
| -------------- | ------------------------------------------------------------ |
| **O_RDONLY**   | 只读方式打开                                                 |
| **O_WRONLY**   | 只写方式打开                                                 |
| **O_RDWR**     | 读写方式打开                                                 |
| **O_CREAT**    | 如果文件不存在则创建新文件。使用此选项时，需要指定mode参数   |
| **O_EXCL**     | 如果同时指定O_CREAT属性，而且文件存在，open()函数返回错误。用这个办法可以测试文件是否存在，如果文件不存在则创建新文件，这个操作是原子的，不会被其他程序打断 |
| **O_APPEND**   | 以添加方式打开文件，在打开文件的同时，文件指针指向文件的末尾，即将写入的数据添加到文件的末尾 |
| **O_TRUNC**    | 如果pathname指向的文件存在，而且作为只读或者只写成功打开，则把文件长度截断为0 |

**mode参数**

> 文件的权限

**例子**

> 以只写方式打开文件1.txt，如果文件不存在则创建，存在则清空

```c
int fd;
fd = open("1.txt",O_WRONLY|O_CREAT|O_TRUNC,666);
if(fd < 0)
{
  perror("open");
  return -1;
}
```

## close函数

> close函数用来关闭一个打开的文件

```c
int close(int fd);
成功返回0，出错返回EOF
程序结束时自动关闭所有打开的文件
```

## read函数

> 从文件中读取数据

```c
#include <unistd.h>
ssize_t read(int fd,void * buf,size_t count)
  
成功时返回实际读取的字节数，出错返回EOF
读到文件末尾返回0
buf是存储读取的数据的
count不应超过buf大小
```

## write函数

```c
#include <unistd.h>
ssize_t write(int fd, const void *buf, size_t count)

成功返回实际写入的字节数，出错返回FOF
buf是发送数据的缓冲区
count不应超过buf


```

## lseek函数

> 文件定位函数

```c
#include <unistd.h>
off_t lseek(int fd, off_t offset, int whence);

成功返回当前文件读写位置，出错返回EOF
offset和whence同fseek的两个参数用法完全一样
```

## 例子

> 复制一个文件内容到另外一个文件

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>


#define MAX_SIZE 64
int main()
{
    int f1,f2,n;
    
    char buff[MAX_SIZE];
    f1 = open("f1.txt",O_RDONLY);
    f2 = open("f2.txt",O_WRONLY|O_CREAT|O_TRUNC,0666);
    
    if(f1 == -1)
    {
        perror("open");
        return -1;
    }
    
    if(f2  == -1)
    {
        perror("open");
        return -1;
    }
    
    while(1)
    {
        n = read(f1,buff,MAX_SIZE);
        if(n > 0)
        {
            write(f2, buff, n);
        }else{
            break;
        }
        
    }
    close(f1);
    close(f2);
    return 0;
}

```

# 三、目录操作和文件属性

## 目录操作

**opendir函数**

> opendir函数打开一个目录文件

```c
#include <dirent.h>
DIR * opendir(const char * name)
  
DIR使用来描述一个打开的目录文件的结构体类型
成功时返回目录流指针，出错返回NULL
```

**readdir函数**

> 访问打开的目录

```c
#include <dirent.h>
struct dirent * readdir(DIR * dirp)

struct dirent是用来描述流中的一个目录项的结构体类型
包含成员char d_name[256]，d_name是目录下文件的名称（一切皆文件）
```

**closedir函数**

> 关闭一个打开的目录文件

```c
#include <sirent.h>
int closedir(DIR * dirp)
```

例子

> 打印指定目录下的所有文件名称(一切皆文件)

```c
#include <stdio.h>
#include <dirent.h>
int main()
{
    DIR * dirp;
    struct dirent * dp;
    if((dirp=opendir("test1")) == NULL)
    {
        perror("opendir");
        return -1;
    }
    while((dp=readdir(dirp))!=NULL)
    {
        printf("%s\n",dp->d_name);
    }
  	closedir(dirp);
    return 0;
}
```

## 文件权限

> Chmod/fchmod函数用来修改文件的访问权限
>
> 成功返回0，失败返回EOF
>
> 只有文件拥有者和roor可以修改文件权限

```c
#include <sys/stat.h>
int chmod(const char * path,mode_t mode)
int fchmod(int fd,mode_t mode)
示例：chmod("test.txt",0666);
```

## 文件属性

