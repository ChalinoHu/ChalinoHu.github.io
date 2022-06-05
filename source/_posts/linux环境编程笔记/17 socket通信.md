---
title: 17 socket通信
date: 2022-05-27 21:02:46
tags: linux环境编程
---



[toc]

# 1、Socket介绍

socket（套接字），就是对网络中**不同主机上的应用进程**之间进行双向通信的端点的抽象。

套接字**上联应用进程，下联网络协议栈**，是应用程序通过网络协议进行通信的接口。

**一个套接字**就是网络上进程通信的**一端**，使用中的每一个套接字都有其类型，并有一个进程与之相连。通信时其中一个网络应用程序将要传输的一段信息写入它所在主机的 socket 中，socket与网卡相连并将数据输送到另一台主机的socket中，使对方能够接受到信息。

Socket 是由 **IP 地址**和**端口**结合的，提供向应用层进程传送数据包的机制。

在 Linux 环境下，socket是一种用于表示进程间网络通信的**特殊文件类型**，本质为**内核借助缓冲区形成的伪文件**。作为一种文件类型，linux系统中可以使用**文件描述符**来引用socket。<u>与管道类似的，Linux 系统将其封装成文件的目的是为了统一接口，使得读写套接字和读写文件的操作一致。</u>区别是管道主要应用于本地进程间通信，而套接字多应用于网络进程间数据的传递。

套接字通信一般分为两部分：

​	\- 服务器端：被动接受连接，一般不会主动发起连接 

​	\- 客户端：主动向服务器发起连接 

# 2、字节序

字节序，顾名思义字节的顺序，就是大于一个字节类型的数据在内存中的存放顺序(一个字节的数据当然就无需谈顺序的问题了)

## 2.1、大端字节序

​	一个整数的**高位字节**（23 ~ 31 bit）存储在内存的**低地址处**，**低位字节**（0 ~ 7 bit）存储在内存的**高地址**处；

### 大端字节序示例

如果一个数据有四个字节 `0x01 02 03 04`， 最低位字节是04， 最高位字节是01

假设内存的地址增长方向是从左往右，低地址在左，高地址在右

存在内存中的数据为 `0x01 0x02 0x03 0x04`

## 2.2、小端字节序

一个整数的**高位字节**存储在内存的**高地址处**，而**低位字节**则存储在内存的**低地址处**；

### 小端字节序示例

如果一个数据有四个字节 `0x01 0x02 0x03 0x04`， 最低位字节是04， 最高位字节是01。

假设内存的地址增长方向是从左往右，低地址在左，高地址在右

存在内存中的数据为 `0x04 0x03 0x02 0x01`

程序验证：

```c
/*
字节序：字节在内存中存储的顺序,每次以多个字节写入内存时会存在不同的字节顺序。
小端字节序：高位字节存储在内存的高地址处，低位字节存储在内存的低地址处
大端字节序：高位字节存储在内存的低地址处，低位字节存储在内存的高地址处
*/

#include <stdio.h>

int main(int argc, char *argv[]){
    
    union byte_order
    {
        short value;    // 2个字节
        char bytes[sizeof(short)];  // char[2]
    } test;
    
    test.value = 0x0102;	// 写入两个字节的数据
    
    if (test.bytes[0] == 0x01 && test.bytes[1] == 0x02){
        // 高字节放低地址处，低字节放高地址处为大端
        printf("大端字节序\n");
    }else if (test.bytes[0] == 0x02 && test.bytes[1] == 0x01){
        // 高字节放高位置处，低字节放低地址处为小端
        printf("小端字节\n");
    }else printf("未知\n");
    return 0;
}
```

## 2.3、字节序转换

当格式化的数据在两台使用不同字节序的主机之间直接传递时，接收端必然错误的解释之。解决问题的方法是：发送端总是把要发送的数据转换成**大端字节序**数据后再发送，而接收端知道对方传送过来的数据总是采用大端字节序，所以接收端可以根据自身采用的字节序决定是否对接收到的数据进行转换（小端机转换，大端机不转换）。

