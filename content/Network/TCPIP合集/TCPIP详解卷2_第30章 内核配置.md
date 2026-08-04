# 第30章 内核配置

> 所属：TCP/IP详解 卷2：实现
> 来源：TCP/IP详解 卷2：实现

本章描述TCP/IP内核的配置，包括编译时配置、运行时调整、接口配置、选路配置和性能调优。

---

## 30.1 引言

### 内核配置的重要性

内核配置：

- 决定内核包含哪些功能
- 影响性能和功能
- 可以编译时配置，也可以运行时调整
- 根据需求定制内核

> 合理的配置可以提高性能和安全性。

---

### 本章讨论的内容

本章讨论的内容：

- 内核配置选项
- 编译时配置
- 运行时调整（sysctl）
- 接口配置
- 选路配置
- 性能调优

> 本章介绍TCP/IP相关的内核配置。

---

## 30.2 内核配置选项

### TCP/IP相关选项

**TCP/IP相关的内核配置选项：**

| 选项 | 说明 |
|------|------|
| INET | 启用Internet协议族 |
| GATEWAY | 启用IP转发（路由器模式） |
| IPFORWARDING | IP转发开关（默认值） |
| SUBNETSARELOCAL | 子网是否算本地 |
| DIRECTED_BROADCAST | 允许定向广播 |
| MROUTING | 多播选路 |
| NSIP | XNS over IP封装 |
| ISO, NS, CCITT | 其他协议族 |
| COMPAT_43 | 4.3BSD兼容 |
| NFS | 网络文件系统 |

> 可以根据需要选择编译哪些功能。

---

### INET选项

**INET：启用Internet协议族**
- 必须的，要使用TCP/IP就得开
- 包含IP、ICMP、TCP、UDP、IGMP等
- 最基本的选项

> INET是TCP/IP的总开关。

---

### GATEWAY选项

**GATEWAY：启用IP转发**
- 路由器需要开
- 主机不需要开
- 开启后可以转发IP数据报
- 影响性能（转发需要处理）

> GATEWAY让系统成为路由器。

---

### IPFORWARDING

**IPFORWARDING：IP转发默认值**
- 设置ipforwarding变量的默认值
- 可以运行时用sysctl修改
- 0：不转发（主机）
- 1：转发（路由器）

> 运行时可以修改，不用重新编译。

---

### 其他选项

**SUBNETSARELOCAL：子网是否算本地**
- 影响UDP校验和等
- 通常开着

**DIRECTED_BROADCAST：定向广播**
- 是否允许定向广播
- 安全考虑可以关
- 防止广播攻击

**MROUTING：多播选路**
- 多播路由器需要
- 普通主机不需要

> 根据实际需求选择。

---

## 30.3 编译时配置

### 内核配置文件

**内核配置文件：**
- 通常在/sys/conf/目录
- 如GENERIC、MYKERNEL等
- 文本格式
- 用config命令处理

> 配置文件定义内核包含哪些功能。

---

### 配置文件格式

**配置文件的格式示例：**

```
#
# 我的内核配置
#
machine         i386
cpu             "I386_CPU"
ident           MYKERNEL

# 标准系统选项
options         INET                    # Internet协议
options         GATEWAY                 # IP转发
options         NFS                     # NFS客户端
options         COMPAT_43               # 4.3BSD兼容

# 伪设备
pseudo-device   loop                    # 环回接口
pseudo-device   pty                     # 伪终端

# 设备
device          le0                     # LANCE以太网
device          sl0                     # SLIP接口
```

> 配置文件用options和device等关键字。

---

### 编译过程

**内核编译的步骤：**

1. **编写配置文件**
   - 选择需要的选项和设备
   - 保存到配置文件

2. **config配置**
   - 运行config命令
   - 生成编译用的文件
   - 创建编译目录

3. **make depend**
   - 生成依赖关系
   - 确保编译顺序正确

4. **make**
   - 编译内核
   - 生成内核文件

5. **安装新内核**
   - 复制到/
   - 重启使用新内核

> 编译内核需要几个步骤。

---

### 编译时配置的优缺点

**优点：**
- 可以去掉不需要的功能
- 内核更小，占用内存少
- 可能更快
- 更安全（减少攻击面）

