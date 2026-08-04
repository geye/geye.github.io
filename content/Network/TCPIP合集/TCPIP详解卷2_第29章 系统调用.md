# 第29章 系统调用

> 所属：TCP/IP详解 卷2：实现
> 来源：TCP/IP详解 卷2：实现

本章描述套接字相关的系统调用在内核中的实现，包括socket、bind、listen、accept、connect、发送接收等。

---

## 29.1 引言

### 套接字系统调用

套接字系统调用：

- 用户态和内核态的接口
- 应用程序通过系统调用使用网络功能
- 内核实现具体的协议逻辑
- 统一的API，不同协议都可以用

> 系统调用是应用和内核之间的接口。

---

### 主要的系统调用

**主要的套接字系统调用：**

| 调用 | 功能 |
|------|------|
| socket | 创建套接字 |
| bind | 绑定本地地址 |
| listen | 监听连接（TCP） |
| accept | 接受连接（TCP） |
| connect | 发起连接 |
| send / sendto / sendmsg | 发送数据 |
| recv / recvfrom / recvmsg | 接收数据 |
| close | 关闭套接字 |
| getsockname / getpeername | 获取地址 |
| getsockopt / setsockopt | 选项控制 |
| ioctl | I/O控制 |
| select / poll | 多路复用 |
| shutdown | 部分关闭 |
| read / write | 读写（流式套接字） |

> 套接字API有很多系统调用。

---

## 29.2 socket系统调用

### socket的作用

**socket：创建套接字**
```c
int socket(int domain, int type, int protocol);
```

- domain：地址族（AF_INET等）
- type：套接字类型（SOCK_STREAM、SOCK_DGRAM等）
- protocol：协议号（通常0，默认）
- 返回：文件描述符，失败返回-1

> 创建一个套接字，返回文件描述符。

---

### 内核处理过程

**socket系统调用的内核处理：**

```
应用调用socket
   |
   v
系统调用进入内核
   |
   v
分配文件描述符
   |
   v
创建file结构
   |
   v
创建socket结构
   |
   v
根据domain和type查找protosw
   |
   v
调用pr_usrreq(PRU_ATTACH)
   |
   v
创建协议控制块（inpcb等）
   |
   v
返回文件描述符
```

---

### socket结构

**socket结构：**

```c
struct socket {
    short   so_type;        /* 类型 */
    short   so_options;     /* 选项 */
    short   so_linger;      /* 延迟关闭时间 */
    short   so_state;       /* 状态 */
    caddr_t so_pcb;         /* 协议控制块指针 */
    struct  protosw *so_proto; /* 协议切换表 */
    struct  sockbuf so_snd; /* 发送缓冲区 */
    struct  sockbuf so_rcv; /* 接收缓冲区 */
    /* ... 其他字段 ... */
};
```

**字段说明：**
- so_type：套接字类型
- so_options：套接字选项
- so_state：状态
- so_pcb：指向协议控制块（TCP的tcpcb、UDP的inpcb等）
- so_proto：指向protosw结构
- so_snd / so_rcv：发送和接收缓冲区

> socket结构是套接字的通用部分。

---

### 协议控制块

**协议控制块（PCB）：**
- 每个协议自己的控制结构
- TCP：tcpcb（包含inpcb）
- UDP：inpcb
- so_pcb指向它
- 包含协议相关的状态

> 协议控制块是协议相关的部分。

---

## 29.3 bind系统调用

### bind的作用

**bind：绑定本地地址**
```c
int bind(int s, struct sockaddr *name, int namelen);
```

- s：套接字描述符
- name：本地地址
- namelen：地址长度
- 返回：0成功，-1失败

> 给套接字绑定一个本地地址。

---

### 内核处理

**bind系统调用的内核处理：**

1. **验证参数**
   - 验证描述符
   - 验证地址
   - 验证长度

2. **检查端口**
   - 端口是否被占用
   - 是否需要特殊权限（<1024）
   - 地址是否可用

3. **分配端口**
   - 如果端口为0，自动分配
   - 从临时端口范围分配

4. **设置地址**
   - 设置本地地址
   - 更新inpcb
   - 加入绑定列表

> 绑定地址和端口。

---

### 端口分配

**端口的分类：**

| 范围 | 名称 | 说明 |
|------|------|------|
| 0-1023 | 知名端口 | 需要root权限 |
| 1024-49151 | 注册端口 | 注册使用 |
| 49152-65535 | 动态/私有端口 | 自动分配 |

