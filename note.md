# sockaddr

```C
struct sockaddr {
  sa_family_t		sa_family;	/* address family, AF_xxx	*/
  char			sa_data[14];	/* 14 bytes of protocol address	*/
};
```

16 bytes

is a generic descriptor for any kind of socket under 16bytes.

# sockaddr_storage

```C
struct sockaddr_storage {
  sa_family_t		ss_family; //根据ss_family来解释结构体内的编码结构
  char			_ss_pad1[_SS_PAD1SIZE];
  int64_t		__ss_align;
  char			_ss_pad2[_SS_PAD2SIZE];
};
```

128 bytes

is a generic descriptor for any kind of socket large enough.

# sockaddr_in

```C
struct sockaddr_in
{
  sa_family_t	 sin_family;	/* Address family		*/
  in_port_t	 sin_port;	/* Port number			*/
  struct in_addr sin_addr;	/* Internet address		*/

  /* Pad to size of `struct sockaddr'. */
  unsigned char  __pad[__SOCK_SIZE__ - sizeof(short int)
			- sizeof(unsigned short int) - sizeof(struct in_addr)];
};
```

16 bytes

is a struct specific to IP-based communication (IIRC, "in" stands for "InterNet").

# scokaddr_in6

```C
struct sockaddr_in6
{
  sa_family_t	  sin6_family;		/* AF_INET6 */
  in_port_t	  sin6_port;		/* Port number. */
  uint32_t	  sin6_flowinfo;	/* Traffic class and flow inf. */
  struct in6_addr sin6_addr;		/* IPv6 address. */
  uint32_t	  sin6_scope_id;	/* Set of interfaces for a scope. */
};
```

28 bytes

sockaddr_in for IPv6.

# 创建 socket

```C
int socket (int family, int socktype, int protocol)
```

**family**: $\color{Red}{IP协议版本}$

- AF_INET || PF_INET: IPv4

- AF_INET6 || PF_INET6: IPv6

**socktype**: $\color{Red}{TCP/UDP模式}$

- SOCK_STREAM: TCP

- SOCK_DGRAM: UDP

**protocol**: 一般都是 0

**return**: $\color{orange}{int}$

- 成功：file descriptor for the new socket

- 错误：-1

# socket 与地址

## bind

```C
int bind (int socket, struct sockaddr *addr, socklen_t length)
```

把指定 $\color{CornflowerBlue}{socket}$ 接到指定的网络地址 $\color{CornflowerBlue}{*addr}$ 上，长度为 $\color{CornflowerBlue}{length}$ （可以理解为插座的形状）。

## getsockname

```C
int getsockname (int socket, struct sockaddr *addr, socklen_t *length-ptr)
```
通过指定 $\color{CornflowerBlue}{socket}$ 标识符，获得指定的网络地址信息到 $\color{CornflowerBlue}{*addr}$ 里，长度为 $\color{CornflowerBlue}{length-ptr}$。长度需要自己定义然后转换为 $\color{CornflowerBlue}{(struct~sockaddr *)}$

## getaddrinfo

## freeaddrinfo

TODO

# 监听&接受

## listen

```C
int listen (int socket, int n)
```

服务器角色，监听 $\color{CornflowerBlue}{socket}$ 标识符的链接请求，最多 n 个，超过的将返回拒绝错误。

## accept

```C
int accept (int socket, struct sockaddr *addr, socklen_t *length_ptr)
```

监听到 $\color{CornflowerBlue}{socket}$ 链接之后，便要接受链接；这里的 地址 $\color{CornflowerBlue}{*addr}$ 和 长度 $\color{CornflowerBlue}{*length-ptr}$ 存放的是客户端的信息。

# 客户端连接服务端

```C
int connect (int socket, struct sockaddr *addr, socklen_t length)
```

把本地的 $\color{CornflowerBlue}{socket}$ 接到远程的 $\color{CornflowerBlue}{*addr}$ 上，长度为 $\color{CornflowerBlue}{length}$

?->会选择一个临时端口与服务端通信<-?。

一般情况下阻塞等待，也可以设置为不阻塞。

**return**：成功 0，失败-1

# 非阻塞I/O
## select
```C
int select (int nfds, fd_set *read-fds, fd_set *write-fds, fd_set *except-fds, struct timeval *timeout)
```
$\color{CornflowerBlue}{nfds}$ 一共有多少个要读写的文件

$\color{CornflowerBlue}{*read-fds}$、 $\color{CornflowerBlue}{*write-fds}$ 检查要读的文件集合是否已经准备好被读，要写的文件集合是否已准备好被写。 $\color{CornflowerBlue}{*except-fds}$ 检查特殊情况，不论是读、写、特殊检查，不需要的情况下都可以置空 **NULL**

$\color{CornflowerBlue}{*timeout}$ 等待的时间，0为永远等待



# TCP 读写

connect 建立连接后读写

## send

```C
ssize_t send (int socket, const void *buffer, size_t size, int flags)
//flags 可以设置同步异步阻塞等。一般可以是0，默认阻塞
```

往 $\color{CornflowerBlue}{socket}$ 里写 $\color{CornflowerBlue}{*buffer}$ 的内容，长度为 $\color{CornflowerBlue}{size}$

## recv

```C
ssize_t recv (int socket, void *buffer, size_t size, int flags)
```

从 $\color{CornflowerBlue}{socket}$ 里读 $\color{CornflowerBlue}{*buffer}$ 的内容，长度为 $\color{CornflowerBlue}{size}$

例子

```C
// Server side C/C++ program to demonstrate Socket
// programming
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>
#define PORT 8080
int main(int argc, char const* argv[])
{
    int server_fd, new_socket, valread;
    struct sockaddr_in address;
    int opt = 1;
    int addrlen = sizeof(address);
    char buffer[1024] = { 0 };
    char* hello = "Hello from server";
 
    // Creating socket file descriptor
    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }
 
    // Forcefully attaching socket to the port 8080
    if (setsockopt(server_fd, SOL_SOCKET,
                   SO_REUSEADDR | SO_REUSEPORT, &opt,
                   sizeof(opt))) {
        perror("setsockopt");
        exit(EXIT_FAILURE);
    }
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);
 
    // Forcefully attaching socket to the port 8080
    if (bind(server_fd, (struct sockaddr*)&address,
             sizeof(address))
        < 0) {
        perror("bind failed");
        exit(EXIT_FAILURE);
    }
    if (listen(server_fd, 3) < 0) {
        perror("listen");
        exit(EXIT_FAILURE);
    }
    if ((new_socket
         = accept(server_fd, (struct sockaddr*)&address,
                  (socklen_t*)&addrlen))
        < 0) {
        perror("accept");
        exit(EXIT_FAILURE);
    }
    valread = read(new_socket, buffer, 1024);
    printf("%s\n", buffer);
    send(new_socket, hello, strlen(hello), 0);
    printf("Hello message sent\n");
 
    // closing the connected socket
    close(new_socket);
    // closing the listening socket
    shutdown(server_fd, SHUT_RDWR);
    return 0;
}
```
```C
// Client side C/C++ program to demonstrate Socket
// programming
#include <arpa/inet.h>
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>
#define PORT 8080
 