**缺点：**
- 需要重新编译
- 需要重启
- 比较麻烦
- 容易出错

> 编译时配置功能强，但麻烦。

---

## 30.4 运行时调整

### sysctl系统调用

**sysctl：运行时调整内核参数**
- 不需要重新编译
- 不需要重启
- 即时生效
- 很方便

> sysctl是运行时调优的主要方式。

---

### sysctl的层级

**sysctl的层级结构：**

```
顶层
  |
  +-- kern   内核相关
  |
  +-- vm     虚拟内存
  |
  +-- net    网络相关
  |     |
  |     +-- inet   Internet
  |     |     |
  |     |     +-- ip     IP层
  |     |     +-- icmp   ICMP
  |     |     +-- tcp    TCP
  |     |     +-- udp    UDP
  |     |
  |     +-- unix   Unix域
  |     +-- route  选路
  |
  +-- fs     文件系统
  |
  +-- hw     硬件
```

> sysctl是树状结构。

---

### 常见的网络sysctl变量

**常见的网络相关sysctl变量：**

| 变量 | 说明 |
|------|------|
| net.inet.ip.forwarding | IP转发开关 |
| net.inet.ip.ttl | 默认TTL |
| net.inet.tcp.sendspace | TCP发送缓冲区大小 |
| net.inet.tcp.recvspace | TCP接收缓冲区大小 |
| net.inet.udp.checksum | UDP校验和开关 |
| net.inet.udp.sendspace | UDP发送缓冲区 |
| net.inet.udp.recvspace | UDP接收缓冲区 |
| net.inet.icmp.maskrepl | ICMP掩码回复 |

> 很多参数都可以用sysctl调整。

---

### sysctl的使用

**使用sysctl：**
- 读取：sysctl 变量名
- 设置：sysctl 变量名=值
- 需要root权限才能修改

**示例：**
```bash
# 查看IP转发状态
sysctl net.inet.ip.forwarding

# 开启IP转发
sysctl net.inet.ip.forwarding=1
```

> sysctl命令很方便。

---

## 30.5 接口配置

### 接口配置

**接口配置：**
- 配置IP地址
- 配置子网掩码
- 配置广播地址
- 启用/禁用接口
- 设置其他参数

> 接口配置是最基本的网络配置。

---

### ifconfig命令

**ifconfig：接口配置命令**

**常用命令：**
```bash
# 查看接口状态
ifconfig le0

# 设置IP地址和子网掩码
ifconfig le0 192.168.1.1 netmask 255.255.255.0

# 启用接口
ifconfig le0 up

# 禁用接口
ifconfig le0 down

# 设置广播地址
ifconfig le0 broadcast 192.168.1.255
```

> ifconfig是最常用的接口配置工具。

---

### 接口配置的系统调用

**底层的系统调用：**
- ioctl(SIOCSIFADDR)：设置接口地址
- ioctl(SIOCSIFNETMASK)：设置子网掩码
- ioctl(SIOCSIFFLAGS)：设置接口标志
- ioctl(SIOCGIFCONF)：获取接口列表
- ioctl(SIOCGIFADDR)：获取接口地址

> ifconfig底层用ioctl实现。

---

## 30.6 选路配置

### 选路配置

**选路配置：**
- 添加静态路由
- 删除路由
- 修改路由
- 配置默认路由
- 配置动态路由

> 选路配置决定数据怎么走。

---

### route命令

**route：选路配置命令**

**常用命令：**
```bash
# 添加路由
route add -net 192.168.2.0 192.168.1.1

# 添加主机路由
route add -host 192.168.1.100 192.168.1.1

# 添加默认路由
route add default 192.168.1.1

# 删除路由
route delete -net 192.168.2.0

# 查看路由表
netstat -rn
```

> route命令管理静态路由。

---

### 动态路由

**动态路由：**
- 路由守护进程
- 运行路由协议
- 自动学习路由
- 自动更新

**常见的路由守护进程：**
- routed：RIP协议
- gated：支持多种协议（RIP、OSPF、BGP等）

**常见的路由协议：**
- RIP：距离矢量，简单
- OSPF：链路状态，复杂但强大
- BGP：边界网关，互联网用

> 动态路由适合复杂网络。

---

## 30.7 性能调优

