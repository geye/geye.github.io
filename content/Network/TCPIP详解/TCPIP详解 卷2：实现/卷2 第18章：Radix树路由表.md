---
title: 卷2 第18章 Radix树路由表
---

# 卷2 第18章 Radix树路由表

## 18.1 引言

Net/3使用Patricia树（Radix树）实现路由表。

### 优势
- 快速查找
- 支持变长地址（OSI）
- 支持主机和网络路由

## 18.2 路由表结构

### 路由标志
| 标志 | 说明 |
|------|------|
| U | 可用 |
| G | 网关（间接） |
| H | 主机路由 |
| S | 静态 |
| C | 可克隆 |
| L | 链路层地址 |

### 路由表项示例
Destination Gateway Flags Refs Use Interface
127.0.0.1 127.0.0.1 UH 0 0 lo0
default sun UG 0 0 le0
140.252.13.32 link#1 UC 0 0 le0
140.252.13.33 0:0:c0:6f:2d:40 UHL 0 0 le0

text

## 18.3 选路插口

- PF_ROUTE域
- 进程通过原始插口发送/接收选路消息

## 18.4 代码介绍

- `net/radix.h`：Radix结点
- `net/raw_cb.h`：选路控制块
- `net/route.h`：选路结构
- `net/radix.c`：Radix函数
- `net/route.c`：选路函数
- `net/rtsock.c`：选路插口

## 18.5 Radix结点数据结构

### radix_node_head
| 成员 | 说明 |
|------|------|
| rnh_treetop | 树顶 |
| rnh_addaddr | 添加函数 |
| rnh_deladdr | 删除函数 |
| rnh_matchaddr | 匹配函数 |

### radix_node
| 成员 | 说明 |
|------|------|
| rn_mklist | 掩码链表 |
| rn_p | 父结点 |
| rn_b | 测试比特（负值=叶子） |
| rn_l/rn_r | 左/右孩子 |

### radix_mask
- 存储网络掩码
- 共享相同掩码

## 18.6 选路结构

### route结构
| 成员 | 说明 |
|------|------|
| ro_rt | 路由表项指针 |
| ro_dst | 目的地址 |

### rtentry结构
| 成员 | 说明 |
|------|------|
| rt_nodes[2] | Radix结点 |
| rt_flags | 标志 |
| rt_gateway | 网关地址 |
| rt_refcnt | 引用计数 |
| rt_use | 使用计数 |
| rt_ifp | 接口指针 |
| rt_ifa | 地址指针 |
| rt_metrics | 度量 |

### 度量结构
| 成员 | 说明 |
|------|------|
| rmx_mtu | MTU |
| rmx_rtt | RTT |
| rmx_rttvar | RTT偏差 |
| rmx_ssthresh | 慢启动门限 |
| rmx_expire | 超时 |

## 18.7 初始化

- `route_init`：初始化选路
- `rtable_init`：初始化各域路由表
- `rn_inithead`：创建Radix树

## 18.8 重复键和掩码列表

- 重复键：多个表项相同键
- 掩码列表：支持不同掩码

## 18.9 rn_match函数

### 查找过程
1. 沿树向下搜索
2. 精确匹配检查
3. 网络掩码匹配
4. 回溯到上层
5. 返回最佳匹配

## 18.10 rn_search函数

- 从指定结点开始搜索
- 返回叶子结点

## 18.11 小结

Radix树支持高效路由查找。rtentry结构存储路由信息。
