# 第7章 域和协议

> 所属：TCP/IP详解 卷2：实现
> 来源：TCP/IP详解 卷2：实现

本章描述内核中协议族（域）和协议的组织方式，包括domain结构、protosw结构、inetsw数组等。

---

## 7.1 引言

### 本章讨论的内容

本章讨论的内容：

- 域（domain）结构
- 协议切换表（protosw）
- Internet协议的protosw数组（inetsw）
- 原始IP接口
- 域的初始化

> 域和协议是内核网络栈的组织框架。

---

## 7.2 domain结构

### 什么是域

域（domain）：

- 一个协议族，如Internet协议族
- 包含该族的所有协议
- 每个域有一个domain结构
- 所有域链接成一个链表

> 域是协议族的抽象。

---

### domain结构定义

```c
struct domain {
    int     dom_family;         /* 地址族，如AF_INET */
    char    *dom_name;          /* 域名，如"internet" */
    void    (*dom_init)(void);  /* 初始化函数 */
    struct  protosw *dom_protosw; /* 协议切换表 */
    struct  protosw *dom_protoswNPROTOSW; /* 表尾指针 */
    struct  domain *dom_next;   /* 下一个域 */
    int     (*dom_rtattach)(void); /* 选路表初始化 */
    int     dom_rtoffset;       /* 选路表偏移 */
    int     dom_maxrtkey;       /* 最大选路键长 */
};
```

---

### 字段说明

| 字段 | 说明 |
|------|------|
| dom_family | 地址族标识，如AF_INET、AF_LOCAL |
| dom_name | 域名，字符串形式 |
| dom_init | 域的初始化函数指针 |
| dom_protosw | 协议切换表的起始 |
| dom_protoswNPROTOSW | 协议切换表的结束 |
| dom_next | 下一个域的指针，链表 |
| dom_rtattach | 选路表初始化函数 |
| dom_rtoffset | 选路表偏移 |
| dom_maxrtkey | 最大选路键长度 |

> 每个协议族对应一个domain结构。

---

### 常见的域

| 域 | 地址族 | 说明 |
|----|--------|------|
| inetdomain | AF_INET | Internet协议族（TCP/IP） |
| unixdomain | AF_LOCAL | 本地（Unix域）套接字 |
| routedomain | AF_ROUTE | 选路套接字 |
| linkdomain | AF_LINK | 链路层接口 |
| nsdomain | AF_NS | Xerox NS协议 |
| isodomain | AF_ISO | OSI协议 |

> 4.4BSD支持多种协议族。

---

## 7.3 protosw结构

### 什么是protosw

protosw（protocol switch）：

- 协议切换表
- 每个协议一个protosw结构
- 包含协议的所有操作函数指针
- 类似于面向对象的虚函数表

> protosw是协议的操作接口。

---

### protosw结构定义

```c
struct protosw {
    short   pr_type;            /* 插口类型 */
    struct  domain *pr_domain;  /* 所属域 */
    short   pr_protocol;        /* 协议号 */
    short   pr_flags;           /* 标志 */
    
    /* 输入输出 */
    void    (*pr_input)(int);
    int     (*pr_output)(struct mbuf *, ...);
    
    /* 控制输入输出 */
    void    (*pr_ctlinput)(int, struct sockaddr *, void *);
    int     (*pr_ctloutput)(int, struct socket *, ...);
    
    /* 用户请求 */
    int     (*pr_usrreq)(struct socket *, int, ...);
    
    /* 定时器 */
    void    (*pr_fasttimo)(void);  /* 快速超时（200ms） */
    void    (*pr_slowtimo)(void);  /* 慢速超时（500ms） */
    
    /* 其他 */
    void    (*pr_drain)(void);     /* 内存回收 */
    int     (*pr_sysctl)(int *, u_int, void *, size_t, ...);
};
```

---

### 字段说明

| 字段 | 说明 |
|------|------|
| pr_type | 插口类型，如SOCK_STREAM、SOCK_DGRAM |
| pr_domain | 所属的域 |
| pr_protocol | 协议号，如IPPROTO_TCP |
| pr_flags | 协议标志 |
| pr_input | 输入处理函数 |
| pr_output | 输出处理函数 |
| pr_ctlinput | 控制输入（如ICMP错误） |
| pr_ctloutput | 控制输出（如setsockopt） |
| pr_usrreq | 用户请求处理 |
| pr_fasttimo | 快速超时处理（200ms一次） |
| pr_slowtimo | 慢速超时处理（500ms一次） |
| pr_drain | 内存回收（内存不足时调用） |
| pr_sysctl | sysctl系统调用处理 |

---

### pr_flags标志

| 标志 | 说明 |
|------|------|
| PR_ATOMIC | 原子消息，不拆分 |
| PR_ADDR | 地址，每个消息带地址 |
| PR_CONNREQUIRED | 需要连接 |
| PR_WANTRCVD | 想要接收通知 |
| PR_RIGHTS | 支持权限传递 |

**例子：**
- TCP：PR_CONNREQUIRED | PR_WANTRCVD
- UDP：PR_ATOMIC | PR_ADDR
- ICMP：PR_ATOMIC | PR_ADDR

> 标志描述协议的特性。