### 性能调优的原则

**性能调优的基本原则：**

1. **先测量，再调优**
   - 先知道瓶颈在哪
   - 不要盲目调优
   - 用工具测量

2. **一次改一个参数**
   - 方便对比效果
   - 容易定位问题
   - 不要一次改很多

3. **记录改动和效果**
   - 记录改了什么
   - 记录效果如何
   - 方便回滚

4. **不要盲目调优**
   - 默认值通常是合理的
   - 调优不一定有效果
   - 可能有副作用

> 调优要谨慎，要有依据。

---

### 常见的调优参数

**常见的性能调优参数：**

| 参数 | 说明 | 影响 |
|------|------|------|
| tcp_sendspace | TCP发送缓冲区 | 影响最大窗口、吞吐量 |
| tcp_recvspace | TCP接收缓冲区 | 影响最大窗口、吞吐量 |
| udp_sendspace | UDP发送缓冲区 | 影响最大数据报大小 |
| udp_recvspace | UDP接收缓冲区 | 影响接收队列大小 |
| ipforwarding | IP转发 | 路由器功能 |
| ttl | 默认TTL | 跳数限制 |

> 根据场景调整不同的参数。

---

### TCP缓冲区调优

**TCP缓冲区大小：**
- 影响最大窗口大小
- 影响吞吐量
- 带宽延迟积越大，需要的缓冲区越大

**计算公式：**
```
缓冲区大小 = 带宽 × 延迟
```

**示例：**
- 带宽：100Mbps = 12.5MB/s
- 延迟：100ms = 0.1s
- 带宽延迟积 = 12.5MB/s × 0.1s = 1.25MB
- 缓冲区需要至少1.25MB

> TCP缓冲区影响最大吞吐量。

---

### 其他调优

**其他调优方面：**
- 接口队列长度
- 选路表大小
- mbuf数量
- 内核内存
- 中断处理

> 调优是系统工程，涉及很多方面。

---

## 30.8 小结

### 内核配置选项

1. **主要选项**
   - INET：Internet协议
   - GATEWAY：IP转发
   - IPFORWARDING：转发默认值
   - 其他：MROUTING、DIRECTED_BROADCAST等

2. **选择原则**
   - 根据需要选择
   - 不需要的可以去掉
   - 减小内核体积

---

### 编译时配置

1. **配置文件**
   - options和device
   - 文本格式
   - config命令处理

2. **编译过程**
   - config → make depend → make → 安装
   - 需要重启

3. **优缺点**
   - 优点：内核小、性能好、安全
   - 缺点：麻烦、需要重启

---

### 运行时调整

1. **sysctl**
   - 运行时修改
   - 不需要重启
   - 树状结构

2. **常见变量**
   - net.inet.ip.forwarding
   - net.inet.tcp.sendspace
   - net.inet.tcp.recvspace
   - 等等

3. **使用方式**
   - sysctl 变量名 查看
   - sysctl 变量名=值 设置

---

### 接口配置

1. **ifconfig命令**
   - 设置IP地址
   - 设置子网掩码
   - 启用/禁用接口

2. **底层ioctl**
   - SIOCSIFADDR
   - SIOCSIFNETMASK
   - SIOCSIFFLAGS

---

### 选路配置

1. **静态路由**
   - route命令
   - 添加/删除/修改

2. **动态路由**
   - 路由守护进程
   - RIP、OSPF、BGP等协议

---

### 性能调优

1. **调优原则**
   - 先测量再调优
   - 一次改一个
   - 记录改动
   - 不盲目调优

2. **常见参数**
   - TCP发送/接收缓冲区
   - UDP发送/接收缓冲区
   - TTL、转发等

3. **TCP缓冲区**
   - 影响吞吐量
   - 带宽延迟积
   - 合理设置大小

---

### 关键概念

1. **内核配置**
   - 编译时配置
   - 运行时调整
   - 各有优缺点

2. **sysctl**
   - 运行时调优
   - 方便灵活
   - 树状结构

3. **接口配置**
   - ifconfig
   - ioctl

4. **选路配置**
   - 静态路由
   - 动态路由

5. **性能调优**
   - 测量优先
   - 谨慎调整
   - TCP缓冲区很重要

---

#内核配置 #sysctl 
