---
title: 23 UDP通信
date: 2022-05-27 21:02:46
tags: linux环境编程
---



# UDP通信





<img src="/images/image/UDP.png" style="zoom:50%;" />

```c

#include <sys/types.h>
#include <sys/socket.h>


ssize_t sendto(int sockfd, const void *buf, size_t len, int flags, const struct sockaddr *dest_addr, socklen_t addrlen);
    - 参数:
        - sockfd : 通信的fd
        - buf : 发送数据的数组
        - len : 发送数据的长度
        - flags: 标志，一般用0
        - dest_addr : 目的地址
        - addrlen : dest_addr的大小
    - 返回值:
        - -1 : 失败
        - 成功发送的字节数

ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);
    - 参数:
        - sockfd : 通信的fd
        - buf : 接收数据的数组
        - len : 接收数据数组的大小
        - flags : 0
        - src_addr : 传出参数, 源地址, 不需要可以指定为NULL
        - addrlen : 源地址的大小
    

```

```c
// UDP 服务端

#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <string.h>

int main(){

    // 创建socket
    int fd = socket(PF_INET, SOCK_DGRAM, 0);
    if (fd == -1){
        perror("socket");
        exit(-1);
    }

    // 绑定
    struct sockaddr_in saddr;
    saddr.sin_family = AF_INET;
    saddr.sin_port = htons(9999);
    saddr.sin_addr.s_addr = INADDR_ANY;
    int ret = bind(fd, (struct sockaddr*)&saddr, sizeof saddr);
    if (ret == -1){
        perror("bind");
        exit(-1);
    }

    // 通信
    while (1){
        
        // 接收数据
        char buf[128] = {0};
        char ipbuf[16] = {0};
        struct sockaddr_in src_addr;
        int len = sizeof src_addr;
        ret = recvfrom(fd, buf, sizeof buf, 0, (struct sockaddr*) &src_addr, &len);
        if (ret == -1){
            perror("recvfrom");
            exit(-1);
        }
        
        // 打印客户端的信息
        printf("client IP : %s, client port : %d\n", inet_ntop(AF_INET, &src_addr.sin_addr.s_addr, ipbuf, sizeof ipbuf), ntohs(src_addr.sin_port));

        // 打印接收的数据
        printf("recv data : %s\n", buf);

        // 发送数据
        sendto(fd, buf, strlen(buf) + 1, 0, (struct sockaddr*)&src_addr, len);

    }
    
    close(fd);
    return 0;
}
```

```c
// UDP 客户端

#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <string.h>

int main(){

    // 创建socket
    int fd = socket(PF_INET, SOCK_DGRAM, 0);
    if (fd == -1){
        perror("socket");
        exit(-1);
    }

    // 服务端地址信息
    struct sockaddr_in saddr;
    saddr.sin_family = AF_INET;
    saddr.sin_port = htons(9999);
    inet_pton(AF_INET, "0.0.0.0", &saddr.sin_addr.s_addr);  

    // 通信
    while (1){
        char buf[128] = {0};
        char ipbuf[16] = {0};
        // 发送数据
        fgets(buf, sizeof buf, stdin);
        sendto(fd, buf, strlen(buf) + 1, 0, (struct sockaddr*)&saddr, sizeof saddr);

        // 接收数据
        struct sockaddr_in saddr;
        int len = sizeof saddr;
        int ret = recvfrom(fd, buf, sizeof buf, 0, NULL, NULL);
        if (ret == -1){
            perror("recvfrom");
            exit(-1);
        }

        // 打印接收的数据
        printf("recv data : %s\n", buf);

    }
    
    close(fd);
    return 0;
}
```