int main(int argc, char const* argv[])
{
    int sock = 0, valread, client_fd;
    struct sockaddr_in serv_addr;
    char* hello = "Hello from client";
    char buffer[1024] = { 0 };
    if ((sock = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        printf("\n Socket creation error \n");
        return -1;
    }
 
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);
 
    // Convert IPv4 and IPv6 addresses from text to binary
    // form
    if (inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr)
        <= 0) {
        printf(
            "\nInvalid address/ Address not supported \n");
        return -1;
    }
 
    if ((client_fd
         = connect(sock, (struct sockaddr*)&serv_addr,
                   sizeof(serv_addr)))
        < 0) {
        printf("\nConnection Failed \n");
        return -1;
    }
    send(sock, hello, strlen(hello), 0);
    printf("Hello message sent\n");
    valread = read(sock, buffer, 1024);
    printf("%s\n", buffer);
 
    // closing the connected socket
    close(client_fd);
    return 0;
}
```

# UDP 读写

不建立链接，读写需要写明目标目的地

## sendto

```C
//写|发送
ssize_t sendto (int socket, const void *buffer, size_t size, int flags, struct sockaddr *addr, socklen_t length)
```

通过 $\color{CornflowerBlue}{socket}$ 发送大小为 $\color{CornflowerBlue}{size}$ 的 $\color{CornflowerBlue}{*buffer}$ 内容，到长度为 $\color{CornflowerBlue}{length}$ 的目标 $\color{CornflowerBlue}{*addr}$ ，忽略 flags；

returns the number of bytes transmitted, or -1 on failure

## recvfrom

```C
//读|接收
ssize_t recvfrom (int socket, void *buffer, size_t size, int flags, struct sockaddr *addr, socklen_t *length-ptr)
```

通过 $\color{CornflowerBlue}{socket}$ 接收大小为 $\color{CornflowerBlue}{size}$ 的 $\color{CornflowerBlue}{*buffer}$ 内容，到长度为 $\color{CornflowerBlue}{*length-ptr}$ 的目标 $\color{CornflowerBlue}{*addr}$ ，忽略 flags；

需要手动申请和释放多线程的资源；

returns the number of bytes received, or -1 on failure.

For a socket in the local domain the address information won’t be meaningful, since you can’t read the address of such a socket (see The Local Namespace). You can specify a null pointer as the addr argument if you are not interested in this information.

# 通用读写

所有的 socket 都可以发送或接收至已使用 connect 设置目的地（UDP 也可以），可以用 write 和 read 是因为 socket 在 unix 系统下就是文件，还可以免除写 flag

```C
//写
ssize_t send (int socket, const void *buffer, size_t size, int flags) //socket
ssize_t write (int filedes, const void *buffer, size_t size) //file

