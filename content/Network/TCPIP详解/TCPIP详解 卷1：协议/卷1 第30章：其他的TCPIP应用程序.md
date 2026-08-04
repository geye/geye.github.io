---
title: 第30章 其他的TCP/IP应用程序
---

# 第30章 其他的TCP/IP应用程序

## 30.1 引言

本章介绍Finger、Whois、Archie、WAIS、Gopher、Veronica、WWW和X Window System。

## 30.2 Finger协议

- 端口：TCP 79
- 查询用户信息
- 空行查询所有在线用户
- 安全问题常被禁用

### 示例
telnet host 79
<CR> # 查询所有用户

text

## 30.3 Whois协议

- 端口：TCP 43
- InterNIC服务器：rs.internic.net
- 查询域、网络、联系人

### 示例
telnet rs.internic.net 43
whois stevens

text

## 30.4 Archie、WAIS、Gopher、Veronica和WWW

### Archie
- FTP文件索引
- 按文件名搜索

### WAIS
- 内容搜索
- 关键字查询数据库

### Gopher
- 菜单驱动
- 统一访问多种资源

### Veronica
- Gopher标题索引

### WWW（万维网）
- 超文本浏览
- 服务器：info.cern.ch

## 30.5 X窗口系统

### 架构
- **X服务器**：管理显示器/键盘/鼠标
- **X客户**：应用程序
- 端口：6000 + display号

### 特点
- 单个服务器服务多个客户
- 使用TCP（或Unix域协议）
- 客户与服务器可不同主机

### Xscope程序
- 监视X连接
- 解析请求和应答

### LBX：低带宽X
- 用于低速链路
- 缓存、压缩等技术

## 30.6 小结

Finger和Whois提供用户信息查询。WWW、Gopher等是资源发现工具。X Window System是网络透明的图形系统。