# 第7章TCP网络编程基础

![img](file:///C:\Users\Gyjz\AppData\Local\Temp\msohtmlclip1\01\clip_image001.png)

客户端程序设计与服务器端不同之处：客户端在套接字初始化之后可以不进行地址绑定,而是直接连接服务器端。

## socket()

```
#include <sys/types.h>
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
```

domain:设置网络通信的域，即选择通信协议的族

![image-20220708133226983](C:\Users\Gyjz\AppData\Roaming\Typora\typora-user-images\image-20220708133226983.png)

type:设置套接字通信类型

![image-20220708133503198](C:\Users\Gyjz\AppData\Roaming\Typora\typora-user-images\image-20220708133503198.png)

**protocol：**用于指定某个协议的特定类型，即type类型中的某个类型

```
int sock = socket(AF_INET, SOCK_STREAM,0);一个TCP套接字
```

成功返回一个文件描述符,-1表示失败

## bind()

将长度addlen的struct sockaddr类型的参数my_addr与sockfd绑定在一起

```
#include <sys/types.h>
#include <sys/socket.h>
int bind(int sockfd, const struct sockaddr *my_addr, socklen_t addrlen);
```

sockfd：socket()创建的文件描述符

my_addr：指向一个结构为sockaddr参数的指针，包含地址，端口，IP地址

addrlen：my_addr结构的长度，可以设置为sizeof(struct sockaddr)  

返回值为0时表示绑定成功,-1表示绑定失败

## listen()

初始化服务器可连接队列，服务器处理客户端连接请求是顺序处理的，同一时间处理一个客户端连接，不能处理的连接请求放在等待队列中，队列长度由listen()来定义



```c
#include <sys/socket.h>
int listen(int sockfd, int backlog);
```

backlog:等待队列长度

成功返回0,失败返回-1

## accept()

​	当一个客户端的连接请求到达服务器主机侦听的端口时,此时客户端的连接会在队列中等待,直到使用服务器处理接收请求。

​	函数accept()成功执行后,会返回一个新的套接口文件描述符来表示客户端的连接,客户端连接的信息可以通过这个新描述符来获得。因此当服务器成功处理客户端的请求连接后,会有两个文件描述符,老的文件描述符表示正在监听的socket,新产生的文件描述符表示客户端的连接,函数send()和recv()通过新的文件描述符进行数据收发。

```
#include <sys/types.h>
#include <sys/socket.h>
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

addr：将客户端的IP地址，端口，协议信息存到这里

addrlen：addr的长度

成功返回新产生的文件描述符，通过这个描述符通信，失败返回-1

## connect()

客户端建立套接字后，不需要绑定地址就可以连接服务器，但需指定服务器的IP地址，端口

```
#include <sys/types.h>
#include <sys/socket.h>
int connect(int sockfd, struct sockaddr *serv_addr, int addrlen);
```

serv_addr  ：指向服务器的IP地址，端口的数据结构

addrlen：serv_addr的大小，用sizeof(struct sockaddr)  

成功返回0，失败返回-1

## write（）

通过套接字进行数据的写入

```
int size ;
char data[1024];
size = write(s, data, 1024);
```

将缓冲区data的数据全部写入套接字文件描述符s中,返回值为成功写入的数据长度。

文件描述符为0：标准输入，1：标准输出

## read()

通过套接字进行数据的读取

```
int size ;
char data[1024];
size = read(s, data, 1024);
```

从套接字描述符s中读取1024个字节,放入缓冲区data中,size值为成功读取的数据大小。

文件描述符为0：标准输入，1：标准输出

## close()

关闭socket连接



## shutdown()

用更多方式来关闭连接，可以单方面切断或切断双方通信

```
#include <sys/socket.h>
int shutdown(int s, int how);
```

how:![image-20220708145812839](C:\Users\Gyjz\AppData\Roaming\Typora\typora-user-images\image-20220708145812839.png)

## signal()

对某事件发生时的通知，软中断，相关进程对信号进行捕捉和处理

```
#include <signal.h>
typedef void (*sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);
```

handler:函数的句柄，进程捕捉到信号时，调用响应函数的句柄

## SIGPIPE

若一个SIGPIPE信号产生，会终止当前进程

```
void sig_pipe(int sign)
{
	printf("Catch a SIGPIPE signal\n");
}
signal(SIGPIPE, sig_pipe);
```

## SIGINT

由Ctrl+C造成

```
void sig_int(int sign)
{
	printf("Catch a SIGINT signal\n");
}
signal(SIGINT, sig_pipe);
```

