# Python 编程最佳实践

本文件定义了 Python 编程的特定规范和最佳实践，遵循 Python 社区广泛认同的标准。

**重要**: 使用 Python 编程时，必须同时遵守：
- `base.md` - 通用编程最佳实践
- `oop.md` - 面向对象编程原则（当使用 OOP 时）
- 本文件中的 Python 特定规范

## 代码风格和格式

### PEP 8 规范
- **严格遵循 PEP 8**: Python 代码必须符合 PEP 8 标准
- **行长度**: 每行最多 79 个字符，文档字符串或注释最多 72 个字符
- **缩进**: 使用 4 个空格缩进，禁止使用制表符
- **空行**: 顶层函数和类定义前后用两个空行分隔，类内方法用一个空行分隔

### 命名约定
- **变量和函数**: 使用 `snake_case` 命名
- **常量**: 使用 `UPPER_CASE` 命名
- **类名**: 使用 `PascalCase` 命名
- **模块名**: 使用 `lowercase` 或 `snake_case` 命名
- **私有属性**: 使用单下划线前缀 `_private_var`
- **特殊方法**: 使用双下划线包围 `__special__`

## 类型提示和文档

### 类型注解
- **使用类型提示**: 为所有公共函数和方法添加类型注解
- **导入类型**: 从 `typing` 模块导入必要的类型
- **向后兼容**: Python 3.8+ 使用 `from __future__ import annotations`

```python
from typing import List, Dict, Optional, Union
from pathlib import Path

def process_data(items: List[str], config: Optional[Dict[str, str]] = None) -> Dict[str, int]:
    """处理数据并返回统计结果。"""
    pass
```

### 文档字符串
- **使用 docstring**: 为所有模块、类、函数添加文档字符串
- **Google 风格**: 推荐使用 Google 风格的 docstring
- **包含类型信息**: 在 docstring 中说明参数和返回值类型

```python
def calculate_statistics(data: List[float]) -> Dict[str, float]:
    """计算数据的基本统计信息。
    
    Args:
        data: 需要计算统计信息的数字列表
        
    Returns:
        包含平均值、中位数、标准差的字典
        
    Raises:
        ValueError: 当输入数据为空时抛出
    """
    pass
```

## 异常处理

### Python 异常最佳实践
- **具体异常类型**: 捕获具体的异常类型而不是裸露的 `except:`
- **异常链**: 使用 `raise ... from` 保持异常链
- **自定义异常**: 为特定业务逻辑创建自定义异常类

```python
class DataValidationError(ValueError):
    """数据验证错误"""
    pass

def validate_data(data: Dict[str, str]) -> None:
    try:
        required_field = data['required_field']
    except KeyError as e:
        raise DataValidationError(f"缺少必需字段: {e}") from e
```

## 数据结构和算法

### 内置数据类型优先
- **优先使用内置类型**: list, dict, set, tuple 等
- **collections 模块**: 合理使用 `defaultdict`, `Counter`, `deque` 等
- **dataclasses**: 使用 `@dataclass` 创建简单的数据容器类

```python
from dataclasses import dataclass
from typing import List
from collections import defaultdict, Counter

@dataclass
class User:
    name: str
    email: str
    age: int
    
    def is_adult(self) -> bool:
        return self.age >= 18
```

### 列表推导和生成器
- **列表推导**: 优先使用列表推导而不是 `map()` 和 `filter()`
- **生成器表达式**: 对于大数据集使用生成器表达式节省内存
- **避免复杂推导**: 复杂逻辑使用传统循环

```python
# 推荐
valid_emails = [email for email in emails if '@' in email]

# 大数据集使用生成器
valid_emails_gen = (email for email in emails if '@' in email)
```

## 函数和方法设计

### 函数设计原则
- **参数默认值**: 避免使用可变对象作为默认参数值
- **关键字参数**: 使用 `*` 强制某些参数只能通过关键字传递
- **返回类型一致**: 函数应该始终返回相同类型

```python
def create_user(name: str, *, email: str, age: int = 0) -> User:
    """创建用户，email 必须通过关键字传递"""
    return User(name=name, email=email, age=age)

# 避免可变默认参数
def add_item(item: str, container: Optional[List[str]] = None) -> List[str]:
    if container is None:
        container = []
    container.append(item)
    return container
```

