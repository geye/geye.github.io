# 第26章 TCP：插口选项

> 所属：TCP/IP详解 卷2：实现
> 来源：TCP/IP详解 卷2：实现

本章描述TCP的插口选项，包括getsockopt和setsockopt的使用，以及各种TCP选项的含义和设置方法。

---

## 26.1 引言

### 什么是插口选项

**插口选项（Socket Options）：**
- 可以设置和获取套接字的参数
- 控制套接字的行为
- 用getsockopt和setsockopt系统调用

> 插口选项控制套接字的行为。

---

### 为什么需要插口选项

**为什么需要插口选项：**
- 不同的应用有不同的需求
- 需要调整TCP的行为
- 需要获取TCP的状态
- 灵活配置

> 插口选项让应用可以灵活配置。

---

### 本章讨论的内容

本章讨论的内容：
- getsockopt和setsockopt
- TCP_NODELAY选项
- TCP_MAXSEG选项
- TCP_KEEPALIVE选项
- 其他TCP选项
- 插口选项的实现

> 本章讨论TCP的插口选项。

---

## 26.2 getsockopt和setsockopt

### 系统调用

**getsockopt和setsockopt：**

```c
int getsockopt(int sockfd, int level, int optname, void *optval, socklen_t *optlen);
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
```

**参数：**
- sockfd：套接字描述符
- level：选项级别（SOL_SOCKET, IPPROTO_TCP等）
- optname：选项名
- optval：选项值
- optlen：选项长度

> 两个系统调用获取和设置选项。

---

### 选项级别

**常见的选项级别：**

| 级别 | 说明 |
|------|------|
| SOL_SOCKET | 通用套接字选项 |
| IPPROTO_IP | IP层选项 |
| IPPROTO_TCP | TCP层选项 |
| IPPROTO_UDP | UDP层选项 |

> 不同层级有不同的选项。

---

### 通用套接字选项

**常见的通用套接字选项：**

| 选项 | 说明 |
|------|------|
| SO_KEEPALIVE | 保活 |
| SO_RCVBUF | 接收缓冲区大小 |
| SO_SNDBUF | 发送缓冲区大小 |
| SO_REUSEADDR | 地址复用 |
| SO_BROADCAST | 广播 |
| SO_ERROR | 获取错误 |
| SO_TYPE | 套接字类型 |

> 通用选项适用于所有套接字。

---

## 26.3 TCP_NODELAY选项

### TCP_NODELAY的作用

**TCP_NODELAY选项：**
- 禁用Nagle算法
- 小数据立即发送
- 不等待
- 降低延迟

> TCP_NODELAY禁用Nagle算法。

---

### Nagle算法回顾

**Nagle算法：**
- 防止大量小报文
- 只有一个未确认的小报文
- 等ACK回来再发下一个
- 或者等数据够一个MSS

**优点：**
- 减少小报文
- 提高效率

**缺点：**
- 增加延迟
- 交互式应用可能不爽

> Nagle算法提高效率但增加延迟。

---

### 什么时候用TCP_NODELAY

**什么时候用TCP_NODELAY：**
- 交互式应用
- 对延迟敏感
- 小数据需要立即发
- 比如Telnet、游戏等

**什么时候不用：**
- 成块数据传输
- 对效率要求高
- 延迟不敏感
- 比如文件传输

> 延迟敏感的应用用TCP_NODELAY。

---

### 设置方法

**设置TCP_NODELAY：**

```c
int nodelay = 1;
setsockopt(sockfd, IPPROTO_TCP, TCP_NODELAY, &nodelay, sizeof(nodelay));
```

**获取：**

```c
int nodelay;
socklen_t len = sizeof(nodelay);
getsockopt(sockfd, IPPROTO_TCP, TCP_NODELAY, &nodelay, &len);
```

> 用setsockopt设置。

---

## 26.4 TCP_MAXSEG选项

### TCP_MAXSEG的作用

**TCP_MAXSEG选项：**
- 获取或设置最大段大小（MSS）
- 告诉TCP每个段最大多少字节
- 影响发送的段大小

> TCP_MAXSEG控制MSS。

---

### MSS的意义

**MSS（Maximum Segment Size）：**
- TCP段最大的数据部分
- 不包含TCP首部和IP首部
- 通常是MTU - IP首部 - TCP首部
- 以太网通常是1460字节

**为什么重要：**
- 大的MSS效率高
- 首部开销比例小
- 但太大了会分片

> MSS影响效率。

---

### 设置方法

**设置TCP_MAXSEG：**

```c
int mss = 1460;
setsockopt(sockfd, IPPROTO_TCP, TCP_MAXSEG, &mss, sizeof(mss));
```

**获取：**

```c
int mss;
socklen_t len = sizeof(mss);
getsockopt(sockfd, IPPROTO_TCP, TCP_MAXSEG, &mss, &len);
```

> 可以设置和获取MSS。

---

### 注意事项

**注意事项：**
- 只能在连接建立前设置
- 连接建立后改不了
- 实际MSS可能更小（路径MTU发现）
- 不是设置多少就是多少

> MSS只是建议，实际可能更小。

---

