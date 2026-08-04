---
title: 卷2 第11章 ICMP：Internet控制报文协议
---

# 卷2 第11章 ICMP：Internet控制报文协议

## 11.1 引言

ICMP传递差错和管理报文，是IP的必要部分。

## 11.2 代码介绍

- `netinet/ip_icmp.h`：ICMP结构
- `netinet/ip_icmp.c`：ICMP处理

## 11.3 icmp结构

```c
struct icmp {
    u_char icmp_type;
    u_char icmp_code;
    u_short icmp_cksum;
    union {
        // 各种类型专用
    } icmp_hun;
};
```

## 11.4 ICMP的protosw结构
类型：SOCK_RAW

协议：IPPROTO_ICMP（1）

输入：icmp_input

## 11.5 输入处理：icmp_input函数
验证步骤
验证长度

验证检验和

检查类型

差错处理
映射到PRC_常量

调用协议控制输入函数

请求处理
回显：转换为回显应答

时间戳：记录时间戳

地址掩码：检查配置

## 11.6 重定向处理
更新路由表

通知所有协议

icmp_redirect处理
验证重定向

查找匹配路由

更新网关

## 11.7 输出处理
icmp_error函数
构造ICMP差错

包含原始数据报首部+8字节

检查不产生差错的条件

icmp_reflect函数
交换源/目的地址

逆转源路由

发送应答

## 11.8 小结
ICMP处理差错和查询报文。内核处理回显、时间戳和地址掩码请求。