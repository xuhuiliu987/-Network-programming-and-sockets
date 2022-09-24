#  1、基于Linux的文件操作

  讨论套接字的过程中是不是突然谈及文件也许有些奇怪，但是呢，在Linux系统下其实一点也不奇怪，socket操作和文件操作对于Linux系统是没有任何区别的，所有我们很有必要学习一下文件操作，那么在这里就是文件IO的操作了

# 2、底层文件访问和文件描述符

 那么“底层”这个表达可以理解为“与标准无关的操作系统独立提供的”，稍后讲解的函数都是有Linux提供的，而发ANSI标准定义的函数，

| 表1-1： 分配给标准输入输出及标准错误的文件描述符             |
| ------------------------------------------------------------ |
| 文件描述符                                                                 对象 |
| 0                                                                standard input |
| 1                                                                 stabdard output |
| 2                                                                 standard error |

文件和套接字一般经过创建过程才会被分配文件描述符，在上表中的3种输入输出对象即使没有经过特殊的创建过程，程序开始运行后也会自动分配文件描述符。

## 2.1：打开文件

  首先介绍一下打开文件以读写数据的函数。调用此函数时需要传递两个参数：第一个参数是打开的目标文件名以及路径信息，第二个参数是文件打开模式（文件特性信息）

```cpp
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int open(const char *path,int flag);
//成功返回文件描述符，失败时返回-1
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 表1-2是此函数第二个参数flag的宏，如果需要传递多个参数，则应通过位或运算。

| 表1-2：文件打开模式                                          |
| ------------------------------------------------------------ |
| 打开模式                                                                     含义 |
| O_CREAT                                                            必要时创建文件 |
| O_TRUNC                                                          删除全部现有数据 |
| O_APPEND                                                         维持现有数据，保存到后面 |
| O_RDONLY                                                            只读打开 |
| O_WRONLY                                                            只写打开 |
| O_RDWR                                                               读写打开 |

## 2.2：关闭文件

 我们知道使用文件后必须关闭，

```cpp
#include <unistd.h>

int close(int fd)
//fd：需要关闭的文件或者套接字的文件描述符
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

如果要调用这个函数则必须传递文件描述符，则关闭相应文件

## 2.3：将数据写入文件

   write函数可以用于向文件输出数据，在Linux中不区分文件与套接字，因此，通过套接字向其他计算机传递数据的时候也会用到这个函数

```cpp
#include <unistd.h>

ssize_t write(int fd,const void *buf,size_t nbytes);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 fd：显示数据传输对象的文件描述符

buf：保持要传输数据的缓冲地址值

nbytes：传输数据的字节数

## 2.4：小实战

那我们用一下上面这三个函数，帮助大家理解一下，我们创建一个文件并保持数据

```cs
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
void error_handling(char *message);

int main()
{
	int fd;
	char buf[]="lest go!";
	fd=open("data.txt",O_WRONLY);
	if(fd<0)
	{
		error_handling("open() error");
	}
	printf("file descriptor %d \n",fd);
	int ret=-1;
	ret=write(fd,buf,sizeof(buf));
	if(ret<0)
	{
		error_handling("write() errro");
	}
	close(fd);
	return 0;
}
void error_handling(char *message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 2.5：读取文件中的数据

跟write基本一样

```cs
#include <unistd.h>

ssize_t read(int fd,void *buf,size_t nbytes);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

我们用read读取数据

```cs
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
void error_handling(char *message);

int main()
{
	int fd;
	char buf[100]={0};
	fd=open("data.txt",O_RDONLY);
	if(fd<0)
	{
		error_handling("open() error");
	}
	printf("file descriptor %d \n",fd);
	int ret=-1;
	ret=read(fd,buf,sizeof(buf));
	if(ret<0)
	{
		error_handling("read() errro");
	}
	printf("%s",buf);
	close(fd);
	return 0;
}
void error_handling(char *message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}
```

## 2.6：完结撒花

基于文件描述符的操作我们到此就结束了，同时希望同学们记住，该内容同样也适用于套接字哦