# 第11章 shell

> 所属：第四部分 环境
> 来源：Linux命令速查手册

shell本身也提供很多实用功能，提升日常操作效率。本章介绍命令历史和别名两大功能。

---

## 11.1 查看命令行历史

```bash
history
```

显示之前执行过的命令历史记录。

默认保存数量由 `HISTSIZE` 变量控制（通常是500或1000条）。

历史文件位置：`~/.bash_history`

---

## 11.2 再次运行最近运行过的命令

```bash
!!
```

重复上一条命令。

常用场景（需要sudo时）：
```bash
apt update
# 报错权限不够
sudo !!
```

---

## 11.3 使用数字再次运行以前运行过的命令

```bash
!123
```

执行历史中第123条命令。

先用 `history` 查看编号。

---

## 11.4 使用字符串再次运行以前运行过的命令

```bash
!ls
```

执行**最近一条**以ls开头的命令。

搜索历史：
```bash
# 向上搜索历史命令
按 Ctrl+r，输入关键词
```

### 历史命令快捷键

| 快捷键 | 功能 |
|--------|------|
| `↑` / `Ctrl+p` | 上一条命令 |
| `↓` / `Ctrl+n` | 下一条命令 |
| `Ctrl+r` | 反向搜索历史 |
| `Ctrl+g` | 退出搜索 |
| `!!` | 重复上一条 |
| `!$` | 上一条命令的最后一个参数 |

---

## 11.5 显示所有命令的别名

```bash
alias
```

列出当前所有别名定义。

很多发行版默认设置了一些别名，比如：
```bash
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
```

---

## 11.6 查看特定命令的别名

```bash
alias ls
alias ll
```

查看某个命令的别名定义。

---

## 11.7 创建新的临时别名

```bash
alias ll='ls -la'
alias gs='git status'
alias ..='cd ..'
```

临时创建别名，**关闭终端后失效**。

---

## 11.8 创建新的永久别名

将别名写入配置文件，永久生效：

```bash
# 写入bash配置
echo "alias ll='ls -la'" >> ~/.bashrc
echo "alias gs='git status'" >> ~/.bashrc

# 立即生效
source ~/.bashrc
```

### 常见配置文件

| Shell | 配置文件 |
|-------|----------|
| bash | `~/.bashrc` 或 `~/.bash_aliases` |
| zsh | `~/.zshrc` |

> 建议别名统一放到 `~/.bash_aliases`，更整洁。

---

## 11.9 删除别名

```bash
unalias ll
```

删除指定别名。

删除所有别名：
```bash
unalias -a
```

---

## 11.10 小结

### 命令历史

- `history` 查看历史
- `!!` 重复上一条
- `!n` 执行第n条
- `!cmd` 执行最近以cmd开头的
- `Ctrl+r` 反向搜索

### 别名 alias

- `alias` 查看所有
- `alias name='command'` 创建
- `unalias name` 删除
- 写入 `~/.bashrc` 永久保存

### 实用别名推荐

```bash
# 文件操作
alias ll='ls -lah'
alias ..='cd ..'
alias ...='cd ../..'

# 安全操作（防止误删覆盖）
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Git
alias gs='git status'
alias ga='git add'
alias gc='git commit'
alias gp='git push'
```

> 历史命令和别名是提升shell效率的两个重要手段，善用它们能少敲很多字。

---

#Linux #Shell #历史命令 #别名 #alias #history #效率
