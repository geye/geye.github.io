# 第4章 接口：以太网

> 所属：TCP/IP详解 卷2：实现
> 来源：TCP/IP详解 卷2：实现

本章详细描述以太网接口的实现，包括LANCE以太网驱动、接口初始化、输入输出处理等。

---

## 4.1 引言

### 以太网接口的作用

以太网接口的作用：

- 连接IP层和物理网络
- 发送和接收以太网帧
- 处理ARP
- 提供接口统计

> 以太网是最常用的局域网技术。

---

### 本章讨论的内容

本章讨论的内容：

- LANCE以太网驱动
- 接口初始化
- 以太网输出
- 以太网输入
- 接口统计

> 以LANCE芯片为例介绍以太网驱动。

---

## 4.2 LANCE以太网驱动

### 什么是LANCE

LANCE（Local Area Network Controller for Ethernet）：

- AMD生产的以太网控制器芯片
- 经典的以太网控制器
- 很多系统使用

> 本书用LANCE作为例子。

---

### le_softc结构

每个以太网接口有一个le_softc结构：

```c
struct le_softc {
    struct  arpcom sc_ac;    /* 通用ARP结构（包含ifnet） */
    struct  le_init_block sc_ib;  /* 初始化块 */
    /* ... 其他硬件相关字段 ... */
};
```

- sc_ac：arpcom结构，包含ifnet和以太网地址
- 第一个成员是arpcom，可以强制类型转换

> arpcom结构是以太网接口的通用部分。

---

### arpcom结构

arpcom结构：

```c
struct arpcom {
    struct  ifnet ac_if;     /* 通用接口结构 */
    u_char  ac_enaddr[6];    /* 以太网地址 */
    /* ... ARP相关 ... */
};
```

- ac_if：ifnet结构，所有接口通用
- ac_enaddr：6字节以太网地址

> 所有以太网接口都有arpcom结构。

---

## 4.3 接口初始化

### leattach函数

leattach函数：以太网接口初始化

```c
void
leattach(unit)
    int unit;
{
    struct le_softc *sc;
    struct ifnet *ifp;

    /* 分配并初始化le_softc */
    sc = &le_softc[unit];
    ifp = &sc->sc_ac.ac_if;

    /* 设置ifnet结构的函数指针 */
    ifp->if_init = leinit;
    ifp->if_output = ether_output;
    ifp->if_start = lestart;
    ifp->if_ioctl = leioctl;
    ifp->if_reset = lereset;

    /* 设置接口标志 */
    ifp->if_flags = IFF_BROADCAST | IFF_SIMPLEX | IFF_MULTICAST;

    /* 调用if_attach注册接口 */
    if_attach(ifp);

    /* 初始化硬件 */
    leinit(unit);
}
```

---

### if_attach函数

if_attach函数：将接口加入系统的接口链表

```c
void
if_attach(ifp)
    struct ifnet *ifp;
{
    /* 将接口加入ifnet链表 */
    ifp->if_next = ifnet;
    ifnet = ifp;
    
    /* 分配链路层地址 */
    /* ... */
}
```

> 所有接口都在ifnet链表中。

---

### leinit函数

leinit函数：初始化LANCE硬件

```c
static void
leinit(unit)
    int unit;
{
    struct le_softc *sc = &le_softc[unit];
    
    /*
     * 设置初始化块
     * 配置接收和发送描述符
     */
     
    /*
     * 启动芯片
     */
     
    /*
     * 设置接口为UP状态
     */
    sc->sc_ac.ac_if.if_flags |= IFF_UP;
}
```

---

## 4.4 以太网输出

### ether_output函数

ether_output函数：以太网输出的通用函数

```c
int
ether_output(ifp, m0, dst, rt0)
    struct ifnet *ifp;
    struct mbuf *m0;
    struct sockaddr *dst;
    struct rtentry *rt0;
{
    struct ether_header *eh;
    struct mbuf *m;
    
    /*
     * 1. 在mbuf前面预留以太网首部空间
     */
    M_PREPEND(m, sizeof(struct ether_header), M_DONTWAIT);
    
    /*
     * 2. 根据目的IP地址查找以太网地址（ARP）
     */
     
    /*
     * 3. 构造以太网首部
     */
    eh = mtod(m, struct ether_header *);
    /* 设置目的地址、源地址、类型 */
    
    /*
     * 4. 调用接口的if_start发送
     */
    (*ifp->if_start)(ifp);
    
    return 0;
}
```

---

### 以太网首部

以太网首部结构：

```c
struct ether_header {
    u_char  ether_dhost[6];   /* 目的地址 */
    u_char  ether_shost[6];   /* 源地址 */
    u_short ether_type;       /* 类型 */
};
```

- 目的地址：6字节
- 源地址：6字节
- 类型：2字节，标识上层协议

常见的类型值：
- 0x0800：IP
- 0x0806：ARP
- 0x8035：RARP

---

### lestart函数

lestart函数：LANCE的发送函数

```c
static void
lestart(ifp)
    struct ifnet *ifp;
{
    struct le_softc *sc = (struct le_softc *)ifp;
    struct mbuf *m;
    
    /*
     * 从接口输出队列取一个分组
     */
    IF_DEQUEUE(&ifp->if_snd, m);
    
    /*
     * 设置发送描述符
     * 启动发送
     */
     
    /*
     * 如果队列还有数据，设置标志稍后再发
     */
}
```