**临时端口分配：**
- bind时端口为0
- 内核自动分配
- 从动态端口范围选
- 确保不冲突

> 临时端口自动分配。

---

## 29.4 listen和accept

### listen的作用

**listen：开始监听**
```c
int listen(int s, int backlog);
```

- s：套接字描述符
- backlog：连接队列长度
- 返回：0成功，-1失败

**作用：**
- 把套接字设置为监听状态
- 准备接受连接
- 只能用于TCP（SOCK_STREAM）

> 监听来自客户端的连接。

---

### accept的作用

**accept：接受连接**
```c
int accept(int s, struct sockaddr *addr, int *addrlen);
```

- s：监听套接字描述符
- addr：返回对端地址
- addrlen：地址长度
- 返回：新的套接字描述符

**作用：**
- 从已完成连接队列取一个连接
- 创建新的套接字
- 返回新的文件描述符
- 默认阻塞

> 接受一个新的连接。

---

### 连接队列

**TCP的连接队列：**

```
客户端                  服务器
   |                      |
   |---- SYN ----------->|  未完成连接队列（SYN_RCVD）
   |                      |
   |<--- SYN+ACK --------|
   |                      |
   |---- ACK ----------->|  已完成连接队列（ESTABLISHED）
   |                      |
   |                      |  accept取出
   |                      |
```

**两个队列：**
1. **未完成连接队列**（incomplete connection queue）
   - 已收到SYN，已发SYN+ACK
   - 等待ACK
   - SYN_RCVD状态

2. **已完成连接队列**（completed connection queue）
   - 已完成三次握手
   - ESTABLISHED状态
   - 等待accept取走

> backlog参数限制已完成队列的长度。

---

## 29.5 connect系统调用

### connect的作用

**connect：发起连接**
```c
int connect(int s, struct sockaddr *name, int namelen);
```

- s：套接字描述符
- name：对端地址
- namelen：地址长度
- 返回：0成功，-1失败

> 发起一个连接。

---

### TCP的connect

**TCP的connect过程：**

1. **分配本地端口**
   - 如果没bind，自动分配

2. **发送SYN**
   - 构造SYN报文
   - 发送出去

3. **进入SYN_SENT状态**
   - 等待SYN+ACK

4. **等待完成**
   - 收到SYN+ACK
   - 发送ACK
   - 进入ESTABLISHED
   - 返回成功

> TCP的connect是有连接的。

---

### UDP的connect

**UDP的connect：**
- 不发送任何数据
- 只是记录对端地址
- 之后可以用send/recv（不用sendto/recvfrom）
- 可以收到异步错误（ICMP错误）

**UDP connect的作用：**
- 简化编程
- 可以收到异步错误
- 提高效率（不用每次指定地址）

> UDP的connect只是记录地址，不发数据。

---

## 29.6 发送和接收

### 发送数据

**发送数据的系统调用：**

| 调用 | 说明 |
|------|------|
| send | 已连接套接字，不需要地址 |
| sendto | 可以指定目的地址 |
| sendmsg | 最通用，支持控制信息 |

**send的参数最少，sendmsg最通用。**

---

### 内核发送处理

**发送数据的内核处理：**

1. **从用户空间复制数据**
   - 复制到内核缓冲区
   - 封装成mbuf链

2. **调用协议层**
   - 调用pr_usrreq(PRU_SEND)
   - 协议层处理

3. **向下传递**
   - TCP/UDP处理
   - IP层处理
   - 接口输出

> 数据从用户空间到内核，再向下传递。

---

### 接收数据

**接收数据的系统调用：**

| 调用 | 说明 |
|------|------|
| recv | 已连接套接字 |
| recvfrom | 可以获取源地址 |
| recvmsg | 最通用，支持控制信息 |

---

### 内核接收处理

**接收数据的内核处理：**

1. **数据到达**
   - 从网络接口收到
   - 逐层向上传递

2. **放入接收缓冲区**
   - 放入socket的接收缓冲区
   - 唤醒等待的进程

3. **进程读取**
   - 进程调用recv
   - 从接收缓冲区取数据
   - 复制到用户空间

> 数据从网络到内核缓冲区，再到用户空间。

---

## 29.7 其他系统调用

### getsockname / getpeername

**getsockname：获取本地地址**
```c
int getsockname(int s, struct sockaddr *name, int *namelen);
```