//读
ssize_t recv (int socket, void *buffer, size_t size, int flags) //socket
ssize_t read (int filedes, void *buffer, size_t size) //file
```

# 为什么用strlen()来获取内容长度？
`sizeof` 可以得到固定数据的整个长度，不论里面有什么

`strlen()` 是当读到一个空字符 `'\0'` 的时候就结束

# 数据转换
## 主机到网络
统一、一致的大小端格式。

htonl()

ntohl()

ioctl()

atoi()

### ???
inet_ntop()

# 多线程
pthread_create

pthread_mutex_lock

pthread_mutex_unlock

# 考试题目

## 函数链式关系
```cpp
passiveTCP(service, qlen) 
    => passivesock(service, "tcp", qlen)

passiveUDP(service) 
    => passivesock(service, "udp", 0)

passivesock(service,...,qlen) 
    => getaddrinfo(...,service) 
        => bind(...) 
        => listen(...,qlen)

passivesock(char *service, char *transport, int qlen)
//service 服务名称
//transport "udp"/"tcp"
//qlen 最大监听连接数
```

第五题：
1. service, "tcp", QLEN `//需传入listen最大连接数，见函数链式关系`
2. mastsocket, (struct sockaddr *)&fromsin, &alen 

    `//accept一个访问主socket的链接请求，并把详细信息记录在第二第三个形参中; 由于fromsin的数据结构和api中的不同，所以强转为(struct sockaddr *)&`

3. (void* (*)(void\*))TCPechod `//返回void的函数的指针，接受一个void* 参数，该callback是子线程的运行函数`
4. (void *)slavesock `//传入答案3中callback的参数，应是子sock`
5. slavesock `//关闭子进程的子sock`
6. sockd, buf, sizeof buf `//echo读取字符串，读了几个nchars就是几`
7. sockd, buf, nchars   `//nchars读了几个就返回几个`
8. service, "udp", 0 `//like 答案1`
9. sockd, buf, sizeof(buf), 0, (struct sockaddr *)&fromsin, &alen 

    ```cpp
    recvfrom(
        int sock标识,
        void* 接受数据, size_t 数据长度,
        int 特殊flag,
        struct sockaddr* 源地址, socklen_t* 源址长度
    )
    ```
10. sockd, buf, sizeof(buf), 0, (struct sockaddr *)&fromsin, alen
    ```cpp
    sendto(
        int sock标识,
        void* 发送数据, size_t 数据长度,
        int 特殊flag,
        struct sockaddr* 目标地址, socklen_t 地址长度
    )
    ```

$\color{orange}{接收API的\,>源地址、长度<\,都为指针}$

$\color{orange}{发送API的\,>长度<\, \color{red}{不为指针}}$
***
第四题
1. NULL `//getaddrinfo的第一个参数是获取目标地址的socket info`
2. servname `//第二个参数是根据指定服务（端口号/字符串）获取socket info`
3. result->ai_family, result->ai_socktype, result->ai_protocol `//根据1、2的函数返回的result的socket信息创建socket`
4. passivesocket, result->ai_addr, result->ai_addrlen `//将result的地址信息绑定在passivesocket上`
5. hints.ai_socktype == SOCK_STREAM && listen(passivesocket, qlen) < 0 `//如果是可以接受的tcp类型，绑定完成之后就要监听，放在if中如果失败则处理错误`
6. close(passivesocket) `//关闭不需要的socket`