## 26.5 TCP_KEEPALIVE选项

### TCP_KEEPALIVE的作用

**TCP_KEEPALIVE选项：**
- 设置保活的空闲时间
- 多久没数据开始保活探查
- 调整保活的参数

> TCP_KEEPALIVE控制保活时间。

---

### 保活参数

**保活的参数：**

1. **空闲时间**
   - 多久没数据开始探查
   - 默认2小时
   - 可以改

2. **探查间隔**
   - 每次探查之间的间隔
   - 默认75秒
   - 可以改

3. **探查次数**
   - 多少次没回就断开
   - 默认10次
   - 可以改

> 三个参数控制保活。

---

### 设置方法

**设置保活空闲时间：**

```c
int keepalive_time = 300;  // 5分钟
setsockopt(sockfd, IPPROTO_TCP, TCP_KEEPALIVE, &keepalive_time, sizeof(keepalive_time));
```

**注意：**
- 不同系统选项名可能不同
- 有的系统是TCP_KEEPIDLE、TCP_KEEPINTVL、TCP_KEEPCNT
- 要查具体系统的文档

> 不同系统选项名可能不同。

---

## 26.6 其他TCP选项

### TCP_CORK

**TCP_CORK选项：**
- 类似Nagle算法，但更激进
- 阻止发送小报文
- 等数据多了一起发
- 提高效率

**用途：**
- 发送大量小数据
- 先攒着，一起发
- 减少报文数量

> TCP_CORK进一步减少小报文。

---

### TCP_QUICKACK

**TCP_QUICKACK选项：**
- 快速ACK
- 不延迟ACK
- 立即发ACK
- 降低延迟

**用途：**
- 对延迟敏感
- 需要立即确认
- 但可能增加报文数量

> TCP_QUICKACK禁用延迟ACK。

---

### TCP_LINGER2

**TCP_LINGER2选项：**
- 设置FIN_WAIT_2的超时时间
- 控制TIME_WAIT的时间
- 调整连接关闭的等待时间

**用途：**
- 短连接很多
- 想减少TIME_WAIT
- 但可能有风险

> TCP_LINGER2调整关闭等待时间。

---

### TCP_INFO

**TCP_INFO选项：**
- 获取TCP连接的信息
- 状态、RTT、窗口、重传等
- 用于监控和调试

**用途：**
- 监控连接状态
- 调试问题
- 性能分析

> TCP_INFO获取连接信息。

---

## 26.7 插口选项的实现

### 内核中的实现

**内核中的实现：**
- setsockopt系统调用
- 根据级别和选项名分发
- TCP的选项由tcp_ctloutput处理
- 设置tcpcb中的字段

> 内核中实现选项的设置和获取。

---

### 选项的验证

**选项的验证：**
- 检查值是否合法
- 检查是否有权限
- 检查时机是否正确（连接前还是连接后）
- 不合法返回错误

> 设置前要验证。

---

## 26.8 小结

### getsockopt和setsockopt

1. **系统调用**
   - getsockopt：获取选项
   - setsockopt：设置选项

2. **选项级别**
   - SOL_SOCKET：通用
   - IPPROTO_IP：IP层
   - IPPROTO_TCP：TCP层

3. **通用选项**
   - SO_KEEPALIVE
   - SO_RCVBUF / SO_SNDBUF
   - SO_REUSEADDR
   - 等等

---

### TCP_NODELAY

1. **作用**
   - 禁用Nagle算法
   - 小数据立即发
   - 降低延迟

2. **适用场景**
   - 交互式应用
   - 延迟敏感

3. **设置方法**
   - setsockopt(IPPROTO_TCP, TCP_NODELAY)

---

### TCP_MAXSEG

1. **作用**
   - 设置/获取MSS
   - 最大段大小

2. **注意事项**
   - 连接前设置
   - 实际可能更小

3. **设置方法**
   - setsockopt(IPPROTO_TCP, TCP_MAXSEG)

---

### TCP_KEEPALIVE

1. **作用**
   - 设置保活参数
   - 空闲时间、间隔、次数

2. **参数**
   - 空闲时间
   - 探查间隔
   - 探查次数

3. **注意事项**
   - 不同系统选项名不同

---

### 其他TCP选项

1. **TCP_CORK**
   - 阻止小报文
   - 提高效率

2. **TCP_QUICKACK**
   - 快速ACK
   - 降低延迟

3. **TCP_LINGER2**
   - 调整关闭等待时间

4. **TCP_INFO**
   - 获取连接信息
   - 监控调试

---

### 实现

1. **内核实现**
   - setsockopt系统调用
   - tcp_ctloutput处理

2. **验证**
   - 检查合法性
   - 检查权限
   - 检查时机

---

### 关键概念

1. **插口选项**
   - 控制套接字行为
   - getsockopt/setsockopt

2. **TCP_NODELAY**
   - 禁用Nagle
   - 降低延迟

3. **TCP_MAXSEG**
   - 控制MSS
   - 影响效率

4. **TCP_KEEPALIVE**
   - 保活参数
   - 控制保活

5. **其他选项**
   - 各种选项
   - 灵活配置

---

#插口选项 
