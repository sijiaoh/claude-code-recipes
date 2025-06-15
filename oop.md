# 面向对象编程最佳实践

本文件定义了面向对象编程的核心原则和最佳实践，强调合理的对象边界设计、逻辑职责分离，以及避免过度抽象和类分割。

**重要**: 当使用面向对象编程时，需要严格遵守本文件中定义的所有设计原则和最佳实践。

## 核心设计原则

### 1. 概念层面的单一职责

每个类应该对一个**概念**负责，而不是对每个具体功能都分割。

```python
# 好的设计 - User 类负责"用户"概念的完整行为
class User:
    def __init__(self, name, email, password, mailer=None, storage=None):
        self.name = name
        self.email = email
        self.password = password
        self._mailer = mailer or Mailer()
        self._storage = storage or UserStorage()
    
    @classmethod
    def sign_up(cls, user_data, **dependencies):
        """用户注册 - 用户概念的核心行为"""
        user = cls(**user_data, **dependencies)
        if not user.is_valid():
            raise ValueError("用户数据无效")
        user.save()
        user.send_welcome_email()
        return user
    
    def validate_email(self):
        """用户数据验证 - 属于用户概念的内在职责"""
        if not self.email or '@' not in self.email:
            raise ValueError("无效邮箱格式")
    
    def send_welcome_email(self):
        """用户邮件行为 - 委托给邮件工具"""
        return self._mailer.send_welcome(self)
    
    def save(self):
        """用户保存行为 - 委托给存储工具"""
        return self._storage.save(self)

# 避免过度分割
class UserEmailValidator:     # 破坏用户概念完整性
    def validate(self, email): pass
```

### 2. 委托模式与代码导航

**核心原则**: 想修改某个概念的功能时，从该概念的类开始，能够清楚找到负责具体实现的类。

```python
class Order:
    def __init__(self, items, payment_processor=None, mailer=None):
        self.items = items
        self.status = OrderStatus.PENDING
        self._payment_processor = payment_processor or PaymentProcessor()
        self._mailer = mailer or Mailer()
    
    def process_payment(self):
        """想改支付逻辑？从这里找到 PaymentProcessor"""
        return self._payment_processor.process_payment(self)
    
    def send_confirmation(self):
        """想改邮件逻辑？从这里找到 Mailer"""
        return self._mailer.send_order_confirmation(self)
    
    def complete_order(self):
        """订单完整业务流程"""
        self.process_payment()
        self.status = OrderStatus.CONFIRMED
        self.send_confirmation()

class PaymentProcessor:
    def process_payment(self, order):
        # 支付处理的具体实现
        pass
```

### 3. 合理的类大小

**原则**: 类应该包含它概念范围内的所有合理内容。复杂概念的大类是合理的。

**重新设计的信号**:
- 类混合了多个不同概念
- 类的职责描述需要用"和"连接多个概念
- 方法之间没有内在关联

```python
# 合理的大类 - User 概念本身复杂
class User:
    # 可能有几十个方法，涵盖：
    # - 身份验证 (authenticate, change_password, enable_two_factor)
    # - 个人资料 (update_profile, get_display_name, update_avatar) 
    # - 偏好设置 (set_language, set_timezone, update_preferences)
    # - 邮件通知 (send_welcome_email, send_security_alert)
    # - 数据管理 (save, delete, archive, export_data)
    # - 业务逻辑 (is_active, can_access_feature, upgrade_subscription)
    
    # 这些都是"用户"概念的合理组成部分
    pass
```

### 4. 组合优于继承

- 仅在真正的"是一个"关系时使用继承
- 优先使用组合构建复杂行为
- 合理使用 Mixin 提供可重用功能片段

```python
# 好的继承 - 真正的"是一个"关系
class Vehicle:
    def start_engine(self): pass

class Car(Vehicle):
    def __init__(self, brand, doors):
        self.brand = brand
        self.doors = doors

# 优先组合 - "有一个"关系  
class User:
    def __init__(self, name, email):
        self.profile = UserProfile()  # 组合
        self.permissions = PermissionManager()  # 组合

# 合理的 Mixin
class TimestampMixin:
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.created_at = datetime.now()

class User(TimestampMixin):
    def __init__(self, name, email):
        super().__init__()
        self.name = name
        self.email = email
```

## 职责分离指导

### 概念 vs 功能的区别

**按概念分离** (推荐):
- `User` - 负责"用户"概念的所有相关行为
- `Order` - 负责"订单"概念的完整业务逻辑
- `Mailer` - 负责"邮件发送"技术实现

**按功能分离** (过度分割):
- `UserEmailValidator` - 仅验证邮箱
- `UserPasswordValidator` - 仅验证密码