**网络字节顺序**是 TCP/IP 中规定好的一种数据表示格式，它与具体的 CPU 类型、操作系统等无关，从而可以保证数据在不同主机之间传输时能够被正确解释，网络字节顺序采用**大端排序**方式。

```c
/*
    #include <arpa/inet.h>
    // 32位数据之间的转换, 一般用于转换IP
    uint32_t htonl(uint32_t hostlong);  // 主机字节序 -> 网络字节序
    uint32_t ntohl(uint32_t netlong);   // 网络字节序 -> 主机字节序

    // 16位数据之间的转换，一般用于转换端口
    uint16_t htons(uint16_t hostshort); // 主机字节序 -> 网络字节序
    uint16_t ntohs(uint16_t netshort);  // 网络字节序 -> 主机字节序

    字母释义：
        h   - host 主机，主机字节序
        n   - network 网络，网络字节序
        to
        s   - unsigned short 16位，2字节
        l   - unsigned long  32位，4字节
*/

// 本机字节序是小端模式，和网络字节序大端模式不同，在进行ntoh和hton时都会对数据进行转换

#include <stdio.h>
#include <arpa/inet.h>

int main(int argc, char *argv[]){
    
    // htons 转换端口
    unsigned short a = 0x0102;
    printf("a: %x\n", a);
    unsigned short b = htons(a);
    printf("b: %x\n", b);  // 输出201


    // htonl 转换IP
    unsigned char buf[4] = {192, 168, 198, 128};
    int num = *((int *)buf);
    int num_ = htonl(num);
    unsigned char *p = (unsigned char *)&num_;
    printf("%d %d %d %d\n", *p, *(p + 1), *(p + 2), *(p + 3));  // 输出 128 198 168 192
    
    
    // ntohl
    unsigned char buf1[4] = {128, 198, 168, 192};
    int num1 = *(int*)buf1;
    int num1_ = ntohl(num1);
    unsigned char *p1 = (unsigned char* ) &num1_;
    printf("%d %d %d %d\n", *p1, *(p1 + 1), *(p1 + 2), *(p1 + 3));  // 输出 192 168 198 128   

    return 0;


}
```

# 3、socket地址

`socket`地址其实是一个结构体，封装端口号和`IP`等信息。客户端 -> 服务器`（IP, Port）`

## 3.1、通用socket地址

`socket` 网络编程接口中表示 `socket` 地址的是结构体 `sockaddr`，其定义如下：

```c++
#include <bits/socket.h> 
struct sockaddr { 
    sa_family_t sa_family; 
    char sa_data[14]; 
};
typedef unsigned short int sa_family_t;
```

`sa_family` 成员是地址族类型（`sa_family_t`）的变量。

地址族类型通常与协议族类型对应。常见的协议族（`protocol family`，也称 `domain`）和对应的地址族入下所示：

<img src="/images/image\AF_XXX协议簇.png" style="zoom:38%;" />

宏 `PF_XXX`和 `AF_XXX` 都定义在 `bits/socket.h` 头文件中，且后者与前者有完全相同的值，所以二者通常混用。

`sa_data` 成员用于存放 `socket` 地址值。但是，不同的协议族的地址值具有不同的含义和长度，如下所示：

<img src="/images/image/协议值含义和长度.png" style="zoom:40%;" />

由上表可知，14 字节的 `sa_data` 根本无法容纳多数协议族的地址值。因此，Linux 定义了<u>下面这个新的通用的 socket 地址结构体</u>，这个结构体不仅提供了足够大的空间用于存放地址值，而且是**内存对齐**的。

```c
#include <bits/socket.h> 
struct sockaddr_storage { 
    sa_family_t sa_family; 
    unsigned long int __ss_align; 
    char __ss_padding[ 128 - sizeof(__ss_align)]; 
};
typedef unsigned short int sa_family_t;
```

## 3.2、专用socket地址

为了向前兼容`struct sockaddr`结构体，现在`sockaddr`退化成了类似(void*)的作用，传递一个地址给函数，至于这个函数是 `sockaddr_in` 还是`sockaddr_in6`，由地址族确定，然后函数内部再强制类型转化为所需的地址类型。

