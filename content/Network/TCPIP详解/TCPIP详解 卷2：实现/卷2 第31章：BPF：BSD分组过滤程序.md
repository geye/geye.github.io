---
title: 卷2 第31章 BPF：BSD分组过滤程序
---

# 卷2 第31章 BPF：BSD分组过滤程序

## 31.1 引言

BPF（BSD Packet Filter）用于截获网络接口的数据流。

### 用途
- tcpdump
- 网络监控
- 流量分析

## 31.2 代码介绍

- `net/bpf.h`：BPF常量
- `net/bpfdesc.h`：BPF结构
- `net/bpf.c`：BPF实现

## 31.3 bpf_if结构

| 成员 | 说明 |
|------|------|
| bif_next | 链表指针 |
| bif_dlist | 描述符链表 |
| bif_driverp | 接口BPF指针 |
| bif_dlt | 链路类型 |
| bif_ifp | 接口指针 |

### 链路类型
| 类型 | 说明 |
|------|------|
| DLT_EN10MB | 以太网 |
| DLT_SLIP | SLIP |
| DLT_NULL | 环回 |

## 31.4 bpf_d结构

| 成员 | 说明 |
|------|------|
| bd_next | 链表 |
| bd_sbuf | 存储缓存 |
| d_buf | 存储缓存数据 |
| bd_hbuf | 暂留缓存 |
| bd_hbuf | 暂留缓存数据 |
| bd_bufsize | 缓存大小 |
| bd_bif | 接口结构 |
| bd_filter | 过滤程序 |
| bd_rcount | 接收计数 |
| bd_dcount | 丢弃计数 |

### ioctl命令
| 命令 | 说明 |
|------|------|
| BIOCSETF | 安装过滤 |
| BIOCSETIF | 设置接口 |
| BIOCPROMISC | 混杂模式 |
| BIOCGBLEN | 缓存大小 |
| BIOCSBLEN | 设置缓存 |
| BIOCIMMEDIATE | 立即模式 |
| BIOCSRTIMEOUT | 超时 |

## 31.5 BPF的输入

### bpf_tap函数
- 设备驱动程序调用
- 遍历BPF设备列表
- 应用过滤器

### catchpacket函数
1. 检查缓存空间
2. 轮转缓存（如需要）
3. 添加BPF首部
4. 复制数据
5. 唤醒进程

### bpf_read函数
- 等待数据
- 从暂留缓存复制

## 31.6 BPF的输出

### bpf_write函数
1. 复制数据到mbuf
2. 构造链路层首部
3. 调用if_output

## 31.7 小结

BPF提供用户级网络分组捕获。支持过滤、缓存和混杂模式。
