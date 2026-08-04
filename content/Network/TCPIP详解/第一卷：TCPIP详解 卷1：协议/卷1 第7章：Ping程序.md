---
title: 第7章 Ping程序
---

# 第7章 Ping程序

## 7.1 引言

Ping程序测试另一台主机是否可达。
- 发送ICMP回显请求
- 等待ICMP回显应答
- 测量往返时间（RTT）

## 7.2 Ping程序

### ICMP回显报文格式
- 类型：8（请求）/ 0（应答）
- 标识符：进程ID
- 序列号：从0开始递增
- 数据：可选

### LAN输出示例
64 bytes from 140.252.13.34: icmp_seq=0 ttl=255 time=0 ms
64 bytes from 140.252.13.34: icmp_seq=1 ttl=255 time=0 ms

text

### WAN输出示例
64 bytes from 140.252.13.34: icmp_seq=0 ttl=248 time=380 ms

text

### 拨号SLIP链路
- V.32（9600b/s）+ V.42 + V.42bis
- RTT约300ms

## 7.3 IP记录路由选项（RR）

- -R选项启用
- 每个路由器记录出口IP地址
- 最多记录9个IP地址（IP首部限制）

### RR选项格式
- code：7
- len：最大39字节
- ptr：指向下一个可用位置

## 7.4 IP时间戳选项

- code：0x44
- 类型：0=仅时间戳，1=地址+时间戳，3=预指定
- 时间戳：UTC午夜后的毫秒数

## 7.5 小结

Ping是测试网络连通性的基本工具，使用ICMP回显请求/应答。支持记录路由和时间戳选项。