### 分离判断标准

**需要分离** - 混合不同概念:
```python
class User:
    def calculate_tax_report(self): pass    # 税务概念
    def manage_inventory(self): pass        # 库存概念
```

**应该委托** - 用户概念但需要技术工具:
```python
class User:
    def send_welcome_email(self):    # 委托给邮件工具
        return self._mailer.send_welcome(self)
    
    def save(self):                  # 委托给存储工具
        return self._storage.save(self)
```

**不需要分离** - 概念的内在职责:
```python
class User:
    def validate_email(self): pass    # 用户数据验证
    def change_password(self): pass   # 用户行为
    def calculate_age(self): pass     # 用户属性计算
```

## 常见反模式

### 1. 贫血领域模型
避免只有数据没有行为的模型。

```python
# 避免 - 贫血模型
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

class UserService:
    def validate_user(self, user): pass

# 推荐 - 富领域模型
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
    
    def validate(self):
        if not self.email or '@' not in self.email:
            raise ValueError("无效邮箱")
```

### 2. 过度抽象
不要为了抽象而抽象。

```python
# 避免过度抽象
class AbstractDataProcessor:
    def process(self, data): pass

# 直接具体实现
class UserRegistrationProcessor:
    def process_registration(self, user_data):
        # 具体的用户注册逻辑
        pass
```

### 3. 特性嫉妒
逻辑应该归位到数据所在的类。

```python
# 避免 - 特性嫉妒
class OrderCalculator:
    def calculate_total(self, order):
        total = 0
        for item in order.items:
            total += item.price * item.quantity
        return total

# 推荐 - 逻辑归位
class Order:
    def calculate_total(self):
        return sum(item.subtotal() for item in self.items)

class OrderItem:
    def subtotal(self):
        return self.price * self.quantity
```

### 4. 单方法类反模式
避免仅为单个方法而创建的类，特别是 FooLogic.call() 模式。

```python
# 避免 - 单方法类反模式
class UserValidationLogic:
    def call(self, user):
        return user.email and '@' in user.email

class OrderProcessingLogic:
    def call(self, order):
        # 复杂的订单处理逻辑
        pass

# 推荐 - 逻辑归位到相关概念
class User:
    def is_valid(self):
        """用户验证 - 用户概念的内在职责"""
        return self.email and '@' in self.email

class Order:
    def process(self):
        """订单处理 - 订单概念的内在行为"""
        # 订单处理逻辑
        pass

# 例外：真正的概念恰好只有一个方法（这是可以接受的）
class PaymentGateway:
    """支付网关 - 独立的业务概念"""
    def __init__(self, api_key, endpoint):
        self.api_key = api_key
        self.endpoint = endpoint
    
    def process_payment(self, amount, payment_info):
        """支付处理 - 支付网关概念的核心职责"""
        # 当前阶段只有一个方法，但这是一个完整的概念
        # 未来可能会添加 refund(), get_transaction_status() 等方法
        pass
```

## 接口设计

### 接口隔离原则
接口应该小而专注，客户端不应依赖不使用的接口。

```python
# 好的接口设计
class Readable:
    def read(self): pass

class Writable:
    def write(self, data): pass

# 组合接口
class FileHandler(Readable, Writable):
    def read(self): pass
    def write(self, data): pass
```

## 设计模式应用

### 策略模式
```python
class PaymentStrategy:
    def process_payment(self, amount): pass

class CreditCardPayment(PaymentStrategy):
    def process_payment(self, amount): pass

class PaymentProcessor:
    def __init__(self, strategy: PaymentStrategy):
        self.strategy = strategy
```

### 工厂模式
```python
class UserFactory:
    @staticmethod
    def create_user(user_type, **kwargs):
        if user_type == "admin":
            return AdminUser(**kwargs)
        elif user_type == "regular":
            return RegularUser(**kwargs)
        else:
            raise ValueError(f"未知用户类型: {user_type}")
```

## 总结

面向对象设计的核心原则：

1. **概念层面的职责分离** - 每个类对一个概念负责
2. **委托模式** - 保持概念完整性，技术实现委托给专门工具
3. **代码导航清晰** - 从概念类能直接找到实现类
4. **合理的类大小** - 复杂概念的大类是自然和合理的
5. **组合优于继承** - 优先使用组合构建复杂行为
6. **避免反模式** - 贫血模型、过度抽象、特性嫉妒、单方法类

**核心原则**: 好的面向对象设计应该自然、直观，能够清晰反映业务逻辑和问题域的概念结构。类应该是完整概念的载体，而不是功能的简单分割。