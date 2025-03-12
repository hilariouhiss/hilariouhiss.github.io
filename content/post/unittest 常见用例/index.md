---
title: unittest 常见用例
date: 2025-03-03
slug: unittest-common-cases
categories:
  - Python
description: Python 测试框架 unittest 常见用例
draft: false
lastmod: 2025-03-12
---

## 准备工作

项目结构如图所示，其中 `Vector` 是被测试模块，`tests` 目录存放所有测试文件

![项目结构](https://cdn.jsdelivr.net/gh/hilariouhiss/images@main/project_structure.png)

`__init__.py` 文件将其所在目录标识为一个模块，`example.py` 则作为整个程序的唯一入口。

`vector.py` 内容如下：

```python
class Vector:
    def __init__(self, x, y):
        if isinstance(x, (int, float)) and isinstance(y, (int, float)):
            self.x = x
            self.y = y
        else:
            raise ValueError("not a number")

    def add(self, other):
        return Vector(self.x + other.x, self.y + other.y)
  
    def mul(self, factor):
        return Vector(self.x * factor, self.y * factor)

    def dot(self, other):
        return self.x * other.x + self.y * other.y

    def norm(self):
        return (self.x * self.x + self.y * self.y) ** 0.5
```

## 编写测试用例

在 `tests` 目录下创建测试文件， 文件本身必须按照格式命名，此文件使用 `test_*.py` 格式

文件中，首先导入 `unittest` 和被测试类 `Vector`

```python
import unittest

from vector import Vector
```

类名一般含有 `Test`，表明这是个测试用例，测试用例必须继承 `unittest` 的测试用例类

```python
class TestVector(unittest.TestCase):
    # 测试类开始时运行，必须使用装饰器@classmethod
    @classmethod
    def setUpClass(self):
        print("测试类准备工作")

    @classmethod
    def tearDownClass(self):
        print("测试类运行结束")

    # 每一个测试用例开始前运行
    def setUp(self):
        print("一个测试开始了")

    def tearDown(self):
        print("一个测试结束了")

    # 测试方法必须以 test_ 开头
    def test_init(self):
        v = Vector(1, 2)
        # 使用assertEqual，报错信息更丰富
        self.assertEqual(v.x, 1)
        self.assertTrue(v.y == 2)

        with self.assertRaises(ValueError):
            v = Vector("1", "2")

    @unittest.skipIf(sys.platform == "win32", "不支持 Windows 平台")
    def test_add(self):
        v1 = Vector(1, 2)
        v2 = Vector(2, 3)
        v3 = v1.add(v2)
        self.assertEqual(v3.x, 3)
```

## 总结

上述用例已能覆盖 Python 测试 80%及以上场景，并且与 JUnit 类似，可以快速理解掌握。