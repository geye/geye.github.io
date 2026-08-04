# 第3章 接口层

> 所属：TCP/IP详解 卷2：实现
> 来源：TCP/IP详解 卷2：实现

本章描述内核中网络接口层的实现，包括ifnet结构、接口操作、接口初始化和接口统计等。

---

## 3.1 引言

### 接口层的作用

接口层的作用：

- 统一各种网络接口的抽象
- 向上提供统一的接口
- 向下对接不同的硬件
- 屏蔽硬件差异

> 接口层是网络栈和硬件之间的桥梁。

---

### 本章讨论的内容

本章讨论的内容：

- ifnet结构
- 接口操作
- 接口初始化
- 接口统计
- 接口ioctl
- 接口队列

> 本章讨论接口层的通用部分。

---

## 3.2 ifnet结构

### 什么是ifnet

ifnet（interface network）：

- 每个网络接口一个ifnet结构
- 统一的接口抽象
- 包含接口的所有信息
- 包含操作函数指针

> ifnet是网络接口的统一抽象。

---

### ifnet结构定义

**ifnet结构的主要字段：**

```c
struct ifnet {
    char    *if_name;           /* 接口名字，如"le" */
    u_char  if_unit;            /* 单元号，如0 */
    short   if_flags;           /* 接口标志 */
    short   if_timer;           /* 定时器 */
    int     if_mtu;             /* 最大传输单元 */
    int     if_metric;          /* 度量值 */
    int     if_baudrate;        /* 波特率 */
    int     if_ipackets;        /* 接收包数 */
    int     if_ierrors;         /* 接收错误数 */
    int     if_opackets;        /* 发送包数 */
    int     if_oerrors;         /* 发送错误数 */
    int     if_collisions;      /* 冲突数 */
    int     if_ibytes;          /* 接收字节数 */
    int     if_obytes;          /* 发送字节数 */
    int     if_imcasts;         /* 接收多播数 */
    int     if_omcasts;         /* 发送多播数 */
    int     if_iqdrops;         /* 输入队列丢弃数 */
    int     if_noproto;         /* 不支持的协议数 */
    
    struct  ifaddr *if_addrlist; /* 地址链表 */
    struct  ifnet *if_next;     /* 下一个接口 */
    
    /* 函数指针 */
    int     (*if_init)(int);    /* 初始化函数 */
    int     (*if_output)(struct ifnet *, struct mbuf *, ...); /* 输出函数 */
    int     (*if_ioctl)(struct ifnet *, int, caddr_t); /* ioctl函数 */
    int     (*if_reset)(int);   /* 重置函数 */
    void    (*if_watchdog)(int); /* 看门狗函数 */
    
    /* 队列 */
    struct  ifqueue if_snd;     /* 发送队列 */
};
```

---

### 字段说明

| 字段 | 说明 |
|------|------|
| if_name | 接口名字，如"le"、"sl"、"lo" |
| if_unit | 单元号，如0、1 |
| if_flags | 接口标志 |
| if_mtu | 最大传输单元 |
| if_metric | 度量值（选路用） |
| if_addrlist | 接口地址链表 |
| if_next | 下一个接口，全局链表 |
| if_init | 初始化函数指针 |
| if_output | 输出函数指针 |
| if_ioctl | ioctl函数指针 |
| if_snd | 发送队列 |

> ifnet结构包含接口的所有信息和操作。

---

### if_flags标志

**常见的接口标志：**

| 标志 | 说明 |
|------|------|
| IFF_UP | 接口已启用 |
| IFF_BROADCAST | 支持广播 |
| IFF_DEBUG | 调试模式 |
| IFF_LOOPBACK | 环回接口 |
| IFF_POINTOPOINT | 点对点接口 |
| IFF_RUNNING | 接口运行中 |
| IFF_NOARP | 不需要ARP |
| IFF_PROMISC | 混杂模式 |
| IFF_ALLMULTI | 接收所有多播 |
| IFF_MULTICAST | 支持多播 |

> 标志描述接口的属性和状态。

---

## 3.3 接口链表

### 全局接口链表

**所有接口链接成一个全局链表：**
- ifnet链表头
- 每个接口的if_next指向下一个
- 遍历所有接口

> 全局链表管理所有接口。

---

### 接口的查找

**按名字查找接口：**
- 遍历链表
- 比较名字和单元号
- 找到返回指针

**例子：**
- "le0" → 名字"le"，单元0
- "sl1" → 名字"sl"，单元1

> 可以按名字查找接口。

---

## 3.4 接口地址

### ifaddr结构

**接口地址结构（ifaddr）：**