<img src="/images/image/sockaddr协议簇.png" style="zoom:40%;" />

`UNIX` 本地域协议族使用如下专用的 `socket` 地址结构体：

```c
#include <sys/un.h> 
struct sockaddr_un { 
    sa_family_t sin_family; 
    char sun_path[108]; 
};
```

`TCP/IP` 协议族有 `sockaddr_in` 和 `sockaddr_in6` 两个专用的 `socket` 地址结构体，它们分别用于 `IPv4` 和`IPv6`：

```c
#include <netinet/in.h> 
struct sockaddr_in { 
    sa_family_t sin_family; /* __SOCKADDR_COMMON(sin_) */ 
    in_port_t sin_port; /* Port number. */ 
    struct in_addr sin_addr; /* Internet address. */ 
    /* Pad to size of `struct sockaddr'. */ 
    unsigned char sin_zero[sizeof (struct sockaddr) - __SOCKADDR_COMMON_SIZE - sizeof (in_port_t) - sizeof (struct in_addr)]; 
};

struct in_addr { 
    in_addr_t s_addr; 
};

struct sockaddr_in6 { 
    sa_family_t sin6_family; 
    in_port_t sin6_port; /* Transport layer port # */ 
    uint32_t sin6_flowinfo; /* IPv6 flow information */ 
    struct in6_addr sin6_addr; /* IPv6 address */ 
    uint32_t sin6_scope_id; /* IPv6 scope-id */ 
};

typedef unsigned short uint16_t; 
typedef unsigned int uint32_t; 
typedef uint16_t in_port_t; 
typedef uint32_t in_addr_t; 
#define __SOCKADDR_COMMON_SIZE (sizeof (unsigned short int))
```

所有专用 socket 地址（以及 `sockaddr_storage`）类型的变量在实际使用时都需要转化为通用 socket 地址类型 `sockaddr`（强制转化即可），因为所有 socket 编程接口使用的地址参数类型都是 `sockaddr`。

# 4、IP地址转换（字符串ip-整数 ，主机、网络字节序的转换）

编程中需要将点分十进制的字符串转换为整数使用，日志记录需要将整数表示的IP转换成字符串表示的IP；

下面是三个古老的IP地址转换函数：

```c
#include <arpa/inet.h> 
in_addr_t inet_addr(const char *cp); 
int inet_aton(const char *cp, struct in_addr *inp); 
char *inet_ntoa(struct in_addr in);
```

现在的开发一般用下面的函数：

```c
/*
#include <arpa/inet.h> 
// p:点分十进制的IP字符串，n:表示network，网络字节序的整数 
int inet_pton(int af, const char *src, void *dst); 
    af:地址族： AF_INET AF_INET6 
    src:需要转换的点分十进制的IP字符串 
    dst:转换后的结果保存在这个里面 

// 将网络字节序的整数，转换成点分十进制的IP地址字符串 
const char *inet_ntop(int af, const void *src, char *dst, socklen_t size); 
    af:地址族： AF_INET AF_INET6 
    src: 要转换的ip的整数的地址 
    dst: 转换成IP地址字符串保存的地方 
    size：第三个参数的大小（数组的大小） 
    返回值：返回转换后的数据的地址（字符串），和 dst 是一样的
*/

#include <stdio.h>
#include <arpa/inet.h>
#include <bits/socket.h>

