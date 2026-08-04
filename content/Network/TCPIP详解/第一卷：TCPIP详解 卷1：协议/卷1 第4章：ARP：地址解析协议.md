## 卷1 第4章：ARP：地址解析协议

```markdown
---
title: 第4章 ARP：地址解析协议
---

# 第4章 ARP：地址解析协议

## 4.1 引言

ARP（Address Resolution Protocol）提供32bit IP地址到48bit硬件地址的动态映射。

## 4.2 一个例子

FTP连接过程中ARP的作用：
1. 解析主机名→IP地址（DNS）
2. TCP请求建立连接
3. 发送IP数据报
4. 判断目的在本地网络
5. 需要将IP地址→硬件地址
6. 广播ARP请求
7. 目的主机发送ARP应答
8. 发送IP数据报

## 4.3 ARP高速缓存

- 存放IP地址到硬件地址的映射
- 生存时间：20分钟（完整表项）
- 查看命令：`arp -a`

## 4.4 ARP的分组格式

### 以太网ARP帧
- 以太网首部：目的地址、源地址、帧类型（0x0806）
- ARP字段：
  - 硬件类型（1=以太网）
  - 协议类型（0x0800=IP）
  - 硬件地址长度（6）
  - 协议地址长度（4）
  - 操作码（1=请求，2=应答）
  - 发送端硬件地址
  - 发送端IP地址
  - 目的端硬件地址
  - 目的端IP地址

## 4.5 ARP举例

### 一般示例
arp who-has svr4 tell bsdi
arp reply svr4 is-at 0:0:c0:c2:9b:26

text
ARP请求是广播，应答是单播。

### 不存在主机的ARP请求
- 超时重传：5.5秒、24秒、29.5秒
- TCP连接超时：75秒

### ARP高速缓存超时
- 完整表项：20分钟
- 不完整表项：3分钟

## 4.6 ARP代理（Proxy ARP）

- 路由器代替目的主机回答ARP请求
- 让发送端误以为路由器就是目的主机
- 也称为混合ARP或ARP出租

## 4.7 免费ARP（Gratuitous ARP）

- 主机发送查询自己IP地址的ARP请求
- 用途：
  1. 检测IP地址冲突
  2. 更新其他主机ARP缓存（硬件地址变更时）

## 4.8 arp命令

```bash
arp -a              # 显示高速缓存
arp -d hostname     # 删除表项
arp -s hostname eth_addr # 添加静态表项
arp -s hostname eth_addr pub # 代理ARP
4.9 小结
ARP提供IP到硬件地址的动态映射。ARP高速缓存是关键组件，通过arp命令可查看和修改。

text
