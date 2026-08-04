## 卷1 第5章：RARP：逆地址解析协议

```markdown
---
title: 第5章 RARP：逆地址解析协议
---

# 第5章 RARP：逆地址解析协议

## 5.1 引言

RARP（Reverse Address Resolution Protocol）用于无盘系统获取IP地址。
- 硬件地址→IP地址
- 广播RARP请求
- RARP服务器应答

## 5.2 RARP的分组格式

- 以太网帧类型：0x8035
- 操作码：3=请求，4=应答
- 其他字段与ARP相同

## 5.3 RARP举例
rarp who-is 8:0:20:3:f6:42 tell 8:0:20:3:f6:42
rarp reply 8:0:20:3:f6:42 at sun

text

- RARP请求是广播
- RARP应答是单播
- 收到IP地址后发送TFTP请求加载引导映像

## 5.4 RARP服务器的设计

### 作为用户进程
- 内核不提供RARP服务
- 需要读取/etc/ethers文件
- 需要发送/接收特殊以太网帧

### 多个RARP服务器
- 提供冗余备份
- 可能增加网络流量
- 客户端使用最先收到的应答

## 5.5 小结

RARP用于无盘系统获取IP地址。RARP服务器实现与系统相关，通常作为用户进程运行。