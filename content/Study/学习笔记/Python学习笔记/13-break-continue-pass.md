---
tags:
  - Python
  - 循环
  - break
  - continue
  - pass
aliases:
  - break
  - continue
  - pass
---

# ⏹️ 13. break/continue/pass

> [!info] 本章概述
> 学习循环控制语句：break（跳出）、continue（跳过）、pass（占位）。
> 预计学习时间：30 分钟

---

## 13.1 break：跳出循环

立即终止整个循环，跳出循环体。

### 示例

```python
# 找到第一个偶数就停止
for i in range(1, 10):
    if i % 2 == 0:
        print(f"找到第一个偶数：{i}")
        break  # 跳出循环
    print(i)
```

输出：
```
1
找到第一个偶数：2
```

### while 循环中的 break

```python
# 用户输入exit就退出
while True:
    cmd = input("请输入命令：")
    if cmd == "exit":
        print("退出程序")
        break
    print(f"执行命令：{cmd}")
```

> [!tip] while True + break
> `while True` 是无限循环，配合 `break` 可以灵活控制退出时机。

### break 只跳出一层循环

> [!warning] 注意
> `break` 只能跳出**当前所在的那一层**循环，不能跳出多层。

```python
for i in range(3):
    for j in range(3):
        print(f"i={i}, j={j}")
        if j == 1:
            break  # 只跳出内层循环
    print("---")
```

输出：
```
i=0, j=0
i=0, j=1
---
i=1, j=0
i=1, j=1
---
i=2, j=0
i=2, j=1
---
```

> [!note] 跳出多层循环？
> Python 没有 goto，跳出多层循环的方法：
> 1. 用函数 + return
> 2. 用异常捕获
> 3. 用 flag 变量

---

## 13.2 continue：跳过本次循环

跳过当前这一次循环的剩余代码，直接进入下一次循环。

### 示例

```python
# 打印1到5，跳过3
for i in range(1, 6):
    if i == 3:
        continue  # 跳过本次循环，后面的print不执行
    print(i)
```

输出：
```
1
2
4
5
```

### 示例：跳过偶数

```python
# 只打印奇数
for i in range(1, 10):
    if i % 2 == 0:
        continue
    print(i)
```

### while 循环中的 continue

```python
i = 0
while i < 5:
    i += 1
    if i == 3:
        continue
    print(i)
```

> [!warning] while 中用 continue 要小心
> 一定要确保计数器在 continue 之前已经增加了，否则可能死循环！
>
> ```python
> # ❌ 错误：i==3时continue，i永远不会增加，死循环
> i = 0
> while i < 5:
>     if i == 3:
>         continue
>     print(i)
>     i += 1
> ```

---

## 13.3 break vs continue

| 语句 | 作用 | 影响范围 |
|------|------|----------|
| `break` | 跳出整个循环 | 终止整个循环 |
| `continue` | 跳过本次循环 | 只跳过当前这一次 |

### 对比示例

```python
# break：遇到3就停止
print("break示例：")
for i in range(1, 6):
    if i == 3:
        break
    print(i)
# 输出：1 2

# continue：遇到3就跳过
print("continue示例：")
for i in range(1, 6):
    if i == 3:
        continue
    print(i)
# 输出：1 2 4 5
```

---

## 13.4 pass：空语句占位符

`pass` 什么也不做，就是一个占位符，用来占位置让语法不报错。

### 什么时候用 pass？

写代码时，某个地方还没想好怎么写，先用 pass 占着位置，让程序能跑起来。

### 示例

```python
# 函数还没实现，先用pass占着
def my_func():
    pass  # 以后再写

# if 分支还没想好
if x > 0:
    print("正数")
elif x < 0:
    pass  # 以后再写
else:
    print("零")

# 类还没实现
class MyClass:
    pass

# 空循环
for i in range(10):
    pass  # 什么也不做，就是循环10次
```

> [!tip] pass 的作用
> - 语法占位，让代码不报错
> - 表示「这里以后要写东西，现在先空着」
> - 相当于 TODO 标记

### pass vs continue

```python
# pass：什么也不做，继续执行后面的代码
for i in range(5):
    if i == 2:
        pass  # 什么也不做
    print(i)
# 输出：0 1 2 3 4

# continue：跳过本次循环后面的代码
for i in range(5):
    if i == 2:
        continue
    print(i)
# 输出：0 1 3 4
```

---

## 13.5 循环 else 与 break

> [!success] Python 特色
> 循环可以带 else，break 会影响 else 是否执行。

### 规则

- 循环**正常结束**（没有遇到 break）→ 执行 else
- 循环被 **break 中断** → 不执行 else

### 示例：查找元素

```python
numbers = [1, 3, 5, 7, 9]
target = 4

for num in numbers:
    if num == target:
        print("找到了")
        break
else:
    print("没找到")  # 没有break，循环正常结束，执行else
```

### 示例：判断质数

```python
n = 17

for i in range(2, n):
    if n % i == 0:
        print(f"{n}不是质数，能被{i}整除")
        break
else:
    print(f"{n}是质数")  # 循环正常结束，说明没有找到因数
```

> [!tip] 很实用的技巧
> 不用再定义 flag 变量了，代码更简洁。

---

## 13.6 综合示例

### 猜数字游戏

```python
import random

answer = random.randint(1, 100)
count = 0

while True:
    count += 1
    guess = int(input("猜一个1-100的数字："))

    if guess < answer:
        print("太小了")
    elif guess > answer:
        print("太大了")
    else:
        print(f"恭喜你猜对了！答案是{answer}")
        print(f"你一共猜了{count}次")
        break  # 猜对了就退出循环
```

### 过滤列表

```python
# 找出列表中的正数
numbers = [1, -2, 3, -4, 5, -6, 7, -8, 9]
result = []

for num in numbers:
    if num <= 0:
        continue  # 跳过非正数
    result.append(num)

print(result)  # [1, 3, 5, 7, 9]
```

> [!tip] 更 Pythonic 的写法
> 用列表推导式更简洁：
> ```python
> result = [x for x in numbers if x > 0]
> ```

---

## 🔗 相关章节

- 上一章：[[12-循环语句|循环语句]]
- 下一章：[[14-函数详解|函数详解]]
- 列表推导式：[[06-列表详解#6.6 列表推导式|列表推导式]]
- 实战：[[24-实战小项目|实战小项目]]

---

## 📝 我的笔记

> 在这里记录你的理解、疑问和练习代码

```python
# 你的练习代码

```

---

## ✅ 本章检查清单

- [ ] 掌握 break 的用法（跳出整个循环）
- [ ] 掌握 continue 的用法（跳过本次循环）
- [ ] 理解 break 和 continue 的区别
- [ ] 掌握 pass 的用法（占位符）
- [ ] 理解循环 else 与 break 的关系
- [ ] 能在实际代码中灵活运用