---

### 输出流程

以太网输出的完整流程：

```
应用层
   |
   v
TCP/UDP
   |
   v
IP层
   |
   v
ether_output
   |
   +---> ARP查找地址
   |
   v
加入接口输出队列
   |
   v
lestart（硬件发送）
   |
   v
物理网络
```

---

## 4.5 以太网输入

### 中断处理

以太网接收是中断驱动的：

1. 网卡收到一个帧
2. 产生中断
3. CPU调用中断处理函数
4. 中断处理函数读取帧
5. 放入输入队列
6. 触发软中断
7. 软中断中调用ether_input

> 中断级别做最少的工作，主要处理在软中断。

---

### leintr函数

leintr函数：LANCE中断处理

```c
int
leintr(unit)
    int unit;
{
    struct le_softc *sc = &le_softc[unit];
    
    /*
     * 检查中断原因
     */
     
    /*
     * 处理接收中断
     * - 从接收描述符取数据
     * - 封装成mbuf
     * - 加入输入队列
     */
     
    /*
     * 处理发送完成中断
     * - 释放mbuf
     * - 继续发送队列中的下一个
     */
     
    /*
     * 触发软中断
     */
    schednetisr(NETISR_ETH);
}
```

---

### ether_input函数

ether_input函数：以太网输入处理（软中断中调用）

```c
void
ether_input(ifp, m)
    struct ifnet *ifp;
    struct mbuf *m;
{
    struct ether_header *eh;
    u_short type;
    
    /*
     * 提取以太网首部
     */
    eh = mtod(m, struct ether_header *);
    type = ntohs(eh->ether_type);
    
    /*
     * 根据类型分用
     */
    switch (type) {
    case ETHERTYPE_IP:
        /* 交给IP */
        schednetisr(NETISR_IP);
        break;
        
    case ETHERTYPE_ARP:
        /* 交给ARP */
        arpinput(ifp, m);
        break;
        
    /* ... 其他类型 ... */
    }
}
```

---

### 输入流程

以太网输入的完整流程：

```
物理网络
   |
   v
网卡接收
   |
   v
硬件中断（leintr）
   |
   v
放入输入队列
   |
   v
软中断（NETISR_ETH）
   |
   v
ether_input
   |
   +---> IP输入（NETISR_IP）
   |
   +---> ARP输入
   |
   +---> 其他协议
```

---

## 4.6 接口统计

### ifnet中的统计字段

ifnet结构中的统计字段：

```c
struct ifnet {
    /* ... */
    
    /* 接收统计 */
    u_long  if_ipackets;   /* 接收分组数 */
    u_long  if_ierrors;    /* 接收错误数 */
    u_long  if_ibytes;     /* 接收字节数 */
    
    /* 发送统计 */
    u_long  if_opackets;   /* 发送分组数 */
    u_long  if_oerrors;    /* 发送错误数 */
    u_long  if_obytes;     /* 发送字节数 */
    
    /* 其他统计 */
    u_long  if_collisions; /* 冲突数 */
    /* ... */
};
```

---

### 常见的错误类型

接收错误类型：

- CRC错误
- 帧过长
- 帧过短
- 对齐错误
- 缓冲区溢出

发送错误类型：

- 载波丢失
- 冲突过多
- 延迟冲突
- 下溢

---

## 4.7 小结

### LANCE驱动结构

1. **le_softc结构**
   - 每个接口一个
   - 包含arpcom和硬件相关字段

2. **arpcom结构**
   - 以太网通用部分
   - 包含ifnet和以太网地址

3. **ifnet结构**
   - 所有接口通用
   - 函数指针、统计、标志等

---

### 接口初始化

1. **leattach**
   - 设置函数指针
   - 设置接口标志
   - 调用if_attach注册

2. **if_attach**
   - 加入接口链表
   - 分配链路层地址

3. **leinit**
   - 初始化硬件
   - 设置描述符
   - 启动芯片

---

### 以太网输出

1. **ether_output**
   - 通用输出函数
   - ARP查找地址
   - 构造以太网首部
   - 调用if_start

2. **lestart**
   - 硬件相关发送
   - 从输出队列取数据
   - 设置发送描述符

3. **以太网首部**
   - 6字节目的地址
   - 6字节源地址
   - 2字节类型

---

### 以太网输入

1. **中断处理（leintr）**
   - 硬件中断
   - 读取数据
   - 放入输入队列
   - 触发软中断

2. **ether_input**
   - 软中断中处理
   - 解析以太网首部
   - 根据类型分用

3. **分用**
   - IP → NETISR_IP
   - ARP → arpinput
   - 其他 → 对应协议

---

### 接口统计

1. **接收统计**
   - 分组数、错误数、字节数

2. **发送统计**
   - 分组数、错误数、字节数

3. **其他统计**
   - 冲突数等

---

### 关键概念

1. **以太网接口**
   - 连接IP层和物理网络
   - 发送接收以太网帧

2. **arpcom结构**
   - 以太网接口通用结构
   - 包含ifnet和MAC地址

3. **ether_output**
   - 通用输出函数
   - ARP + 构造首部 + 发送

4. **ether_input**
   - 通用输入函数
   - 分用上层协议

5. **中断 + 软中断**
   - 中断做最少工作
   - 软中断做主要处理

---

#以太网接口 #LANCE 
