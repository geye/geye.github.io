---
title: 卷2 第5章 接口：SLIP和环回
---

# 卷2 第5章 接口：SLIP和环回

## 5.1 引言

本章讨论SLIP和环回接口的实现。

## 5.2 代码介绍

- `net/if_slvar.h`：SLIP定义
- `net/if_sl.c`：SLIP驱动
- `net/if_loop.c`：环回驱动

## 5.3 SLIP接口

### SLIP帧格式
- 帧分隔符：0xc0（END）
- 转义字符：0xdb
- 0xc0 → 0xdb 0xdc
- 0xdb → 0xdb 0xdd

### 线路规程（SLIPDISC）
- SLIP作为TTY子系统的一个线路规程
- 过滤输入/输出数据

### slopen/slinit
- 打开TTY设备
- 关联sl_softc结构

### slinput
- 处理输入字符
- 解压缩TCP首部
- 组装帧→IP输入队列

### sloutput
- 选择队列（普通/交互）
- 支持TOS排队

### slstart
- 从队列取分组
- 压缩TCP首部（CSLIP）
- 传输到TTY

### 性能优化
- MTU=296（低时延）
- 交互队列（sc_fastq）
- CSLIP首部压缩

## 5.4 环回接口

- 输出直接变为输入
- 支持BPF

### loutput函数
- 检查路由标志
- 按地址族分用
- 放入相应输入队列

## 5.5 小结

SLIP处理串行链路IP封装，支持CSLIP。环回接口用于本地通信。