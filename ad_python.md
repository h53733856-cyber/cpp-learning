# Python 基础阶段学习笔记

> 主线：`list → dict → set → function → class → import`

---

## 1. list

### 核心要点

- 有序、可修改、允许重复。
- 下标从 `0` 开始，支持负数下标。
- 常用：
  - `append()`：末尾添加
  - `insert()`：指定位置插入
  - `pop()`：删除并返回元素
  - `len()`：长度
  - `in`：判断是否存在
- 遍历优先考虑：
  ```python
  for x in nums:
      ...
  ```
  需要下标时再用：
  ```python
  for i, x in enumerate(nums):
      ...
  ```
- 切片：
  ```python
  nums[start:end]
  ```
  `end` 不包含，左闭右开
- 列表推导式：
  ```python
  [x for x in nums if x > 0]
  ```

### 经典例题

**过滤偶数：**
```python
[x for x in nums if x % 2 == 0]
```


### 容易犯的错误

- C++ 习惯写：
  ```python
  if (x % 2 == 0)
  ```
  Python 不需要括号。

- Python 用缩进表示代码块，没有 `{}`。
- `nums[1:5]` 实际包含 `1~4`，不包含 `5`。

### 初学者重点注意

- `for x in nums` 得到的是**元素**，不是下标。
- 需要下标时不要手动写：
  ```python
  for i in range(len(nums)):
  ```
  除非确实需要下标，优先考虑 `enumerate()`。

---

## 2. dict

### 核心要点

核心结构：

```python
{
    key: value
}
```

通过 key 找 value：

```python
person["name"]
```

常用：

```python
person["age"] = 20       # 添加 / 修改
person.get("age")        # 不存在时返回 None
del person["age"]        # 删除
"name" in person         # 判断 key
```

遍历：

```python
for key in person:
    ...

for value in person.values():
    ...

for key, value in person.items():
    ...
```

### 经典例题

统计 / 遍历信息：

```python
person = {
    "name": "Tom",
    "age": 20
}

for key, value in person.items():
    print(key, value)
```

### 典型易错点

曾经容易写成：

```python
for key, value in person:
```

正确的是：

```python
for key, value in person.items():
```

因为直接遍历 dict 默认得到的是 **key**。

### 初学者重点注意

```python
person["xxx"]
```

key 不存在会：

```text
KeyError
```

而：

```python
person.get("xxx")
```

不存在时默认：

```python
None
```

所以“不确定 key 是否存在”时，`.get()` 很实用。

---

## 3. set

### 核心要点

- 元素唯一，自动去重。
- 不依赖下标访问。
- 常用：
  ```python
  add()
  remove()
  discard()
  ```
- 集合运算：
  ```python
  A & B    # 交集
  A | B    # 并集
  A - B    # 差集
  ```

### 经典例题

列表去重：

```python
nums = [1, 2, 2, 3, 3, 3]
nums = list(set(nums))
```

求两个集合共同元素：

```python
common = A & B
```

### 初学者重点注意

空集合不能写：

```python
{}
```

因为：

```python
{}
```

是空 dict。

空 set：

```python
set()
```

另外，set 不应该依赖“元素顺序”。

---

# 4. function

### 核心要点

基本结构：

```python
def add(a, b):
    return a + b
```

重点理解：

- 参数
- 返回值
- 默认参数
- 关键字参数
- `*args`
- `**kwargs`
- 局部变量 / 全局变量
- 可变对象修改 vs 重新绑定

### 经典例题

**求最大最小值：**

```python
def get_min_max(nums):
    minimum = nums[0]
    maximum = nums[0]

    for x in nums:
        if x < minimum:
            minimum = x
        if x > maximum:
            maximum = x

    return minimum, maximum
```

返回多个值本质上是返回 tuple：

```python
minimum, maximum = get_min_max(nums)
```

### `*args`

定义时：

```python
def add(*args):
    total = 0

    for x in args:
        total += x

    return total
```

`args` 是一个 tuple。

### `**kwargs`

定义时：

```python
def show_info(**kwargs):
    for key, value in kwargs.items():
        print(key, value)
```

`kwargs` 是一个 dict。

### 最重要的区别

定义：

```python
def func(*args, **kwargs):
```

意思是：

> 收集参数。

调用：

```python
func(*nums, **person)
```

意思是：

> 解包参数。

`nums = [1,2,3]`

- `func(nums)`：只传**1 个参数**，args = `([1,2,3],)`
- `func(*nums)`：**解包列表，拆开成 3 个独立位置参数**，等价`func(1,2,3)`，args=`(1,2,3)`

`person = {"name":"bob", "age":20}`

- `func(person)`：传 1 个字典参数，放进 args
- `func(**person)`：**把字典拆成关键字参数**，等价`func(name="bob", age=20)`，kwargs 拿到这个字典

### 典型易错点

`kwargs` 遍历时曾经容易写：

```python
for key, value in kwargs:
```

正确：

```python
for key, value in kwargs.items():
```

### 初学者重点注意

不要随意用：

```python
min
max
list
dict
set
```

作为变量名，否则可能覆盖 Python 内置函数 / 类型。

例如：

```python
max = 100
```

之后：

```python
max(nums)
```

就可能出问题。

### 可变对象陷阱

```python
def add_item(nums):
    nums.append(100)
```

调用后：

```python
a = [1, 2, 3]
add_item(a)
```

`a` 会变成：