```c
struct ifaddr {
    struct  sockaddr *ifa_addr;    /* 接口地址 */
    struct  sockaddr *ifa_dstaddr; /* 对端地址（点对点） */
    struct  sockaddr *ifa_netmask; /* 子网掩码 */
    struct  ifnet *ifa_ifp;        /* 所属接口 */
    struct  ifaddr *ifa_next;      /* 下一个地址 */
    /* ... 其他字段 ... */
};
```

> 每个地址一个ifaddr结构。

---

### 多地址支持

**一个接口可以有多个地址：**
- if_addrlist是链表头
- 每个地址链接在链表上
- 支持多协议地址
- 支持多个IP地址

> 一个接口可以有多个地址。

---

## 3.5 接口操作

### 接口初始化

**接口初始化（if_init）：**
- 系统启动时调用
- 初始化硬件
- 初始化ifnet结构
- 注册中断处理
- 分配缓冲区

> 初始化是接口的第一个操作。

---

### 接口输出

**接口输出（if_output）：**
- 把数据发送出去
- 参数：接口、mbuf、目的地址等
- 把mbuf放到发送队列
- 启动发送
- 或直接发送

> if_output是输出的统一入口。

---

### 接口输入

**接口输入：**
- 硬件收到数据
- 触发中断
- 中断处理程序读取数据
- 封装成mbuf
- 交给上层协议处理

> 输入是从硬件到上层的过程。

---

### ioctl操作

**接口ioctl（if_ioctl）：**
- 控制接口的各种操作
- 设置地址
- 设置标志
- 获取统计
- 等等

> ioctl提供了控制接口的手段。

---

## 3.6 接口队列

### 发送队列

**发送队列（if_snd）：**
- 待发送的分组队列
- 如果接口忙，就排队
- 按顺序发送
- 流控作用

> 发送队列缓存待发送的数据。

---

### 队列操作

**队列操作：**
- 入队：把分组加到队列尾
- 出队：从队列头取分组
- 队列长度限制
- 满了就丢弃

> 队列有长度限制，防止无限增长。

---

## 3.7 接口统计

### 统计字段

**接口统计字段：**
- if_ipackets：接收包数
- if_ierrors：接收错误数
- if_opackets：发送包数
- if_oerrors：发送错误数
- if_collisions：冲突数
- if_ibytes：接收字节数
- if_obytes：发送字节数
- if_imcasts：接收多播数
- if_omcasts：发送多播数

> 各种统计信息，方便监控。

---

### netstat命令

**netstat命令：**
- 查看接口统计
- netstat -i
- 显示每个接口的统计信息

> netstat是常用的监控工具。

---

## 3.8 小结

### ifnet结构

1. **结构定义**
   - if_name：接口名字
   - if_unit：单元号
   - if_flags：标志
   - if_mtu：MTU
   - if_addrlist：地址链表
   - if_next：全局链表
   - 各种函数指针

2. **if_flags标志**
   - IFF_UP：启用
   - IFF_BROADCAST：广播
   - IFF_LOOPBACK：环回
   - IFF_POINTOPOINT：点对点
   - IFF_RUNNING：运行中
   - IFF_MULTICAST：多播

---

### 接口链表

1. **全局链表**
   - 所有接口链接在一起
   - 可以遍历所有接口

2. **接口查找**
   - 按名字查找
   - 名字+单元号

---

### 接口地址

1. **ifaddr结构**
   - ifa_addr：地址
   - ifa_dstaddr：对端地址
   - ifa_netmask：掩码
   - ifa_ifp：所属接口

2. **多地址支持**
   - 一个接口多个地址
   - 链表组织

---

### 接口操作

1. **初始化**
   - if_init
   - 硬件初始化

2. **输出**
   - if_output
   - 统一输出入口

3. **输入**
   - 中断处理
   - 向上层传递

4. **ioctl**
   - if_ioctl
   - 各种控制操作

---

### 接口队列

1. **发送队列**
   - if_snd
   - 缓存待发送数据

2. **队列操作**
   - 入队出队
   - 长度限制

---

### 接口统计

1. **统计字段**
   - 收发包数
   - 错误数
   - 字节数
   - 多播数

2. **netstat**
   - 查看统计
   - 监控接口状态

---

### 关键概念

1. **ifnet结构**
   - 统一的接口抽象
   - 包含信息和操作

2. **接口标志**
   - 描述接口属性和状态

3. **接口地址**
   - 一个接口多个地址
   - 链表组织

4. **接口操作**
   - 初始化、输入、输出、ioctl

5. **接口队列**
   - 发送队列
   - 流控作用

---

#接口层 #ifnet
