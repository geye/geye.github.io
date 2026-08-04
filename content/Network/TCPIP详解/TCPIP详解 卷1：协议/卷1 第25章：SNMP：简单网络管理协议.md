---
title: 第25章 SNMP：简单网络管理协议
---

# 第25章 SNMP：简单网络管理协议

## 25.1 引言

基于TCP/IP的网络管理包含两部分：
- **管理进程（Manager）**：通常在工作站上运行
- **代理进程（Agent）**：在被管设备上运行

### 三个组成部分
1. **管理信息库（MIB）**：可查询和修改的参数
2. **管理信息结构（SMI）**：MIB的公用结构和表示符号
3. **简单网络管理协议（SNMP）**：通信协议

## 25.2 协议

### 五种操作
| 操作 | 方向 |
|------|------|
| get-request | Manager → Agent |
| get-next-request | Manager → Agent |
| set-request | Manager → Agent |
| get-response | Agent → Manager |
| trap | Agent → Manager |

### 端口号
- Manager → Agent：UDP 161
- Agent → Manager（trap）：UDP 162

### SNMP报文格式
- 版本（0 = SNMPv1）
- 共同体（明文口令，默认public）
- PDU类型（0-4）
- 请求标识
- 差错状态
- 差错索引
- 变量绑定（名称+值）

## 25.3 管理信息结构

### 数据类型
| 类型 | 说明 |
|------|------|
| INTEGER | 整数 |
| OCTET STRING | 字节串 |
| DisplayString | ASCII字符串（≤255） |
| OBJECT IDENTIFIER | 对象标识 |
| NULL | 空值 |
| IpAddress | 4字节IP地址 |
| PhysAddress | 物理地址 |
| Counter | 非负计数器 |
| Gauge | 非负值（可增可减） |
| TimeTicks | 时间计数器 |
| SEQUENCE | 结构 |
| SEQUENCE OF | 表 |

## 25.4 对象标识符

- 树型结构
- 从根开始：`1.3.6.1.2.1` = `iso.org.dod.internet.mgmt.mib`
- 叶子节点是可操作的

## 25.5 管理信息库介绍

### MIB-II分组
- system（系统）
- interfaces（接口）
- at（地址转换）
- ip（IP）
- icmp（ICMP）
- tcp（TCP）
- udp（UDP）

### UDP组示例
| 变量 | 说明 |
|------|------|
| udpInDatagrams | 输入数据报数 |
| udpNoPorts | 端口不可达数 |
| udpInErrors | 输入差错数 |
| udpOutDatagrams | 输出数据报数 |
| udpTable | UDP监听表 |

## 25.6 实例标识

### 简单变量
- 在对象标识后添加 `.0`
- 示例：`udpInDatagrams.0`

### 表格
- 使用索引值标识行
- 示例：`udpLocalAddress.0.0.0.0.67`

## 25.7 一些简单的例子

### get操作
snmpi> get udpInDatagrams.0
udpInDatagrams.0 = 42

text

### get-next操作
- 基于字典式排序
- 可遍历整个MIB

### 表格访问
- 使用get-next遍历
- 按"先列后行"顺序

## 25.8 管理信息库（续）

### system组
| 变量 | 说明 |
|------|------|
| sysDescr | 系统描述 |
| sysObjectID | 厂商标识 |
| sysUpTime | 运行时间 |
| sysContact | 联系人 |
| sysName | 主机名 |
| sysLocation | 位置 |
| sysServices | 服务值 |

### interface组
- ifNumber：接口数
- ifTable：接口表（22列）

### ip组
- ipForwarding：转发标志
- ipDefaultTTL：默认TTL
- ipAddrTable：IP地址表
- ipRouteTable：IP路由表
- ipNetToMediaTable：地址转换表

### icmp组
- 输入/输出计数器
- 各种ICMP报文类型计数器

### tcp组
- tcpRtoAlgorithm：RTO算法
- tcpRtoMin/Max：RTO范围
- tcpActiveOpens/PassiveOpens
- tcpCurrEstab：当前连接数
- tcpConnTable：TCP连接表

## 25.9 其他一些例子

### 接口MTU
1. 查询路由表获取接口索引
2. 查询接口表获取MTU

### 路由表
- 使用SNMP查看路由表
- 分析路由决策

## 25.10 Trap

### Trap类型
| 类型 | 说明 |
|------|------|
| 0 | coldStart |
| 1 | warmStart |
| 2 | linkDown |
| 3 | linkUp |
| 4 | authenticationFailure |
| 5 | egpNeighborLoss |
| 6 | enterpriseSpecific |

## 25.11 ASN.1和BER

- **ASN.1**：描述数据的正式语言
- **BER**：编码方法
- 实现SNMP时需要

## 25.12 SNMPv2

### 改进
1. get-bulk-request：高效读取大块数据
2. inform-request：管理进程间通信
3. 更好的安全性（鉴别和加密）

## 25.13 小结

SNMP是网络管理协议，使用MIB定义可管理变量。get-next操作可遍历MIB树。trap用于主动通知。