int main(){
    // 创建一个点分十进制的IP字符串
    char buf[] = "192.168.1.4";
    unsigned int num = 0;

    // 将点分十进制的Ip字符串转换为网络字节序的整数
    inet_pton(AF_INET, buf, &num);
    unsigned char *p = (unsigned char*)&num;
    printf("%d %d %d %d\n", *p, *(p + 1), *(p + 2), *(p + 3));  // 输出 192 168 1 4

    // 将网络字节序的Ip整数转换为十进制的IP字符串
    char ip[16] = "";
    const char* str = inet_ntop(AF_INET, &num, ip, 16);
    printf("%s\n", str);    // 输出 192.168.1.4
    printf("%s\n", ip);     // 输出 192.168.1.4

    return 0;
}
```

# 5、TCP通信流程

|                | UDP                            | TCP                        |
| -------------- | ------------------------------ | -------------------------- |
| 是否创建链接   | 无连接                         | 有连接                     |
| 是否可靠       | 不可靠                         | 可靠的                     |
| 连接的对象个数 | 一对一、一对多、多对一、多对多 | 支持一对一                 |
| 传输的方式     | 面向数据报                     | 面向字节流                 |
| 首部开销       | 8个字节                        | 最少需要20个字节           |
| 适用场景       | 实时应用（视频会议，直播）     | 可靠性高的应用（文件传输） |

<img src="/images/image/TCP通信流程.png" style="zoom:60%;" />

## TCP 服务端的通信流程

```
// 服务器端 （被动接受连接的角色） 
1. 创建一个用于监听的套接字 
	- 监听：监听有客户端的连接 
	- 套接字：这个套接字其实就是一个文件描述符 
2. 将这个监听文件描述符和本地的IP和端口绑定（IP和端口就是服务器的地址信息） 
	- 客户端连接服务器的时候使用的就是这个IP和端口 
3. 设置监听，监听的fd开始工作 
4. 阻塞等待，当有客户端发起连接，解除阻塞，接受客户端的连接，会得到一个和客户端通信的套接字（fd） 
5. 通信 - 接收数据 - 发送数据 
6. 通信结束，断开连接
```



## TCP客户端的通信流程

```
// 客户端 
1. 创建一个用于通信的套接字（fd） 
2. 连接服务器，需要指定连接的服务器的 IP 和 端口 
3. 连接成功了，客户端可以直接和服务器通信 
	- 接收数据 
	- 发送数据 
4. 通信结束，断开连接
```

## socket函数

```c++

    #include <sys/types.h>
    #include <sys/socket.h>
	#include <arpa/inet.h>	// 包含这个头文件，上面两个就可以忽略

    int socket(int domain, int type, int protocol);
        - 功能:创建一个套接字
        - 参数:
            - domain:协议族
                - AF_UNIX, AF_LOCAL   Local communication
                - AF_INET             IPv4 Internet protocols
                - AF_INET6            IPv6 Internet protocols
            - type:通信过程中使用的协议语义
                - SOCK_STREAM: 流式协议
                - SOCK_DGRAM ：报式协议 
            - protoc: 具体的一个协议，一般写0
                - SOCK_STREAM: 流式协议默认使用TCP
                - SOCK_DGRAM: 报式协议默认使用 UDP
        - 返回值: 
            - 成功: 返回文件描述符，操作内存缓冲区
            - 失败: -1, 并设置错误号

    int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
        - 功能: 绑定，将fd与地址为addr的服务器(IP + 端口)进行绑定
        - 参数:
            - sockfd: 通过socket函数得到的文件描述符
            - addr: 需要绑定的socket地址，这个地址封装了ip和端口号的信息
            - addrlen: 第二个参数结构体所占内存的大小
    
    int listen(int sockfd, int backlog);
        - 功能: 监听这个socket上的连接
        - 参数:
            - sockfd: 通过socket()函数创建的文件描述符
            - backlog: 队列中要排队的未完成的连接请求的数量(套接字在监听时会创建两个队列，一个是未连接的，一个是已经连接的，backlog是规定的两个队列中请求的最大值)
        - 返回值:
            - 成功: 返回0
            - 失败: 返回-1
    
    int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
        - 功能: 接受客户端的连接，默认是一个阻塞的函数，阻塞等待客户端连接
        - 参数：
            sockfd: 用于监听的文件描述符
            addr: 传出参数， 记录连接成功后客户端的地址信息(ip, 端口)
            addrlen: 指定第二个参数的所占内存的大小
        - 返回值:
            成功: 返回用于通信的文件描述符
            失败: -1
    
    int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
        - 功能: 客户端连接服务器
        - 参数: 
            - sockfd: 用于通信的文件描述符
            - addr: 客户端要连接的服务器地址信息
            - addrlen: 第二个参数的内存大小
        - 返回值:
            - 成功: 0
            - 失败: -1
    
    ssize_t read(int fd, void *buf, size_t count);          // 读数据
    ssize_t write(int fd, const void *buf, size_t count);   // 写数据


