---
title: 第28章 SMTP：简单邮件传送协议
---

# 第28章 SMTP：简单邮件传送协议

## 28.1 引言

电子邮件是最流行的应用。SMTP使用TCP端口25。

### 组件
- **用户代理（UA）**：用户界面
- **报文传送代理（MTA）**：邮件传输

## 28.2 SMTP协议

### 基本命令
| 命令 | 说明 |
|------|------|
| HELO | 标识发送方 |
| MAIL FROM | 发件人 |
| RCPT TO | 收件人 |
| DATA | 邮件内容 |
| QUIT | 结束 |
| RSET | 重置 |
| VRFY | 验证地址 |
| NOOP | 空操作 |

### 邮件三部分
1. **信封**：MTA使用（MAIL FROM/RCPT TO）
2. **首部**：UA使用（From, To, Subject等）
3. **正文**：消息内容

### 中继代理
- 机构使用邮件集线器
- 简化配置
- 使用MX记录

## 28.3 SMTP的例子

### MX记录示例
mlfarm.com MX 10 mercury.hsi.com
mlfarm.com MX 20 mail-relay.uu.net

text

### VRFY和EXPN命令
- VRFY：验证地址
- EXPN：扩充邮件列表

## 28.4 SMTP的未来

### 扩充SMTP（ESMTP）
- 使用EHLO替代HELO
- 支持SIZE、8BITMIME等扩展

### MIME（通用Internet邮件扩充）
- 5个新首部字段
- 内容类型和编码

### MIME内容类型
| 类型 | 子类型 |
|------|--------|
| text | plain, rich, enriched |
| multipart | mixed, parallel, digest, alternative |
| message | rfc822, partial, external-body |
| application | octet-stream, postscript |
| image | jpeg, gif |
| audio | basic |
| video | mpeg |

### 编码方式
1. 7bit（默认）
2. quoted-printable（少量非ASCII）
3. base64（二进制数据）
4. 8bit
5. binary

## 28.5 小结

SMTP使用NVT ASCII传输邮件。ESMTP提供扩展功能，MIME支持多媒体内容。