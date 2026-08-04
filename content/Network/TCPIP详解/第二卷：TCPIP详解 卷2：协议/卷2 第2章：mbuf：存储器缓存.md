---
title: 卷2 第2章 mbuf：存储器缓存
---

# 卷2 第2章 mbuf：存储器缓存

## 2.1 引言

mbuf（memory buffer）是内核中的存储器缓存，用于：
- 保存用户数据
- 保存源/目标地址
- 保存插口选项
- 保存协议控制块

## 2.2 代码介绍

- `sys/mbuf.h`：mbuf结构和宏定义
- `kern/uipc_mbuf.c`：mbuf函数

### 统计量
```bash
netstat -m
2.3 mbuf的定义
常量
常量	值	说明
MSIZE	128	mbuf大小
MLEN	108	普通mbuf数据量
MHLEN	100	分组首部mbuf数据量
MINCLSIZE	208	簇最小数据量
MCLBYTES	2048	簇大小
2.4 mbuf结构
四种mbuf类型
无标志：108字节数据

M_PKTHDR：100字节数据 + 分组首部

M_EXT：簇（外部缓存）

M_PKTHDR + M_EXT：分组首部 + 簇

成员
成员	说明
m_next	链中下一个mbuf
m_nextpkt	队列中下一个记录
m_len	数据长度
m_data	数据指针
m_type	数据类型
m_flags	标志
标志
标志	说明
M_BCAST	链路层广播
M_MCAST	链路层多播
M_EXT	有簇
M_PKTHDR	分组首部
2.5 简单的mbuf宏和函数
m_get函数
分配一个mbuf。

MGET宏
调用MALLOC分配内存

更新统计

初始化指针

m_retry函数
内存不足时调用

调用m_reclaim释放内存

2.6 m_devget和m_pullup函数
m_devget
设备驱动程序创建mbuf链

根据数据长度选择不同格式

m_pullup
保证前N字节连续

用于协议首部处理

用于IP分片重装

2.7 mbuf宏和函数的小结
常用宏
宏	说明
MGET	分配mbuf
MGETHDR	分配分组首部mbuf
M_PREPEND	在前面添加数据
dtom	数据指针→mbuf指针
mtd	mbuf数据→类型指针
常用函数
函数	说明
m_adj	删除数据
m_cat	连接mbuf链
m_copy	复制mbuf链
m_copydata	复制数据
m_pullup	保证连续
2.8 Net/3联网数据结构小结
mbuf链：通过m_next链接

有头指针的链表：通过m_nextpkt链接记录

有头/尾指针的链表

双向循环链表

2.9 m_copy和簇引用计数
簇可被多个mbuf共享

引用计数管理

避免数据复制

2.10 其他选择
mbuf复杂性是历史产物

现代系统可能使用更简单的缓存机制

2.11 小结
mbuf是Net/3网络代码的基础数据结构，支持可变长度数据和高效操作。