---

## 7.4 inetsw数组

### Internet协议的protosw数组

Internet协议族的所有协议组织在inetsw数组中：

```c
struct protosw inetsw[] = {
    /* 0: IP层，只能由内核访问 */
    { SOCK_RAW, &inetdomain, IPPROTO_IP, PR_ATOMIC|PR_ADDR,
      ip_input, 0, ip_ctlinput, ip_ctloutput,
      ip_usrreq, 0, 0, 0, 0, ip_sysctl },
    
    /* 1: 未使用 */
    
    /* 2: TCP */
    { SOCK_STREAM, &inetdomain, IPPROTO_TCP, PR_CONNREQUIRED|PR_WANTRCVD,
      tcp_input, 0, tcp_ctlinput, tcp_ctloutput,
      tcp_usrreq, tcp_fasttimo, tcp_slowtimo, tcp_drain, tcp_sysctl },
    
    /* 3: UDP */
    { SOCK_DGRAM, &inetdomain, IPPROTO_UDP, PR_ATOMIC|PR_ADDR,
      udp_input, 0, udp_ctlinput, udp_ctloutput,
      udp_usrreq, 0, 0, 0, udp_sysctl },
    
    /* 4: ICMP */
    { SOCK_RAW, &inetdomain, IPPROTO_ICMP, PR_ATOMIC|PR_ADDR,
      icmp_input, 0, icmp_ctlinput, icmp_ctloutput,
      rip_usrreq, 0, 0, 0, 0 },
    
    /* ... 其他协议 ... */
};
```

---

### 数组索引

**inetsw数组的索引：**
- 0：IP层（raw，内核用）
- 1：未使用
- 2：TCP
- 3：UDP
- 4：ICMP
- ... 其他协议

> 索引和协议号不完全对应，需要查找。

---

### 原始IP接口

**原始IP（Raw IP）：**
- SOCK_RAW类型
- 允许进程直接发送和接收IP分组
- 不经过传输层处理
- 需要特殊权限

**用途：**
- ping、traceroute等诊断工具
- 实现用户态协议
- 安全扫描工具
- 网络研究和实验

> 原始套接字给用户态直接访问IP层的能力。

---

## 7.5 域的初始化

### 初始化过程

**域初始化的步骤：**
1. 系统启动时，初始化网络子系统
2. 遍历所有域
3. 调用每个域的dom_init
4. 初始化协议数据结构
5. 注册定时器
6. 初始化选路表

**inetdomain的初始化：**
- 初始化IP层
- 初始化TCP
- 初始化UDP
- 初始化ICMP
- 初始化IGMP
- 等等

> 每个域自己负责初始化自己的协议。

---

### 协议切换表的查找

**查找protosw的函数：**
```c
struct protosw *
pffindproto(domain, protocol, type)
    struct domain *domain;
    int protocol;
    int type;
{
    /*
     * 在域的protosw数组中查找
     * 匹配协议号和类型
     */
}
```

**使用场景：**
- socket系统调用：根据域、类型、协议找protosw
- IP分用：根据协议号找传输层协议
- 各种协议操作

> 通过pffindproto找到对应的协议。

---

## 7.6 小结

### domain结构

1. **结构定义**
   - dom_family：地址族
   - dom_name：域名
   - dom_init：初始化函数
   - dom_protosw：协议切换表
   - dom_next：链表指针

2. **常见的域**
   - AF_INET：Internet
   - AF_LOCAL：Unix域
   - AF_ROUTE：选路
   - AF_LINK：链路层

---

### protosw结构

1. **结构定义**
   - pr_type：插口类型
   - pr_protocol：协议号
   - pr_flags：标志
   - 各种函数指针

2. **函数指针**
   - pr_input：输入处理
   - pr_output：输出处理
   - pr_usrreq：用户请求
   - pr_fasttimo / pr_slowtimo：定时器
   - pr_drain：内存回收
   - pr_sysctl：系统控制

3. **pr_flags标志**
   - PR_ATOMIC：原子消息
   - PR_ADDR：带地址
   - PR_CONNREQUIRED：需要连接
   - PR_WANTRCVD：接收通知

---

### inetsw数组

1. **数组结构**
   - IP层
   - TCP
   - UDP
   - ICMP
   - 其他协议

2. **原始IP接口**
   - SOCK_RAW
   - 直接访问IP层
   - 用于诊断和实验

---

### 域的初始化

1. **初始化过程**
   - 系统启动时
   - 调用每个域的dom_init
   - 初始化协议数据结构

2. **协议查找**
   - pffindproto函数
   - 根据域、协议号、类型查找
   - 用于socket系统调用等

---

### 关键概念

1. **域（domain）**
   - 协议族的抽象
   - 包含多个协议
   - 链表组织

2. **协议切换表（protosw）**
   - 每个协议一个
   - 函数指针表
   - 统一的接口

3. **inetsw数组**
   - Internet协议的protosw数组
   - IP、TCP、UDP、ICMP等
   - 原始IP接口

4. **协议查找**
   - pffindproto
   - 根据域、类型、协议号查找

---

#TCP/IP #卷2 #实现 #域 #协议 #domain #protosw #inetsw #原始IP #内核