## 面向对象编程

### Python 特定 OOP 实践
- **遵循 oop.md 原则**: 严格遵循面向对象编程最佳实践
- **使用属性**: 使用 `@property` 装饰器创建计算属性
- **魔术方法**: 适当实现 `__str__`, `__repr__`, `__eq__` 等魔术方法
- **多重继承**: 谨慎使用多重继承，优先使用组合

```python
class BankAccount:
    def __init__(self, initial_balance: float = 0.0):
        self._balance = initial_balance
    
    @property
    def balance(self) -> float:
        return self._balance
    
    def deposit(self, amount: float) -> None:
        if amount <= 0:
            raise ValueError("存款金额必须大于零")
        self._balance += amount
    
    def __repr__(self) -> str:
        return f"BankAccount(balance={self._balance})"
```

## 模块和包组织

### 导入规范
- **导入顺序**: 标准库 → 第三方库 → 本地模块
- **避免星号导入**: 不使用 `from module import *`
- **相对导入**: 包内使用相对导入，明确导入路径

```python
# 标准库
import os
import sys
from pathlib import Path

# 第三方库
import requests
import pandas as pd

# 本地模块
from .models import User
from .utils import validate_email
```

### 包结构
- **__init__.py**: 明确定义包的公共接口
- **模块命名**: 使用小写字母和下划线
- **避免循环导入**: 合理组织模块依赖关系

## 性能和内存优化

### Python 特定性能实践
- **使用 `__slots__`**: 对于大量实例的类使用 `__slots__` 节省内存
- **字符串连接**: 使用 `join()` 而不是 `+` 连接大量字符串
- **本地变量**: 在循环中使用本地变量提高性能

```python
class Point:
    __slots__ = ['x', 'y']
    
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

# 字符串连接
def build_sql_query(conditions: List[str]) -> str:
    return "SELECT * FROM table WHERE " + " AND ".join(conditions)
```

## 测试

### Python 测试最佳实践
- **unittest 或 pytest**: 优先使用 pytest，必要时使用 unittest
- **测试命名**: 测试函数使用 `test_` 前缀
- **断言清晰**: 使用清晰的断言消息
- **Mock 使用**: 使用 `unittest.mock` 或 `pytest-mock` 进行模拟

```python
import pytest
from unittest.mock import Mock, patch

def test_user_creation():
    user = User(name="Alice", email="alice@example.com", age=25)
    assert user.name == "Alice"
    assert user.is_adult() is True

@patch('requests.get')
def test_api_call(mock_get):
    mock_get.return_value.json.return_value = {"status": "success"}
    result = api_call()
    assert result["status"] == "success"
```

## 工具和环境

### 开发工具
- **格式化工具**: 使用 `black` 或 `autopep8` 自动格式化代码
- **类型检查**: 使用 `mypy` 进行静态类型检查
- **代码质量**: 使用 `pylint` 或 `flake8` 检查代码质量
- **依赖管理**: 使用 `uv` 管理 Python 依赖和虚拟环境

### 项目环境管理
- **使用 uv**: 使用 `uv` 创建和管理虚拟环境
- **pyproject.toml**: 使用 `pyproject.toml` 定义项目依赖
- **锁定文件**: 使用 `uv.lock` 锁定具体依赖版本

## 安全实践

### Python 特定安全
- **输入验证**: 验证所有外部输入
- **SQL 注入**: 使用参数化查询或 ORM
- **序列化安全**: 避免使用 `pickle` 处理不可信数据
- **随机数**: 使用 `secrets` 模块生成加密安全的随机数

```python
import secrets
import hashlib
from typing import Dict, Any

def generate_secure_token() -> str:
    return secrets.token_urlsafe(32)

def hash_password(password: str, salt: bytes) -> str:
    return hashlib.pbkdf2_hmac('sha256', password.encode(), salt, 100000).hex()
```

---

*本规范应该与项目的具体需求相结合，在遵循社区标准的基础上保持适当的灵活性。*