```

## socket案例：TCP服务器和客户端通信

```c
// TCP 通信的服务端

#include <stdio.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main(){
    
    // 1、创建用于监听的套接字
    int lfd = socket(AF_INET, SOCK_STREAM, 0);    // ipv4, tcp协议
    if (lfd == -1){
        perror("socket");
        exit(-1);
    }


    // 2、绑定
    struct sockaddr_in saddr;
    saddr.sin_family = PF_INET;
    inet_pton(AF_INET, "192.168.198.128", &saddr.sin_addr.s_addr);
    // saddr.sin_addr.s_addr = INADDR_ANY; // 0.0.0.0
    saddr.sin_port = htons(9999);
    int ret = bind(lfd, (struct sockaddr*)&saddr, sizeof(saddr));
    if (ret == -1){
        perror("bind");
        exit(-1);
    }

    // 3、监听
    ret = listen(lfd, 8);
    if (ret == -1){
        perror("listen");
        exit(-1);
    }

    // 4、接受客户端连接
    struct sockaddr_in clientaddr;
    socklen_t len = sizeof(clientaddr);
    int cfd = accept(lfd, (struct sockaddr *)&clientaddr, &len);        // 等待连接是阻塞的
    if (cfd == -1){
        perror("accept");
        exit(-1);
    }

    // 输出客户端的信息
    char client_ip[16] = "";
    inet_ntop(AF_INET, &clientaddr.sin_addr.s_addr, client_ip, 16); // ip地址的网络字节序转换为点分十进制
    unsigned short client_port = ntohs(clientaddr.sin_port);
    printf("The client ip is %s, port is %d\n", client_ip, client_port);
    
    // 5、
    // 获取客户端的数据
    char recvBuf[1024] = {0};

    while(1){
        memset(recvBuf, 0, sizeof recvBuf);
        ret = read(cfd, recvBuf, sizeof(recvBuf));
        if (ret == -1){
            perror("read");
            exit(-1);
        }else if (ret > 0){
            printf("recv data : %s\n", recvBuf);
        }else if (ret == 0){
            // 表示客户端断开连接
            printf("client closed...\n");
            break;
        }


        // 给客户端发送数据
        // char *sendData = "ni hao client, i am server";
        write(cfd, recvBuf, strlen(recvBuf));
    }


    // 关闭文件描述符
    close(lfd);
    close(cfd);
    return 0;
}
```

```c

// TCP通信客户端

#include <unistd.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
int main(){

    // 1、创建socket
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd == -1){
        perror("socket");
        exit(-1);
    }

    // 2、连接服务器
    struct sockaddr_in serveraddr;
    serveraddr.sin_family = AF_INET;    // ipv4
    inet_pton(AF_INET, "192.168.198.128", &serveraddr.sin_addr.s_addr); // 点分十进制的IP地址转换为网络字节序的整数
    serveraddr.sin_port = htons(9999);  // 本地小端端口号转换为网络字节序
    int ret = connect(fd, (const struct sockaddr*)&serveraddr, sizeof(serveraddr));
    if (ret == -1){
        perror("connect");
        exit(-1);
    }

    // 3、通信
    char recvBuf[1024] = {0};

    const char *buf = "ni hao, I am client";
    write(fd, buf, strlen(buf));
    while (1){

        memset(recvBuf, 0, sizeof recvBuf);
        int len = read(fd, recvBuf, sizeof(recvBuf));
        if (len > 0){
            printf("recv server data: %s\n", recvBuf);
        }else if (len == -1){
            perror("read");
            exit(-1);
        }else if (len == 0){
            printf("server closed\n");
            break;
        }
        memset(recvBuf, 0, sizeof recvBuf);
        read(STDIN_FILENO, recvBuf, sizeof recvBuf);
        write(fd, recvBuf, strlen(recvBuf));      
    }


    close(fd);
    return 0;
}   
```