```python
[1, 2, 3, 100]
```

但：

```python
def reset(nums):
    nums = []
```

不会让外面的 `a` 变成空列表。

核心区别：

> **修改对象**和**让局部变量重新指向另一个对象**不是一回事。

---

# 5. class

### 核心要点

基本结构：

```python
class Student:
    def __init__(self, name):
        self.name = name

    def study(self):
        print(self.name, "正在学习")
```

需要牢牢记住：

```text
self ≈ C++ this
```

但 Python 必须显式写出 `self`。

---

### `__init__`

类似 C++ 构造函数：

```python
def __init__(self, name):
    self.name = name
```

创建：

```python
s = Student("Tom")
```

会自动调用 `__init__`。

---

### class attribute vs instance attribute

```python
class Dog:
    species = "犬科"

    def __init__(self, name):
        self.name = name
```

这里：

```python
species
```

是 class attribute。

```python
self.name
```

是 instance attribute。

---

### `@property`

用于把“方法”包装成像属性一样访问：

```python
class BankAccount:
    def __init__(self, balance):
        self._balance = balance

    @property
    def balance(self):
        return self._balance
```

使用：

```python
account.balance
```

而不是：

```python
account.balance()
```

---

### inheritance

```python
class Animal:
    def speak(self):
        print("动物")

class Dog(Animal):
    def speak(self):
        print("汪汪")
```

Python 不需要 C++ 那样写：

```cpp
virtual
override
```

方法覆盖后会进行动态派发。

---

### `super()`

```python
class Dog(Animal):
    def __init__(self, name, age):
        super().__init__(name)
        self.age = age
```

不要简单死记成“调用父类”。

更准确：

> `super()` 按照 MRO 找下一个类。

多继承时尤其重要。

---

### 多态 / Duck Typing

Python 不一定要求：

```text
必须继承同一个父类
```

只要对象提供需要的方法，就可能可以使用。

例如：

```python
def make_speak(animal):
    animal.speak()
```

只要传入的对象有 `speak()` 就可以。

---

### `classmethod` / `staticmethod`

```python
@classmethod
def show(cls):
    ...
```

关注的是**类**，第一个参数通常是 `cls`。

```python
@staticmethod
def add(a, b):
    ...
```

不需要 `self` / `cls`。

---

### MRO

多继承：

```python
class C(B, A):
    pass
```

查找顺序：

```text
C → B → A → object
```

可以查看：

```python
C.__mro__
```

### 典型易错点

最开始容易把多继承理解成：

> “两个父类的方法都会执行。”

实际上：

```python
class Father:
    def hello(self):
        print("Father")

class Mother:
    def hello(self):
        print("Mother")

class Son(Father, Mother):
    pass
```

执行：

```python
Son().hello()
```

只会找到第一个匹配的：

```text
Father
```

不会自动把 `Mother.hello()` 也执行。

---

# 6. import / module / package

## module

一个 `.py` 文件就是一个 module。

例如：

```text
math_utils.py
```

里面：

```python
def add(a, b):
    return a + b
```

其他文件：

```python
import math_utils

math_utils.add(1, 2)
```

也可以：

```python
from math_utils import add
```

或者：

```python
import math_utils as mu
```

---

## package

例如：

```text
project/
├── main.py
└── utils/
    ├── __init__.py
    └── math_utils.py
```

导入：

```python
from utils.math_utils import multiply
```

### 重点理解

```text
.py 文件 → module
目录      → package
```

---

# 7. 标准库 vs 第三方库

标准库：

```python
import math
import random
import os
```

通常不需要自己安装。

第三方库：

```python
import requests
```

需要先：

```bash
python -m pip install requests
```

关键区别：

```text
pip install
    ↓
安装

import
    ↓
代码中使用
```

不要混淆。

---

# 8. venv / pip / requirements.txt

### venv

创建独立 Python 环境：

```bash
python3 -m venv .venv
```

激活：

```bash
source .venv/bin/activate
```

看到：

```text
(.venv)
```

说明当前 shell 已进入虚拟环境。

---

### pip

推荐：

```bash
python -m pip install requests
```

比直接：

```bash
pip install requests
```

更容易保证 pip 和当前 Python 是同一个环境。

---

### requirements.txt

记录项目依赖：

```bash
python -m pip freeze > requirements.txt
```

恢复依赖：

```bash
python -m pip install -r requirements.txt
```

核心关系：

```text
.venv
  ↓
实际的 Python 环境

requirements.txt
  ↓
环境依赖清单
```

`.venv` 通常不提交 Git，`requirements.txt` 通常提交。

---

# 9. `if __name__ == "__main__":`

一个 `.py` 文件有两种常见身份：

```text
直接运行
python test.py

被别人 import
from test import xxx
```

直接运行时：

```python
__name__ == "__main__"
```

被 import 时：

```python
__name__ == "test"
```

所以经常写：

```python
def main():
    print("程序开始")


if __name__ == "__main__":
    main()
```

作用：

> 直接运行文件时执行主程序；被 import 时不要自动执行主程序代码。

### 经典理解题

```python
# test.py

print("A")

if __name__ == "__main__":
    print("B")
```

直接：

```bash
python test.py
```

输出：

```text
A
B
```

如果：

```python
import test
```

输出：

```text
A
```

因为 import 模块时，模块的顶层代码仍然会执行，但 `__name__ == "__main__"` 不成立。