---
title: 21 TCP状态转换
date: 2022-05-27 21:02:46
tags: linux环境编程
---



# TCP状态状态

<img src="/images/image/TCP连接状态.png" style="zoom:38%;" />

​	

主动断开连接的一方，最后会进入到一个TIME_WAIT状态，这个状态会持续2MSL(Maximum Segment Lifetime)

MSL官方建议是2分钟，linux中实际是30秒。

当TCP连接主动关闭方接收到被动关闭方发送FIN和Ack后，连接的主动关闭方必须处于TIME_WAIT状态闭关持续2MSL时间。

这样就能让TCP连接的主动关闭方在它发送的Ack丢失的情况下重新发送最终的Ack。

主动关闭方重新发送的最终Ack并不是因为被动关闭方重传了Ack（它们并不消耗序列号，被动关闭方也不会重传），而是因为被动关闭方重传了它的FIN。事实上被动关闭放总是重传FIN直到它收到一个最终的Ack。

使用LinuxAPI来设定半连接。

```c
#include <sys/socket.h> 
int shutdown(int sockfd, int how); 
	sockfd: 需要关闭的socket的描述符 
    how: 允许为shutdown操作选择以下几种方式: 
		SHUT_RD(0)： 关闭sockfd上的读功能，此选项将不允许sockfd进行读操作。 该套接字不再接收数据，任何当前在套接字接受缓冲区的数据将被无声的丢弃掉。		  SHUT_WR(1): 关闭sockfd的写功能，此选项将不允许sockfd进行写操作。进程不能在对此套接字发 出写操作。 
		SHUT_RDWR(2):关闭sockfd的读写功能。相当于调用shutdown两次：首先是以SHUT_RD,然后以 SHUT_WR。
```

使用 `close()` 中止一个连接，但它只是减少描述符的引用计数，并不直接关闭连接，只有当描述符的引用计数为 0 时才关闭连接。`shutdown()` 不考虑描述符的引用计数，直接关闭描述符。也可选择中止一个方向的连接，只中止读或只中止写。

注意: 

1. 如果有多个进程共享一个套接字，`close()`每被调用一次，计数减 1 ，直到计数为 0 时，也就是所用进程都调用了 `close()`，套接字将被释放。

2. 在多进程中如果一个进程调用了 `shutdown(sfd, SHUT_RDWR)` 后，其它的进程将无法进行通信。但如果一个进程 `close(sfd)` 将不会影响到其它进程。



端口复用

```c
#include <sys/types.h> 
#include <sys/socket.h> 
// 设置套接字的属性（不仅仅能设置端口复用） 
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen); 
	参数：
	- sockfd : 要操作的文件描述符 
	- level : 级别 - SOL_SOCKET (端口复用的级别) 
	- optname : 选项的名称 
		- SO_REUSEADDR 
		- SO_REUSEPORT 
	- optval : 端口复用的值（整形） - 1 : 可以复用 - 0 : 不可以复用 
	- optlen : optval参数的大小 端口复用，设置的时机是在服务器绑定端口之前。 
	
    // 调用先后顺序
	setsockopt(); 
	bind();
```

查看网络相关信息的状态：

```bash
netstat
	-a 所有的socket
	-p 显示正在使用socket的应用程序的名称
	-n 直接使用IP地址，而不是通过域名服务器
```

服务端

```c
#include <stdio.h>
#include <ctype.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char *argv[]) {

    // 创建socket
    int lfd = socket(PF_INET, SOCK_STREAM, 0);

    if(lfd == -1) {
        perror("socket");
        return -1;
    }

    struct sockaddr_in saddr;
    saddr.sin_family = AF_INET;
    saddr.sin_addr.s_addr = INADDR_ANY;
    saddr.sin_port = htons(9999);
    
    //int optval = 1;
    //setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &optval, sizeof(optval));
    int optval = 1;
    setsockopt(lfd, SOL_SOCKET, SO_REUSEPORT, &optval, sizeof(optval));

    // 绑定
    int ret = bind(lfd, (struct sockaddr *)&saddr, sizeof(saddr));
    if(ret == -1) {
        perror("bind");
        return -1;
    }

    // 监听
    ret = listen(lfd, 8);
    if(ret == -1) {
        perror("listen");
        return -1;
    }

    // 接收客户端连接
    struct sockaddr_in cliaddr;
    socklen_t len = sizeof(cliaddr);
    int cfd = accept(lfd, (struct sockaddr *)&cliaddr, &len);
    if(cfd == -1) {
        perror("accpet");
        return -1;
    }

    // 获取客户端信息
    char cliIp[16];
    inet_ntop(AF_INET, &cliaddr.sin_addr.s_addr, cliIp, sizeof(cliIp));
    unsigned short cliPort = ntohs(cliaddr.sin_port);

    // 输出客户端的信息
    printf("client's ip is %s, and port is %d\n", cliIp, cliPort );

    // 接收客户端发来的数据
    char recvBuf[1024] = {0};
    while(1) {
        int len = recv(cfd, recvBuf, sizeof(recvBuf), 0);
        if(len == -1) {
            perror("recv");
            return -1;
        } else if(len == 0) {
            printf("客户端已经断开连接...\n");
            break;
        } else if(len > 0) {
            printf("read buf = %s\n", recvBuf);
        }

        // 小写转大写
        for(int i = 0; i < len; ++i) {
            recvBuf[i] = toupper(recvBuf[i]);
        }

        printf("after buf = %s\n", recvBuf);

        // 大写字符串发给客户端
        ret = send(cfd, recvBuf, strlen(recvBuf) + 1, 0);
        if(ret == -1) {
            perror("send");
            return -1;
        }
    }
    
    close(cfd);
    close(lfd);

    return 0;
}

```

客户端

```c
#include <stdio.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main() {

    // 创建socket
    int fd = socket(PF_INET, SOCK_STREAM, 0);
    if(fd == -1) {
        perror("socket");
        return -1;
    }

    struct sockaddr_in seraddr;
    inet_pton(AF_INET, "127.0.0.1", &seraddr.sin_addr.s_addr);
    seraddr.sin_family = AF_INET;
    seraddr.sin_port = htons(9999);

    // 连接服务器
    int ret = connect(fd, (struct sockaddr *)&seraddr, sizeof(seraddr));

    if(ret == -1){
        perror("connect");
        return -1;
    }

    while(1) {
        char sendBuf[1024] = {0};
        fgets(sendBuf, sizeof(sendBuf), stdin);

        write(fd, sendBuf, strlen(sendBuf) + 1);

        // 接收
        int len = read(fd, sendBuf, sizeof(sendBuf));
        if(len == -1) {
            perror("read");
            return -1;
        }else if(len > 0) {
            printf("read buf = %s\n", sendBuf);
        } else {
            printf("服务器已经断开连接...\n");
            break;
        }
    }

    close(fd);

    return 0;
}

```

