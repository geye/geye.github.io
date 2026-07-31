# 第16章 Windows联网

> 所属：第五部分 网络
> 来源：Linux命令速查手册

Linux与Windows系统之间的文件共享，主要通过 **Samba**（SMB/CIFS协议）实现。

---

## 16.1 查找工作组的主浏览器

```bash
nmblookup -M workgroup
nmblookup -M WORKGROUP
```

查找Windows工作组中的主浏览器（Master Browser）。

### 查看本机NetBIOS名称

```bash
nmblookup -S 本机IP
```

### 列出网络上的主机

```bash
smbtree
```

以树状显示整个网络的工作组和计算机。

---

## 16.2 NetBIOS名称和IP地址的查询和映射

```bash
nmblookup -A ip_address
nmblookup -A 192.168.1.100
```

通过IP地址查询NetBIOS名称（反向查询）。

### 正向查询

```bash
nmblookup hostname
```

通过主机名查IP。

### nbtscan

```bash
nbtscan 192.168.1.0/24
```

扫描整个网段的NetBIOS名称，需要额外安装。

---

## 16.3 列出机器上的Samba共享

```bash
smbclient -L hostname
smbclient -L ip_address
smbclient -L //192.168.1.100 -U username
```

列出目标机器上的所有共享文件夹。

- `-L` 列出共享（list）
- `-U` 指定用户名

匿名查看：
```bash
smbclient -N -L hostname
```

---

## 16.4 用类似FTP的客户端访问Samba资源

```bash
smbclient //hostname/share_name -U username
smbclient "//192.168.1.100/Shared Files" -U user
```

进入Samba FTP式交互客户端，操作类似ftp命令。

### smbclient 常用命令

| 命令 | 功能 |
|------|------|
| `ls` / `dir` | 列出文件 |
| `cd 目录` | 切换目录 |
| `get 文件` | 下载文件 |
| `put 文件` | 上传文件 |
| `mget *.txt` | 批量下载 |
| `mput *.txt` | 批量上传 |
| `mkdir` / `rmdir` | 创建/删除目录 |
| `delete` | 删除文件 |
| `pwd` | 显示当前路径 |
| `help` | 帮助 |
| `quit` / `exit` | 退出 |

### 直接传文件（非交互）

```bash
# 上传
smbclient //host/share -U user -c "put localfile.txt remotefile.txt"

# 下载
smbclient //host/share -U user -c "get remotefile.txt"
```

---

## 16.5 挂载Samba文件系统

### 挂载Windows共享

```bash
mount -t cifs //hostname/share /mnt/point -o username=user,password=pass
```

将Windows共享挂载到本地目录，像本地磁盘一样使用。

### 更安全的方式（密码不写在命令行）

创建凭证文件 `/etc/samba/credentials`：
```
username=your_user
password=your_password
domain=WORKGROUP
```

```bash
chmod 600 /etc/samba/credentials
mount -t cifs //host/share /mnt/point -o credentials=/etc/samba/credentials
```

### 常用挂载选项

| 选项 | 功能 |
|------|------|
| `username=` | 用户名 |
| `password=` | 密码 |
| `domain=` | 域/工作组 |
| `uid=` | 挂载后文件所有者UID |
| `gid=` | 挂载后文件所属组GID |
| `file_mode=` | 文件权限（如0644） |
| `dir_mode=` | 目录权限（如0755） |
| `ro` | 只读挂载 |
| `rw` | 读写挂载 |

### 完整示例

```bash
mount -t cifs //192.168.1.100/Share /mnt/winshare \
  -o username=scott,password=123456,uid=1000,gid=1000,file_mode=0644,dir_mode=0755
```

### 卸载

```bash
umount /mnt/winshare
```

### fstab 开机自动挂载

在 `/etc/fstab` 中添加：
```
//host/share  /mnt/winshare  cifs  credentials=/etc/samba/credentials,uid=1000,gid=1000  0  0
```

---

## 16.6 小结

### Samba工具速查

| 命令 | 用途 |
|------|------|
| `smbtree` | 浏览整个网络的工作组和主机 |
| `nmblookup` | NetBIOS名称查询 |
| `smbclient -L` | 列出目标主机的共享 |
| `smbclient //host/share` | FTP式交互访问共享 |
| `mount -t cifs` | 挂载共享到本地目录 |

### 访问Windows共享的三种方式

1. **smbclient**：类似FTP，临时传几个文件用
2. **挂载cifs**：长期使用，像本地目录一样访问
3. **文件管理器**：图形界面直接打开 smb://host/share

### Linux端提供Samba共享

反过来让Windows访问Linux文件，需要安装配置Samba服务：
```bash
# 安装
sudo apt install samba      # Debian/Ubuntu
sudo yum install samba      # CentOS

# 配置
sudo vi /etc/samba/smb.conf

# 添加Samba用户
sudo smbpasswd -a username

# 启动服务
sudo systemctl start smbd
```

> Samba是Linux与Windows互联互通的桥梁，混合环境必备。

---

#Linux #Shell #Samba #Windows共享 #SMB #CIFS #smbclient #nmblookup