**getpeername：获取对端地址**
```c
int getpeername(int s, struct sockaddr *name, int *namelen);
```

**用途：**
- 获取自动分配的端口
- 获取对端地址
- 验证连接

> 获取套接字的地址信息。

---

### getsockopt / setsockopt

**getsockopt：获取套接字选项**
```c
int getsockopt(int s, int level, int optname, void *optval, int *optlen);
```

**setsockopt：设置套接字选项**
```c
int setsockopt(int s, int level, int optname, const void *optval, int optlen);
```

**选项层级：**
- SOL_SOCKET：套接字层
- IPPROTO_TCP：TCP层
- IPPROTO_IP：IP层

> 获取和设置各种选项。

---

### ioctl

**ioctl：I/O控制**
```c
int ioctl(int d, unsigned long request, ...);
```

**常见的网络相关ioctl：**
- SIOCGIFCONF：获取接口列表
- SIOCGIFADDR：获取接口地址
- SIOCSIFADDR：设置接口地址
- SIOCGIFNETMASK：获取子网掩码
- SIOCSIFNETMASK：设置子网掩码
- SIOCSIFFLAGS：设置接口标志
- SIOCADDRT：添加路由
- SIOCDELRT：删除路由
- ... 还有很多

> ioctl功能很多，各种控制操作。

---

### select / poll

**select：多路复用**
```c
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

**poll：多路复用**
```c
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

**作用：**
- 等待多个套接字就绪
- 提高并发效率
- 单进程处理多个连接

> 多路复用提高并发能力。

---

### shutdown

**shutdown：部分关闭**
```c
int shutdown(int s, int how);
```

**how的取值：**
- SHUT_RD：关闭读
- SHUT_WR：关闭写
- SHUT_RDWR：都关闭

**作用：**
- 可以只关读或只关写
- 半关闭
- TCP的半关闭用这个

> 可以部分关闭套接字。

---

## 29.8 小结

### 套接字系统调用

1. **主要的系统调用**
   - socket、bind、listen、accept
   - connect、send、recv、close
   - getsockname、getpeername
   - getsockopt、setsockopt
   - ioctl、select、shutdown

2. **统一的API**
   - 不同协议都用同样的接口
   - 应用程序不用关心底层协议
   - 灵活性高

---

### socket系统调用

1. **作用**
   - 创建套接字
   - 返回文件描述符

2. **内核处理**
   - 分配描述符
   - 创建socket结构
   - 查找protosw
   - 创建协议控制块

3. **socket结构**
   - 通用部分
   - so_pcb指向协议控制块
   - 发送接收缓冲区

---

### bind系统调用

1. **作用**
   - 绑定本地地址和端口

2. **端口分配**
   - 知名端口：需要权限
   - 注册端口
   - 动态端口：自动分配

---

### listen和accept

1. **listen**
   - 设置监听状态
   - backlog参数

2. **accept**
   - 接受连接
   - 返回新的描述符

3. **连接队列**
   - 未完成连接队列
   - 已完成连接队列
   - backlog限制已完成队列

---

### connect系统调用

1. **TCP的connect**
   - 发送SYN
   - 三次握手
   - 有连接

2. **UDP的connect**
   - 只记录地址
   - 不发数据
   - 可以收异步错误

---

### 发送和接收

1. **发送**
   - send / sendto / sendmsg
   - 用户空间 → 内核 → 网络

2. **接收**
   - recv / recvfrom / recvmsg
   - 网络 → 内核缓冲区 → 用户空间

---

### 其他系统调用

1. **getsockname / getpeername**
   - 获取本地/对端地址

2. **getsockopt / setsockopt**
   - 获取/设置选项

3. **ioctl**
   - 各种控制操作
   - 接口、路由等

4. **select / poll**
   - 多路复用
   - 提高并发

5. **shutdown**
   - 部分关闭
   - 半关闭

---

### 关键概念

1. **系统调用**
   - 用户态和内核态的接口
   - 统一的API

2. **socket结构**
   - 通用部分
   - 协议控制块

3. **连接队列**
   - 未完成和已完成
   - backlog参数

4. **发送接收**
   - 用户空间和内核空间之间复制
   - 缓冲区

5. **多路复用**
   - select / poll
   - 单进程多连接

---

#TCP/IP #卷2 #实现 #系统调用 #socket #bind #listen #accept #connect #send #recv #内核
