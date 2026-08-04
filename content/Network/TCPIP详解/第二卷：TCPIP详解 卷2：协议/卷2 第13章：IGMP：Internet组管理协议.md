---
title: 卷2 第13章 IGMP：Internet组管理协议
---

# 卷2 第13章 IGMP：Internet组管理协议

## 13.1 引言

IGMP在本地网络传播组成员信息。
- 协议号：2
- 封装在IP中

## 13.2 代码介绍

- `netinet/igmp.h`：协议定义
- `netinet/igmp_var.h`：实现定义
- `netinet/in_var.h`：多播结构
- `netinet/igmp.c`：实现

## 13.3 igmp结构

```c
struct igmp {
    u_char igmp_type;    // 1=查询，2=报告
    u_char igmp_code;    // 未使用
    u_short igmp_cksum;
    struct in_addr igmp_group;
};
报文类型
类型	说明
0x11	成员关系查询
0x12	成员关系报告
0x13	DVMRP
13.4 IGMP的protosw结构
类型：SOCK_RAW

协议：IPPROTO_IGMP（2）

输入：igmp_input

13.5 加入组：igmp_joingroup
发送成员报告

设置随机定时器

13.6 igmp_fasttimo函数
每秒调用5次

递减组定时器

超时发送报告

igmp_sendreport
构造IGMP报告

发送到多播组

13.7 输入处理：igmp_input
查询处理
设置随机延迟

延迟后发送报告

报告处理
取消该组定时器

避免重复报告

13.8 离开组：igmp_leavegroup
不发送离开通知

下次查询时不再报告

13.9 小结
IGMP管理多播组成员，使用查询/报告机制。报告延迟减少网络负载。