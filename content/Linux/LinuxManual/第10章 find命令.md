# 第10章 find命令

> 所属：第三部分 查找资料
> 来源：Linux命令速查手册

find 是Linux中最强大的文件搜索工具，可以按文件名、大小、时间、所有者、类型等多种条件查找，还能对找到的文件执行操作。

---

## 10.1 根据文件名搜索文件

```bash
find /path -name "pattern"
find ~ -name "*.txt"
find / -name "passwd" 2>/dev/null
```

按文件名搜索，支持通配符（需要加引号防止shell展开）。

- `-name` 区分大小写
- `-iname` 不区分大小写

示例：
```bash
# 在当前目录找所有jpg文件
find . -name "*.jpg"

# 不区分大小写找README
find . -iname "readme*"
```

---

## 10.2 根据拥有者搜索文件

```bash
find /path -user username
find ~ -user scott
```

按文件所有者搜索。

---

## 10.3 根据用户组搜索文件

```bash
find /path -group groupname
```

按所属组搜索。

---

## 10.4 根据文件大小搜索文件

```bash
find /path -size +10M     # 大于10MB
find /path -size -100k    # 小于100KB
find /path -size 1G       # 约等于1GB（±50%）
```

**单位**：
- `c` - 字节（bytes）
- `k` - KB（1024字节）
- `M` - MB
- `G` - GB

**前缀**：
- `+` 大于
- `-` 小于
- 无前缀 约等于（512字节块）

示例：
```bash
# 找大于100MB的文件
find / -size +100M 2>/dev/null

# 找空文件
find . -size 0
find . -empty
```

---

## 10.5 根据文件类型搜索文件

```bash
find /path -type f    # 普通文件
find /path -type d    # 目录
find /path -type l    # 符号链接
```

### 文件类型参数

| 类型 | 含义 |
|------|------|
| `f` | 普通文件（regular file） |
| `d` | 目录（directory） |
| `l` | 符号链接（symbolic link） |
| `b` | 块设备（block device） |
| `c` | 字符设备（character device） |
| `p` | 命名管道（named pipe / FIFO） |
| `s` | 套接字（socket） |

示例：
```bash
# 只找目录
find . -type d

# 只找普通文件
find . -type f
```

---

## 10.6 当表达式均为true时显示结果(AND)

```bash
find /path -name "*.txt" -size +1M
find /path -name "*.jpg" -type f
```

多个条件默认就是 **AND** 关系，全部满足才匹配。

---

## 10.7 当表达式中只有一个为true时就显示结果(OR)

```bash
find /path -name "*.txt" -o -name "*.md"
find /path \( -name "*.jpg" -o -name "*.png" \)
```

使用 `-o` 表示 **OR**，满足任一即可。

> ⚠️ 复杂组合时用括号分组，括号需要转义 `\( ... \)`。

---

## 10.8 当表达式为not true时显示结果(NOT)

```bash
find /path ! -name "*.tmp"
find /path -not -name "*.tmp"
```

取反，不满足条件的才匹配。

示例：
```bash
# 找不是目录的项
find . ! -type d
```

---

## 10.9 对搜索到的每个文件执行命令

### -exec 方式

```bash
find /path -name "*.tmp" -exec rm {} \;
find /path -name "*.jpg" -exec cp {} /backup/ \;
```

- `-exec` 对每个找到的文件执行命令
- `{}` 代表找到的文件路径
- `\;` 命令结束标记（分号要转义）

### 更高效的方式

**删除文件**（专用选项，更安全高效）：
```bash
find /path -name "*.tmp" -delete
```

**xargs 方式**（批量执行，更快）：
```bash
find /path -name "*.tmp" | xargs rm
find /path -name "*.txt" | xargs grep "pattern"
```

带空格的文件名用：
```bash
find /path -name "*.tmp" -print0 | xargs -0 rm
```

### 交互确认

```bash
find /path -name "*.tmp" -ok rm {} \;
```

`-ok` 和 `-exec` 类似，但每个文件执行前会提示确认。

---

## 10.10 将搜索结果打印到文件

```bash
find /path -name "pattern" > results.txt
find /path -name "pattern" -print > results.txt
```

重定向输出到文件。

`-print` 是默认行为，可以省略。

---

## 10.11 小结

### find 常用条件速查

| 条件 | 说明 | 示例 |
|------|------|------|
| `-name` | 文件名（区分大小写） | `-name "*.txt"` |
| `-iname` | 文件名（不区分大小写） | `-iname "readme"` |
| `-type` | 文件类型 | `-type f` |
| `-size` | 文件大小 | `-size +10M` |
| `-user` | 所有者 | `-user root` |
| `-group` | 所属组 | `-group admin` |
| `-mtime` | 修改时间（天） | `-mtime -7`（7天内） |
| `-atime` | 访问时间（天） | `-atime +30`（30天前） |
| `-perm` | 权限 | `-perm 644` |
| `-empty` | 空文件/目录 | `-empty` |

### 逻辑运算

| 运算符 | 含义 |
|--------|------|
| 默认 / `-a` | AND（与） |
| `-o` | OR（或） |
| `!` / `-not` | NOT（非） |
| `\( ... \)` | 分组 |

### 对结果执行操作

| 操作 | 说明 |
|------|------|
| `-print` | 打印路径（默认） |
| `-exec cmd {} \;` | 执行命令 |
| `-ok cmd {} \;` | 执行命令（需确认） |
| `-delete` | 删除文件 |
| `\| xargs cmd` | 管道给xargs批量执行 |

### 经典示例

```bash
# 找7天内修改过的txt文件
find . -name "*.txt" -mtime -7

# 找大于50MB的日志并删除
find /var/log -name "*.log" -size +50M -delete

# 找所有权限777的文件
find / -perm 777 -type f 2>/dev/null

# 批量修改文件后缀
find . -name "*.htm" -exec mv {} {}.html \;
```

> find功能强大，是系统管理必备技能。多练习组合各种条件，能解决很多批量处理问题。

---

#Linux #Shell #find #文件搜索 #查找 #批量操作
